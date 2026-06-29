---
title: "Vulnerability Assessment"
topic: "Análisis de Vulnerabilidades"
layout: default
---

---------------------------------
OVERVIEW OF WINDOWS VULNERABILITY
---------------------------------
Microsoft Windows, se puede considerar el sistema operativo por excelencia.
Debido a u popularidad, ha sido el objetivo de muchos ataques, asi como de creación de código malicioso facilmente accesible y explotable.
- Los sistemas operativos Windows se han desarrollado en C. Esto lo hace vulnerable a desbordamiento de buffer, 
ejecucíon de código arbitrario, etc...
- Windows no es un sistema operativo que este protegido desde el principio desde la actualización a Windows 10, requiere de unos puntos
finales de seguridad avanzados. Se deben de gestionar, firewalls, antivirus, parches, etc...
- Debido a que las empresas usas versiones antiguas de Windows, salen cada vez nuevas vulnerabilidades que se pueden explotar.
- Dentro de sistemas Windows, se pueden encontrar vulnerabilidades multiplataformas como inyecciones SQL, scripting, etc...

=== TIPOS DE VULNERABILIDADES DE WINDOWS ===    

- Divulgación de información.
- Desbordamientos de búfer.
- Ejecución de código remoto.
- Escala de privilegios.
- Ataques de denegación de servicio(DOS).

------------------------------------
FRECUENTLY EXPLOITS WINDOWS SERVICES
------------------------------------
Frecuentemente, en Windows, existen servicios y protocolos que pueden ser configurados para ejecutarse sobre un host concreto.
Todas estas situaciones permiten tener un vector de ataque que utilizar por parte del pentester para obtener acceso al host.

- Tener un buen entendimiento de como funcionan los servicios, como trabajan y sus principales vulnerabilidades es vital para los pentester.


=== TABLA DE PROTOCOLO/SERVICIO/PUERTO/PROPOSITO ===
Microsoft IIS - TCP ports 80/443 - Servidores de software web propietario. Desarrollado por Microsoft usado en Windows.
WebDAV - TCP ports 80/443 - Es una extensión web que permite a los usuarios mover,copiar o eliminar archivos en un servidor web.
SMB/ICFS -TCP port 445 - Es un protocolo de intercambio de archivos entre host dentro de una red local o LAN.
RDP - TCP port 3389 - Control remoto con interfaz gráfica, que permite loguearse en el host objetivo.
WinRM TCP ports 5986/443 - Administración remota de Windows. Facilita el acceso remoto a sistemas Windows.

----------
METASPLOIT
----------
En este apartado, nos centraremos en la explotación de vulnerabilidades y servicios dentro del marco de trabajo de METASPLOIT.
También exploraremos el proceso de importación de escaneos NSS a METASPLOIT.

¡OJO! EXISTE UN COMANDO EN KALI LINUX QUE NOS PERMITE BUSCAR AUXILIARY MODULES, PARA EXPLOTR VULNERABILIDADES LLAMADA: searchsploit.

- Existe un plugin de metasploit llamado "AUTOPWN", que nos permite y facilita atacar la vulnerabilidad, mediante la identificación de 
módulos de explotación para los puertos que están abiertos en el target host. Tiene repositorio de GITHUB.

- Para cargar plugins dentro de metasploit, se usa: load "plugin_a_usar".

