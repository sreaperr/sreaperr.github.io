---
title: "Windows Privilege Escalation"
topic: "Pentesting Windows"
layout: default
---

-----------------------
WINDOWS KERNEL EXPLOITS
-----------------------
**Escalada de Privilegios**

- La escalada de privilegios es el proceso de explotar vulnerabilidades o configuraciones incorrectas en sistemas para elevar los privilegios de un usuario a otro, típicamente a un usuario con acceso administrativo o root en un sistema.
- La escalada de privilegios es un elemento vital del ciclo de vida de un ataque y es un factor determinante en el éxito general de una prueba de penetración.
- Tras obtener un punto de apoyo inicial en un sistema objetivo, se te requerirá elevar tus privilegios con el fin de realizar tareas y funcionalidades que requieren privilegios administrativos.
- La importancia de la escalada de privilegios en el proceso de pruebas de penetración no puede ser exagerada ni pasada por alto. Desarrollar tus habilidades de escalada de privilegios te marcará como un buen pentester.

**Windows Kernel**

- Un Kernel es un programa informático que es el núcleo de un sistema operativo y tiene control total sobre todos los recursos y el hardware de un sistema. Actúa como una capa de traducción entre el hardware y el software, y facilita la comunicación entre estas dos capas.

- Windows NT es el kernel que viene preinstalado en todas las versiones de Microsoft Windows y funciona como un kernel tradicional con algunas excepciones basadas en la filosofía de diseño del usuario. Consta de dos modos principales de operación que determinan el acceso a los recursos del sistema y al hardware:
  - Modo Usuario – Los programas y servicios que se ejecutan en modo usuario tienen acceso limitado a los recursos y funcionalidades del sistema.
  - Modo Kernel – El modo kernel tiene acceso sin restricciones a los recursos y funcionalidades del sistema, con la funcionalidad adicional de gestionar los dispositivos y la memoria del sistema.

**Explotación del Kernel de Windows**

- Los exploits de kernel en Windows típicamente apuntarán a vulnerabilidades en el kernel de Windows para ejecutar código arbitrario con el fin de ejecutar comandos de sistema privilegiados u obtener una shell del sistema.

- Este proceso variará según la versión de Windows que se esté atacando y el exploit de kernel que se utilice.

- La escalada de privilegios en sistemas Windows típicamente seguirá la siguiente metodología:
  - Identificar vulnerabilidades del kernel.
  - Descargar, compilar y transferir exploits del kernel al sistema objetivo.

  === HERRAMIENTAS ===

  - Windows-Exploit-Suggester.
  -Windows-Kernel-Exploits.

  === EJEMPLO PRÁCTICO ===
- Muestra que ya tiene acceso a un sistema Windows objetivo, pero que los privilegios del kernel son limitados, debido a todo lo que ya hemos dicho antes.

- Nuestro primer objetivo será obtener información acerca del kernel.
COMANDOS: getsystem
- Podemos usar un módulo auxiliar llamado "suggester" para obtener información a cerca del kernel, para ello, debemos de hacer que nuestro módulo apuntes a la sesión donde tenemos abierta línea de comandos de meterpreter con nuestro sistema objetivo.

- Con respecto a la HERRAMIENTA que usa scripts, debemos acceder al repositorio de Github y leer toda la documentación cuidadosamente. Saber que descargar y cual usar dependiendo de las características de nuestro objetivo.

------------------------
BYPASSING UAC WITH UACME
------------------------
**UAC (Control de Cuentas de Usuario)**

- El Control de Cuentas de Usuario (UAC) es una función de seguridad de Windows introducida en Windows Vista que se utiliza para evitar que se realicen cambios no autorizados en el sistema operativo.

- UAC se utiliza para garantizar que los cambios en el sistema operativo requieran la aprobación del administrador o de una cuenta de usuario que forme parte del grupo de administradores locales.

- Un usuario sin privilegios que intente ejecutar un programa con privilegios elevados recibirá el aviso de credenciales de UAC, mientras que un usuario con privilegios recibirá un aviso de consentimiento.

- Los ataques pueden eludir el UAC con el fin de ejecutar archivos ejecutables maliciosos con privilegios elevados.

**Eludir el UAC**

- Para eludir el UAC con éxito, necesitaremos tener acceso a una cuenta de usuario que forme parte del grupo de administradores locales en el sistema Windows objetivo.

- El UAC permite que un programa se ejecute con privilegios administrativos, solicitando al usuario su confirmación.

- El UAC tiene varios niveles de integridad que van de bajo a alto; si el nivel de protección del UAC está configurado por debajo de alto, los programas de Windows pueden ejecutarse con privilegios elevados sin solicitar confirmación al usuario.

- Existen múltiples herramientas y técnicas que pueden utilizarse para eludir el UAC; sin embargo, la herramienta y técnica empleada dependerá de la versión de Windows que se esté ejecutando en el sistema objetivo.

**Eludir el UAC con UACMe**

- UACMe es una herramienta de escalada de privilegios de código abierto y robusta desarrollada por @hfire0x. Puede utilizarse para eludir el UAC de Windows aprovechando diversas técnicas.
  - GitHub: https://github.com/hfiref0x/UACME

- El repositorio de GitHub de UACMe contiene una lista muy bien documentada de métodos que pueden utilizarse para eludir el UAC en múltiples versiones de Windows, desde Windows 7 hasta Windows 10.

- Permite a los atacantes ejecutar cargas maliciosas en un objetivo Windows con privilegios administrativos/elevados abusando de la herramienta integrada de Windows AutoElevate.

- El repositorio de GitHub de UACMe tiene más de 60 exploits que pueden utilizarse para eludir el UAC dependiendo de la versión de Windows que se ejecute en el objetivo.

=== EJEMPLO PRÁCTICO ===
- Comienza con un mapeo de la red: nmap (ip_target)
- Nos cambiamos al marco de METASPLOIT.
- Use, search and exploit del módulo que buscamos: palabra clave: "rejetto".
- Esto nos abre una sesión de meterpreter en nuestro equipo para poder explotar el target.

--------------------------
ACCESS TOKEN IMPERSONATION
--------------------------
- Windows access tokens son un conjunto de elementos de autenticación de procesos en windows que son creados y manegados por LSASS.
- Es responsable de identificar y describir el contexto de seguridad de un proceso que esta corriendo en un sistema. Puede actuar como una llave hacia una web cookie que provee a los usuarios acceso a un sistema o web sin tener que poner credenciales cada vez que un sistema es iniciado.
- Es generado por el winlogon.exe cada vez que un usuario se autentifica satisfactoriamente e incluye la identidad y los privilegios del usuario asociados con el proceso que se esta ejecutando. Va ligado al userinit.exe, después todo proceso hijo usado por el usuario heredara una copia del token del creador y podrar correr bajo los privilegios del mismo token de acceso.
- Están categorizados basados en la vsriedad de niveles de seguridad asignados a ellos. Estos niveles de seguridad son determinados por los privilegios que tienen asigandos estos tokens en específico.
- Un token de acceso estará asignado s uno de estos niveles de seguridad:
** Nivel-impersonal, creado como resultado directo de una no interación de inicio de sesión.
** DELEGATE-nivel, resultado directo de un inicio de sesión en windows, mediante login tradicional o de aceso remoto(RDP).
