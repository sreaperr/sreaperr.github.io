---
title: "Vulnerability Scanning"
topic: "Análisis de Vulnerabilidades"
layout: default
---

---------------
SCANNING NESSUS
---------------
- Es un escaner de vulnerabilidades.
Automatiza el proceso de identificar vulnerabilidades y provee a nuestro sistema de información pertinente a vulnerabilidades como el código CVE.
Se puede usar la versión de pago  para escanear hasta 16 IPs.


----------------------
WEB APP SCAN WITH WMAP
----------------------
- Es un escaner de vulnerabilidades de aplicaciones web que puede ser usada para automatizar enumeración de servicios web y escaneo de vulnerabilidades
en las aplicaciones web.

Esta disponible como un plugin de metasploit  y puede descargarse directamente en metasploit.
Esta totalmente integrado con MSF.

load wmap
wmap_sites -h: ayuda
wmap_sites -a: añadir un sitio(concretamente la ip_target).
wmap_targets -h: ayuda
wmap_targets -t : http://(ip_target)
wmap_sites -l: lista contenido.
wmap_targets -l: lista objetivos.

wmap_run -h: ayuda.
wmap_run -t: muestra los módulos auxiliares que se usaran para explotar la vulnerabilidad.
wmap_run -e: ejecuta los módulos permitidos.

wmap_vulns -h: ayuda.
wmap_vulns -l: lista las vulnerabilidades.
