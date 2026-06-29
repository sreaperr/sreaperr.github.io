---
title: "Windows File System Vulnerabilities"
topic: "Pentesting Windows"
layout: default
---

----------------------
ALTERNATE DATA STREAMS
----------------------
- Es un atributo de un archivo NTFS que fue diseñado para proveer de compatibilidad con MacOS HFS.
- Cualquier archivo creado con NTFS estara formado de dos partes:
** Data stream: stream por defecto que contiene datos del archivo.
** Resource stream: contiene los metadatos del archivo.
- Los atancantes pueden usar ADS para ocultar código malicioso o ejecutable en archivos legítimos para evadir su detección.
- Esto puede ser hecho ocultando el código en el atributo del archivo resource stream(metadata) de un archivo legítimo.
- Esta técnica es usada normalmente para evadir firmas básicas basadas en AVs y herramientas de escaneo estático.
  
 == EJEMPLO PRÁCTICO ==
