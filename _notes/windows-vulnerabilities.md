---
title: "Windows Vulnerabilities — Overview"
topic: "Pentesting Windows"
layout: default
---

----------------------------------
OVERVIEW OF WINDOWS VULNERABILITES
----------------------------------
- Microsoft Windows es el sistema operativo dominante en todo el mundo con una cuota de mercado mayor o igual al 70% a partir de 2021.

- La popularidad y el despliegue de Windows por parte de individuos y empresas lo convierte en un objetivo principal para los atacantes dada la superficie de amenaza.

- En los últimos 15 años, Windows ha tenido su cuota justa de vulnerabilidades graves, que van desde MS08-067 (Conflicker) hasta MS17-010 (EternalBlue).

- Dada la popularidad de Windows, la mayoría de estas vulnerabilidades tienen código de exploit accesible públicamente, lo que las hace relativamente sencillas de explotar.


=== Vulnerabilidades de Windows ===

- Microsoft Windows tiene varias versiones y lanzamientos del sistema operativo, lo que fragmenta la superficie de amenaza en términos de vulnerabilidades. Por ejemplo, vulnerabilidades que existen en Windows 7 no están presentes en Windows 10.

- Independientemente de las diversas versiones y lanzamientos, todos los sistemas operativos Windows comparten similitudes dadas su modelo y filosofía de desarrollo:
    - Los sistemas operativos Windows han sido desarrollados en el lenguaje de programación C, lo que los hace vulnerables a desbordamientos de búfer, ejecución arbitraria de código, etc.
    - Por defecto, Windows no está configurado para ejecutarse de forma segura y requiere una implementación proactiva de prácticas de seguridad para configurarlo correctamente.
    - Las vulnerabilidades recién descubiertas no son parcheadas inmediatamente por Microsoft y, dada la naturaleza fragmentada de Windows, muchos sistemas quedan sin parchear.

- Las frecuentes versiones nuevas de Windows también son un factor que contribuye a la explotación, ya que muchas empresas tardan un tiempo considerable en actualizar sus sistemas a la última versión y optan por usar versiones más antiguas que pueden verse afectadas por un número creciente de vulnerabilidades.

- Además de las vulnerabilidades inherentes, Windows también es vulnerable a vulnerabilidades multiplataforma, como los ataques de inyección SQL.

- Los sistemas/hosts que ejecutan Windows también son vulnerables a ataques físicos como: robo, dispositivos periféricos maliciosos, etc.


Tipos de Vulnerabilidades de Windows

- Divulgación de información: Vulnerabilidad que permite a un atacante acceder a datos confidenciales.

- Desbordamientos de búfer: Causados por un error de programación, permite a los atacantes escribir datos en un búfer y desbordarlo, escribiendo consecuentemente datos en direcciones de memoria asignadas.

- Ejecución remota de código: Vulnerabilidad que permite a un atacante ejecutar código de forma remota en el sistema objetivo.

- Escalada de privilegios: Vulnerabilidad que permite a un atacante elevar sus privilegios tras el compromiso inicial.

- Denegación de Servicio (DOS): Vulnerabilidad que permite a un atacante consumir los recursos de un sistema/host (CPU, RAM, Red, etc.) impidiendo que el sistema funcione con normalidad.

-------------------------------------
FREQUENTLY EXPLOITED WINDOWS SERVICES
-------------------------------------
- Microsoft Windows tiene varios servicios y protocolos nativos que pueden configurarse para ejecutarse en un host.

- Estos servicios proporcionan a un atacante un vector de acceso que puede utilizar para obtener acceso a un host objetivo.

- Tener un buen conocimiento de qué son estos servicios, cómo funcionan y sus vulnerabilidades potenciales es una habilidad vitalmente importante para un pentester.


Protocolo/Servicio | Puertos | Propósito

Microsoft IIS (Internet Information Services)
- Puertos: TCP 80/443
- Propósito: Software de servidor web propietario desarrollado por Microsoft que se ejecuta en Windows.

WebDAV (Web Distributed Authoring & Versioning)
- Puertos: TCP 80/443
- Propósito: Extensión HTTP que permite a los clientes actualizar, eliminar, mover y copiar archivos en un servidor web. WebDAV se usa para habilitar un servidor web para que actúe como servidor de archivos.

SMB/CIFS (Server Message Block Protocol)
- Puertos: TCP 445
- Propósito: Protocolo de uso compartido de archivos en red que se utiliza para facilitar el intercambio de archivos y periféricos entre computadoras en una red de área local (LAN).

RDP (Remote Desktop Protocol)
- Puertos: TCP 3389
- Propósito: Protocolo de acceso remoto GUI propietario desarrollado por Microsoft que se utiliza para autenticarse e interactuar de forma remota con un sistema Windows.

WinRM (Windows Remote Management Protocol)
- Puertos: TCP 5986/443
- Propósito: Protocolo de gestión remota de Windows que puede utilizarse para facilitar el acceso remoto a sistemas Windows.