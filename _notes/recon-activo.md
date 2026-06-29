---
title: "Reconocimiento Activo"
topic: "Reconocimiento"
layout: default
---

======================
====  LECCIÓN 2  =====
======================

-----------------------------
ZONAS DE TRANSFERENCIA DE DNS
-----------------------------
DNS o sistema de nombres de dominio, es un protocolo que se utiliza para resolver nombres de dominio o nombres
de host por direcciones IPs.

DNS RECORDS
------------
A: nombre de dominio o de host en una ipv4.
AAAA: lo mismo pero en una ipv6.
NS: el dominio del servidor de nombres del dominio eral.
MX: resuelve un dominio en un servidor de correo.
CNAME: usado para alias de dominios.
TXT: resgistro de texto.
HINFO: se usa para información del host. 
SOA: autoridad del dominio.
SRV: resgistros de servicio.
PTR: resuelve una IP a un hostname.

INTERROGACIÓN DNS
------------------
Es el proceso de enumerar registros de DNS para un determinado dominio. Su objetivo es sondear un servidor DNS para
proporcionarnos registros DNS para un dominio específico.
Concretamente registros adicionales a los que no hayamos podido tener acceso.


ZONAS DE TRANSFERENCIA DNS
---------------------------
Cuando un servidor DNS quiere transferir archivos de zona a otro servidor DNS.


--------------
DNSENUM
--------------
Nos permite recopliar información de manera automatizada y crítica sobre nombres de dominio.


--------------
DIG
--------------
Su nombre completo es Domain Information Groper, y se emplea para obtener información sobre servidores de nombres,
direcciones IP, servidores de correo, registros TXT, y otros datos relacionados con el sistema DNS.


--------------
NMAP
--------------
*********************
CURIOSIDADES DE NMAP
*********************
Si la dirección de destino de nmap es un sistema windows, estos bloquean por lo general todos los ping de ICMP
o las sondas de ping. Nmap nos dira que el host está caido.

nmap -sn: detección de dispositivos en la red sin escaneo de puertos. Especificar la máscara de red. 
Ejecutar con privilegios de sudo.

nmap "-Pn": hace sondeo de puertos de un host destino sin verificar si el host esta activo o no.

nmap -Pn "-p-/(puerto_concreto)": hace un escaneo de todos los puertos TCP, que son alrededor de 65000.

nmap -Pn "-F": hace un escaneo de puertos de los 100 puertos más usados del host de destino.

nmap -Pn "-sU": escaneo de puertos UDP de un host de destino.

-v: parametro para verbose, detallado.

Nmap -Pn -F "-sV": detección específica de versiones de servicios de cada puerto del host de destino.

-o: Detecta el sistema operativo que el sistema host de destino esta ejecutando.(NO ES MUY PRECISO).

-sC: ejecuta scripts dentro de nmap que permiten identificar más información de los puertos que estan abiertos.

-A: escaneo agresivo, engloba -sC y -sV.

-oN/-oX: reenvia la salida de los comandos de nmap a formato .txt o .xml.

***¿Porqué ralentizar o acelerar el escaneo?***
Principalmente, para no ser detectado por sistemas de detección de intrusos que pueda haber en la red 
o en el propio host.




---------------
NETDISCOVER
---------------
Funciona mediante el envío de solicitudes ARP, que esencialmente resuelven MACs a IPs.



