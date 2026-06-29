---
title: "Port Scanning y Enumeración"
topic: "Enumeración"
layout: default
---

nmap -Pn -sV -O:

---------------------------------------------------------------------
IMPORTAR ESCANEO DE PUERTOS DE NMAP AL MARCO DE METASPLOIT FRAMEWORK
---------------------------------------------------------------------
- Para entrar al servicio de postgreSQL: service postgresql start
msfconsole
db_status
- Una vez hecho esto, es recomendable crear un workspace para nuestro escaneo.
Se crea de la siguiente manera: workspace -a (nombre).
- Para importar los datos de nuestro último escaneo, usaremos: db_import /(ruta de donde se ecuentra el archivo).
- Para que se haga una importación directa, podemos ejeuctar el comando nmap dentro del marco de METASPLOIT de esta manera:
db_nmap -Pn -sV -O (host)
- Podemos enumerar diferentes cosas dentro del marco de METASPLOIT como hosts, services o vuls(vulnerabilities).

-------------------
MÓDULOS AUXILIARES
-------------------
- Su ojetivo es realizar una funcionalidad como un escaneo, un descubirmiento o un fuzzing. No tienen nada que ver con la explotación.
Nos permite encontrar vulnerabilidades dentro de una red interna en la que no podemos realizar comandos de nmap o explotación directa.

- Para poder activar los modulos auxiliares, debemos de estar dentro de marco de METASPLOIT.
* Primer ejemplo: search portscan(con esto los listamos, escaneo de puertos).
* Para usarlo: use auxiliary/scanner/portscan/tcp(estamos con escaneo de puertos tcp)
* Para mostrar las opciones: show options
* Para modificar las opciones: set RHOSTS (host que estemos usando)

 === EJECUTAR EXPLOIT PARA HACER LOS MÓDULOS AUXILIARES ===

- Para cambiar usamos en el marco de METASPLOIT: exploit
*OJO* Lo que nos permite esto es abrir una línea de comando en el terminal del host de destino.

- Para hacer un escaneo de la victima correctamente, debemos poner en la línea 
de comandos: run autorute -s (subred o una de las ips de la subred).
 
- Para poner las sesiones en segundo plano, dentro de la sesión en el interpreteponemos: background.
***OJO*** PARA VER LAS SESIONES PODEMOS USAR "SESSIONS".

--------
OBJETIVO
--------
- En resumidas cuentas, el objetivo de los módulos auxiliares, es llegar a poder explotar un objetivo secundario,
al cual no tenemos acceso directo mediante la red.
