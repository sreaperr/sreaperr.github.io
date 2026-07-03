---
title: "DockerLabs - Extraviado"
difficulty: "Fácil"
source: "DockerLabs"
---

Codificación base64 en la página por defecto de Apache para obtener acceso SSH, lateral movement entre usuarios mediante credenciales en directorios ocultos, y escalada a root resolviendo un acertijo con la respuesta ofuscada en leet speak.

## Despliegue

```bash
docker load -i extraviado.tar
docker run -d --rm --name extraviado extraviado:latest
```

## Reconocimiento

```bash
nmap -sCV -p- 172.17.0.2
```

Puertos detectados:
- **22/tcp** — SSH
- **80/tcp** — HTTP (Apache 2.4.58)

## Credential leakage — Base64 en Apache

La página por defecto de Apache contiene al final del source dos cadenas en base64:

```
ZGFuaWVsYQ== : Zm9jYXJvamE=
```

Decodificar:

```bash
echo "ZGFuaWVsYQ==" | base64 -d   # daniela
echo "Zm9jYXJvamE=" | base64 -d   # focaroja
```

Credenciales: **daniela : focaroja**

## Intrusión — SSH como daniela

```bash
ssh daniela@172.17.0.2
# password: focaroja
```

## Lateral movement — daniela → diego

Navegando por el directorio personal de daniela se encuentra un directorio oculto con la contraseña de diego:

```bash
ls -la ~
cd .secreto
cat passdiego
# YmFsbGVuYW5lZ3Jh
echo "YmFsbGVuYW5lZ3Jh" | base64 -d
# ballena negra
```

Cambiar a diego:

```bash
su diego
# password: ballena negra
```

## Escalada — diego → root

Navegando manualmente por los directorios ocultos de diego se encuentra un acertijo en `.local/share/.-`:

```bash
ls -la ~/.local/share/
cd ~/.local/share/.-
cat *
```

Contenido del acertijo:

```
En un mundo de hielo, me muevo sin prisa,
con un pelaje que brilla, como la brisa.
No soy un rey, pero en cuentos soy fiel,
de un color inusual, como el cielo y el mar también.
Soy amigo de los niños, en historias de ensueño.
Quien soy, que en el frío encuentro mi dueño?
```

La respuesta es **oso azul** — animal ártico de color inusual protagonista de cuentos infantiles. En el fichero `.bash_logout` de diego hay otro base64:

```bash
echo "YjNvYXp1bA==" | base64 -d
# b3oazul
```

`b3oazul` es "osoazul" en leet speak (b=o, 3=s). La contraseña de root es **osoazul**.

```bash
su root
# password: osoazul
whoami
# root
```

## Conclusión

Cadena de ataque completa:

1. **Reconocimiento** — nmap detecta puertos 22 y 80
2. **Credential leakage** — source de Apache contiene credenciales en base64 → daniela:focaroja
3. **SSH** — acceso como daniela
4. **Lateral movement** — directorio oculto `.secreto/passdiego` con base64 → ballena negra → su diego
5. **Acertijo** — directorio oculto `.local/share/.-` contiene pista del oso azul
6. **Leet speak** — `b3oazul` en `.bash_logout` = osoazul → su root

**Lecciones aprendidas:**
- El source HTML de páginas por defecto puede contener credenciales — siempre revisar.
- `base64 -d` decodifica; `base64` codifica; `base64 -w 0` codifica sin saltos de línea.
- Los directorios ocultos (`.nombre`) se listan con `ls -la` o navegando manualmente.
- Leet speak ofusca texto visualmente: b=o, 3=s, 4=a, 1=i, 0=o, 5=s.
