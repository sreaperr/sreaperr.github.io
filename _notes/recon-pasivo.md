---
title: "Reconocimiento Pasivo"
topic: "Reconocimiento"
layout: default
---

======================
====  LECCIÓN 2  =====
======================


====ALCANCE DEL OBJETIVO====

Consiste en denfinir que sistemas o redes o apps como pentester estas autorizado a probar.

Normalmente la autorización la dan las empresas al igual que los límites.

--------------------------
TIPOS COMUNES DE OBJETIVOS
--------------------------
Basados en nombres de dominio y sus subdominios.

Basados en IPs y redes. Comunes en pruebas de labs.

Basados en aplicaciones, un inicio de sesión, punto final de una API o una aplicación web específica.


-----------------------------------------------------------------------------------------------------------------------


====ACTIVOvsPASIVO====

PASIVO
-------
Características:
- No hay conexión directa con el objetivo.
- Menor riesgo de detección.
- Permite un monitoreo antes de empezar con las pruebas activas.
Ejemplos:
- Dominios públicos.
- Web públicas.
- Motores de búsqueda.
- Emails públicos.

ACTIVO
-------
Características:
- Enviar tráfico al objetivo.
- Aumentar la visibilidad.
- Se realiza después de la recoplación pasiva.
Ejemplos:
- Hosts activos.
- Puertos abiertos.
- Servicios en ejecución.
- Respuestas de la red.

-------------------------------------------
				MAPING FLOW					
-------------------------------------------
1. Define Target Scope
│
└── 2. Passive Reconnaissance
    │
    ├── 2.1 Passive Techniques
    │   │
    │   ├── Domains & Subdomains
    │   ├── DNS Records
    │   ├── Whois Data
    │   ├── Website Footprinting
    │   ├── OSINT
    │   │   ├── Search Engines
    │   │   ├── Emails
    │   │   └── Breach Awareness
    │   └── Technology Fingerprinting
    │
    └── 2.2 Active Reconnaissance
        │
        ├── Host Discovery
        ├── Port Scanning
        ├── Basic Service Identification
        ├── DNS Zone Transfer Testing
        │
        └── Organize Findings
            │
            └── Proceed to Enumeration & Exploitation

----------------
ERRORES COMUNES
----------------
- Escanear sin un objetivo.
- Evitar reconocimiento pasivo.
- Escanearlo todo sin un objetivo claro.
- No documentar nada.
- Confiar en herramientas de escaneo sin verificación.


-----------------------------------------------------------------------------------------------------------------------

====RECOPILACION DE INFORMACIÓN====

Primer paso de inicio de cualquier pentestinf test.

Consiste en recopilar la mayor información de tu objetivo.

Vulnerabilidades, arquitectura, correos, credenciales, protocolos, puertos, etc...


Se divide en dos fases. Activa y pasiva: 

-Pasiva: información que no esta relacionada con comprometerse activamente con el objetivo. Por ejemplo, servicios e información públicos o accesibles. IPs del servidor que aloja la web...

-Activa: información mediante la participación activa con el objetivo. Por ejemplo, escaneo de puertos e identificación de los servicios que se están ejecutando en dichos puertos.

¿QUE IFORMACIÓN BUSCAMOS DEPENDIENDO DE SI ES ACTIVA O PASIVA ? --RESUMEN


------
PASIVA
------
IP y DNS.
nombres de dominio y propietarios de dominio.
correos electrónicos y redes sociales.
tecnologías web usadas para el sitio web.
subdominios.

------
ACTIVA
------
puertos abiertos.
conocer la infraestructura objetivo.
organización de la red.
enumerar formar de explotar la información obtenida.


-----------------------------------------------------------------------------------------------------------------------


==== RECONOCIEMIENTO DE PÁGINAS WEB Y HUELLA DIGITAL====

Vamos a enumerar lo primero que vamos a buscar cuando hacemos un reconocimiento de paginas web:
- IPs.
- Direcotrios ocultos para los motores de búsqueda.
- Nombres. 
- Correo electrónico.
- Números de teléfono.
- Direcciones físicas.
- Tecnologías que se usan para hacer la página.

----------------------------------------------------
EJEMPLOS DE COMANDOS O UTILIDADES Y PARA QUE SIRVEN
----------------------------------------------------
Host: resuelve nombres de dominio.

robots.txt: revela información que el host no desea que sea pública de un sitio web.

sitemap.xml: es un archivo que garantiza a los motores de búsqueda una forma de indexar la web, o mejor dicho que directorios o páginas que hay indexadas a la web.

Complementos de google como BuiltWith: nos da una vista más detallada de la página web y sobre como se ha hecho.

whatweb: escaneo sigiloso e información detallada del nombre de dominio en cuestión.

HTTrack: paquete que nos permite analizar en profundidad el código fuente de una página web.


-------------------
WHO IS ENUMERATION
-------------------
Whois: es un protocolo basado en TCP puerto 43 que nos permite acceder a bases de datos públicas para obtener información técnica y de registro de nombres de dominio y direcciones IP.


--------------------
NETCRAFT
--------------------
Netcraft: es un servicio para obtener la mayor información sobre un nombre de dominio destino.
Todo lo que te puedas imaginar.


--------------------
DNS RECON
--------------------
Es un script de python que nos permite conocer e investigar los registros de transferencias de zona y números de registro de DNS generales.

Viene preinstalado con Kali Linux.


---------------------
DNSDUMPSTER
---------------------
Es igual que dnsrecon pero se trata de un sitio web y la información te la ad mucho más aclarada y limpia.


---------------------
WAFWOOF
---------------------
Un WAF es un firewall de aplicaciones web. WAFWOOF es la herramienta para detectar estos firewall.
Tiene repositorio en github.
Viene también preinstalado en Kali Linux.


---------------------
SUBLIST3R
---------------------
Nos permite enumerar subdominios anexados a un sitio web por los motores de búsqueda de los navegadores, es pasivo, se usan servicios públicos.
Tiene repositorio en github.
Hay que instalarlo en Kali Linux.


-------------------------------
GOOGLE DORKS OR GOOGLE HACKING
-------------------------------
Permite conocer información pertinente del objetivo.
Cosas útiles:
- Para enumerar limitando a zonas del sitio web en concreto, usamos site:(ULR SITIO WEB)
- inurl: URLs que contengan una palabra específica.
- site:*, expone subdominios que algunas empresas no saben que son accesibles desde google, o que no quieren que se vean.
- intitle: limita la búsqueda a titulos reales de los sitios web relacionados con esa palabra.
- filetype: (tipo de archivo), limita el filtro de búsqueda a subdominios/dominios que tengan archivos como el especificado.
- intitle:index of// lista de archivos de servidores, quizás de archivos o de servidores de reproducción en vivo.
- cache: mostrara si existe, versiones anteriores de la página web que estamos investigando.
- TAMBIÉN PODEMOS USAR WAYBACKMACHINE que hacia snapshots exactamente de versiones anteriores de páginas web.
- inurl:auth_user_file.txt/passwd.txt//Son filtraciones de permisos de usuario o contraseñas.
¡OJO! LAS COMBINACIONES DE DORKS Y LOS RESPECTIVOS ARCHIVOS, SITIOS O PALABRAS CONCRETAS PUEDE SER MUY VARIADA AQUÍ SOLO HAY ALGUNOS EJEMPLOS.
------------------------
GOOGLE HACKING DATABASE
------------------------
Permite acceso a archivos confidenciales,paneles de administración sin protección, credenciales expuestas en ficheros, servidores o dispositivos IoT vulnerables...


------------------------
THEHARVESTER
------------------------
Es una herramienta de código abierto para recopilar OSINT.
Tiene repositorio de github.
Automatiza reconocimiento pasivo de una gran diversa librerái de fuentes en línea(motores de búsqueda, bbdd...)


-----------------------------
FILTRACIÓN DE BASES DE DATOS
-----------------------------
haveibeenpwned.com: agrega bases de datos de todas las filtraciones de datos de empresas que han sucedido a lo largo de la historia.
