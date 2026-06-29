---
title: "Metasploit Framework"
topic: "Enumeración"
layout: default
---

¿Qué es Metasploit Framework?

Metasploit es un framework de pruebas de penetración (pentesting) desarrollado por Rapid7. Su objetivo es ayudar a evaluar la seguridad de sistemas simulando ataques reales de forma controlada.

No es “una herramienta para hackear sin más”, sino una plataforma profesional usada por:
	•	Auditores de seguridad
	•	Equipos Red Team
	•	Investigadores
	•	Administradores que quieren validar sus defensas

Permite:
	•	Detectar vulnerabilidades
	•	Explotarlas (en entornos controlados)
	•	Mantener acceso (post-explotación)
	•	Generar informes

⸻

🧠 Cómo funciona Metasploit (conceptos clave)

Metasploit se basa en módulos. Los principales tipos:

1. Exploits

Código que aprovecha una vulnerabilidad.
	•	Ejemplo: explotar un fallo en SMB de Windows

2. Payloads

Lo que ejecutas tras explotar:
	•	reverse shell → la víctima se conecta a ti
	•	bind shell → tú te conectas a la víctima
	•	meterpreter → payload avanzado interactivo

3. Auxiliary

No explotan directamente, sirven para:
	•	Escaneo
	•	Fuerza bruta
	•	Enumeración

4. Post

Acciones después de comprometer el sistema:
	•	Escalar privilegios
	•	Extraer contraseñas
	•	Persistencia

5. Encoders

Ofuscan payloads para evitar detección.

⸻

🖥️ Consola principal: msfconsole

Es el núcleo de Metasploit. Todo se controla desde aquí.

⸻

🔧 Comandos esenciales (los más usados)

🔍 Navegación y ayuda
	•	help → muestra ayuda general
	•	? → equivalente a help
	•	search <palabra> → busca módulos
	•	info <modulo> → info detallada
	•	use <modulo> → selecciona módulo
	•	back → salir del módulo

⸻

⚙️ Configuración
	•	show options → ver opciones necesarias
	•	set <param> <valor> → asignar valor
	•	set RHOSTS 192.168.1.10 → objetivo
	•	set LHOST 192.168.1.5 → tu máquina
	•	set LPORT 4444 → puerto
	•	unset <param> → eliminar valor
	•	setg → global (para todos los módulos)

⸻

🚀 Ejecución
	•	run → ejecuta módulo
	•	exploit → igual que run
	•	check → verifica vulnerabilidad

⸻

📡 Sesiones
	•	sessions → ver sesiones activas
	•	sessions -i <id> → interactuar
	•	sessions -k <id> → cerrar sesión

⸻

🧪 Payloads
	•	show payloads → ver disponibles
	•	set payload <payload> → elegir payload

⸻

🧰 Módulos auxiliares
	•	show auxiliary
	•	use auxiliary/scanner/...

⸻

💻 Meterpreter (cuando ya tienes acceso)

Dentro de una sesión:
	•	sysinfo → info del sistema
	•	getuid → usuario actual
	•	shell → abrir shell normal
	•	pwd → directorio actual
	•	ls → listar archivos
	•	cd → cambiar directorio
	•	download → descargar archivo
	•	upload → subir archivo

⸻

🔐 Post-explotación
	•	run post/... → ejecutar módulos post
	•	hashdump → extraer hashes
	•	migrate → moverse a otro proceso
