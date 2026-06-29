---
title: "Windows Credential Dumping"
topic: "Pentesting Windows"
layout: default
---

-----------------------
WINDOWS PASSWORD HASHES
-----------------------
- Los SO Windows guardan los hashes  de contraseñas de usuario en el SAM(una basde de datos).
- Se conoce como hashing al proceso de convertir una cadena de texto plano en otro valor, mediante un algorítmo de hash. El resultado es conocido como valor de hash.
- La autenticación y verifiación de las credenciales del usuario son facilitadas por la ILSA.
- Windows actualmente utiliza este tipo de hash: NTLM.

-SAM(security account manager) es una base de datos que se responsabiliza de manegar las contraseñas de las cuentas de usuario de windows. Todas las contraseñas de  usuario de windows estan haseadas.
- La base de datos de SAM  no puede ser copiada cuando el SO esta corriendo.
- El windows NT kernel mantiene el SAM protegido y como resultado, los atacantes utilizan técnicas y herramientas in-memory para saltar de los procesos LSASS a los SAM hashes.
- En versiones  modernas de windows, el SAM esta encriptado con una SYSKEY.

- NTLM(NTHASH): es una colección de protocolos de autenticación que son utilizados en windows para facilitar la autenticación entre ordenadores. El proceso de autenticación envuelve a usar un usuario y contraseña válido para autenticarse exitosamente.
- Cuando un usuario es creado, es encriptado usando el MD4 hashing algortim, mientras la contraseña original es desechada.

- NTLM mejora LM en las siguientes maneras:
** Splitea el hash en dos chunks.
** Case sensitive.
** Permite el uso de símbolos unicode y caracteres.

== NTLM(NTHASH) ==
CONTRASEÑA --> MD4 --> HASH (NTLM HASH)

------------------------------------------------------
SEARCHING FOR PASSWORDS IN WINDOWS CONFIGURATION FILES
------------------------------------------------------
- Windows puede automatizar varias tareas repetitivas como hacer rollout o la instalación de windows de varios sistemas.
- Esto es tipicamente hecho mediante el uso de "Unattended windows Setup utility". Es usado para automatizar el mass instalación/desarrollo de sistemas windows.
- Estas herramientas utilizan archivos de configuración que contienen configuraciones específicas y credenciales de cuentas de usuarios específicamente la contraseña de la cuenta de Administrador.
- Si la configuración del setup de windows desatendida con sus ficheros de configuración son el objetivo del sistema después de la instalación. Pueden revelar credenciales de cuentas de usuario que pueden ser usadas por los atacantes para autentificarse con legitimidad de windows.
- la "UNATTENDED WINDOWS SETUP UTILITY" tipicamente utiliza one de los siguientes archivos de configuración:
** C:\WINDOWS\PANTHER\UNATTEND.XML
** C:\WINDOWS\PANTHER\AUTOATTEND.XML

=== EJEMPLO PRÁCTICO ===