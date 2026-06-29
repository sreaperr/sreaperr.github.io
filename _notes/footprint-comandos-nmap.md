---
title: "Comandos Nmap"
topic: "Footprint y Scanning"
layout: default
---

------------
BUSCAR HOSTS
------------
nmap -sn (IP DE RED CON NETMASK): descubre equipos activos sin escanear puertos.
nmap -RP: usa ARP REQUESTS.
nmap -PS 80: envía SYN al puerto 80.
nmap -PA: envia ACK al puerto 80.
nmap -PE: usa ICMP PING REQUESTS.

nmap -sS -sV -F --host-timeout (tiempo en segundos):
nmap -sS -Sv -f --scan-delay:
------------------
ESCANEO DE PUERTOS
------------------
***OJO***CUANDO SE OBTIENEN RESULTADOS COMO FILTERED EN WINDOWS HOST, SE PUEDE CONFIRMAR QUE ESTÁS TRATANDO CON UN FIREWALL.
SI ESTA CERRADO, ES QUE EL FIREWALL NO ESTÁ ACTIVO O NO TIENE REGLAS PARA ESE PUERTO EN PARTICULAR.

nmap:Escaneo básico de puertos.
- Características:
	•	Escanea los 1000 puertos más comunes
	•	Realiza descubrimiento de host
	•	Usa TCP SYN scan si eres root, si no usa TCP connect
nmap -sS: escaneo TCP SYN.
	1.	Envía paquete SYN
	2.	Si recibe SYN-ACK → puerto abierto
	3.	Si recibe RST → puerto cerrado
- No completa el handshake TCP → es más sigiloso.
  Requiere privilegios de root.
nmap -Pn: Desactiva la detección de host.
Trata al host como si estuviera activo.
Se usa cuando el hosts bloquea pings o el hosts parece caido pero no lo esta.
nmap -Pn -F: escanea alrededor de 100 puertos comunes sin ping previo.
nmap -Pn -p: escanea puertos específicos.
nmap -Pn -p-: escanea todos los puertos TCP.
nmap -T(1,2,3,4,5)...: velocidad de nmap para el escaneo.
Más lento, más sigiloso y más rápido peores resultados por pérdida.
nmap -Pn -sS -F: escaneo rápido con SYN, sin ping previo a los puertos más comunes.
mmap -Pn -sU -p: escaneo de puertos UDP.

--------------------------------------
DETECCIÓN DE SO Y VERSIÓN DEL SERVICIO
--------------------------------------
nmap -sV: detección de servicio que corre en un puerto.
nmap -O: detecta sistema operativo del host.
nmap -O --osscan-guess: estimación más agresiva.
nmap --version-intensity -O --osscan-guess: regular la intensidad de la agresividad de la detección del sistema operativo.

----------------------------
MOTOR DE SECUENCIAS DE NMAP
----------------------------
Existen scripts para automatizar todos estos procesos.
Como nmap es de código abierto, la comunidad ha creado scripts para facilitar ciertas tareas de nmap.
Se encuentran en /usr/share/nmap/scripts/*.
Los scripts están clasificados en descrubimiento, explotación, fuerza bruta y seguridad.
nmap -sC -p-:(importante escaneo de puertos)
nmap --script=(nombre_del_script)
nmap --script-help=(nombre_del_script)
***OJO***PARA MAS CONSULTAS, MIRAR EL MAN DE NMAP.

----------------------------------------
EVASION DE FIREWALL Y ESCANEO DE PUERTOS
----------------------------------------
nmap -Pn -sA -p(puertos): nos dicen si los puertos estan filtrados a través de un firewall.
nmap -Pn -sS -sV -F:
nmap -Pn -sS -sV-p(puertos) -f --mtu (host):
nmap -Pn -sS -sV-p(puertos) -f --data-length -g(puerto de origen para parecer menos sospechoso) (cantidad) -D (host)(PUEDES ESPECIFICAR VARIAS IPS): 

----------------------------
FORMATOS DE OUTPUTS DE NMAP
----------------------------
SIEMPRE ALFINAL DEL COMANDO DESPUÉS DE LA IP HOST.
-oN (ruta)/nombre.txt o nombre.txt que se creara en la ruta acutal.
-oX (ruta)/nombre.xml o nombre.xml que se creara en la ruta acutal.
-oG (ruta)/nombre.txt o nombre.txt que se creara en la ruta acutal.