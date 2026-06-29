---
title: "Teoría de Redes y Protoclos"
topic: "Footprint y Scanning"
layout: default
---

-------------------------------------------------------------
MAPEO DE LA RED, DESCUBIRMIENTO DE HOST Y ESCANEO DE PUERTOS
-------------------------------------------------------------
Dentro de los protocolos de red, cuando se envián paquetes, cada paquete esta compuesto por: un header y un playload.
- El header contiene la estructura específica del protocolo, es decir, asegura que que el host receptor puede interpretar correctamente lo que esta en el payload.
- El payload es la inforamción que se quiere enviar.

Tenemos que tener en cuenta el modelo OSI, que divide la comunicación entre sistemas en capas:

Capa 7 – La capa de aplicación
------------------------------------
La Capa de Aplicación es la capa más alta del modelo OSI y sirve como la interfaz directa entre el usuario y la red. Gestiona servicios de comunicación como la navegación web (HTTP, HTTPS), el correo electrónico (SMTP, IMAP, POP3), las transferencias de archivos (FTP, SFTP) y las sesiones remotas (SSH, Telnet).
Esta capa proporciona servicios de red a las aplicaciones del usuario final, asegurando que los datos estén correctamente empaquetados y listos para la transmisión. También maneja funciones como la autenticación, el intercambio de recursos y la gestión de sesiones para aplicaciones distribuidas y API.

Capa 6 – La capa de presentación
------------------------------------
La Capa de Presentación asegura que los datos enviados por la capa de aplicación de un sistema sean legibles por la capa de aplicación de otro. Es responsable del formato de los datos, la traducción, la compresión y el cifrado.
Los estándares comunes que operan en esta capa incluyen MIME, SSL/TLS y la codificación JPEG/MP3. Esencialmente, esta capa actúa como un traductor transformando las estructuras de datos en un formato que ambos sistemas pueden entender, manteniendo la eficiencia y la seguridad durante la transmisión.

Capa 5 – La capa de sesión 
-------------------------------------
La Capa de Sesión gestiona y controla el diálogo entre dos dispositivos o aplicaciones. Establece, mantiene, sincroniza y termina las sesiones de comunicación, asegurando que el intercambio de datos ocurra de manera organizada y coordinada.
Los protocolos como NetBIOS, RPC (Remote Procedure Call) y PPTP (Point-to-Point Tunneling Protocol) a menudo operan aquí. La capa también maneja los puntos de control de sesión y la recuperación, lo cual es útil para mantener la estabilidad durante transferencias de datos largas o complejas.

Capa 4 – La capa de transporte
--------------------------------------
La Capa de Transporte proporciona comunicación de extremo a extremo y entrega confiable de datos entre dispositivos. Segmenta los datos en unidades manejables y asegura que lleguen intactos, en orden y sin duplicación.
Dos protocolos clave definen esta capa:
TCP (Transmission Control Protocol): Comunicación confiable y orientada a la conexión utilizada por aplicaciones como navegadores web y clientes de correo electrónico.
UDP (User Datagram Protocol): Comunicación más rápida y sin conexión, utilizada a menudo en medios de transmisión o juegos donde la velocidad es más importante que la fiabilidad.
El control de flujo, la detección de errores y la retransmisión ocurren todos aquí, haciendo de esta capa una de las más críticas para el rendimiento y la fiabilidad de la red.

Capa 3 – La capa de red
--------------------------------------
La Capa de Red es responsable de determinar la ruta lógica que los datos toman a través de una red. Maneja la direccionamiento, el enrutamiento y el reenvío de paquetes a través de múltiples redes interconectadas.
Los protocolos principales incluyen IP (Internet Protocol), ICMP (Internet Control Message Protocol) e IPSec. Los dispositivos como los enrutadores operan en esta capa, utilizando algoritmos y tablas de enrutamiento para dirigir los paquetes de manera eficiente hacia su destino, incluso a través de internetworks vastos y complejos.

Capa 2 – La capa de enlace de datos
---------------------------------------
La Capa de Enlace de Datos proporciona una transferencia de datos confiable de nodo a nodo. Organiza bits crudos en tramas, maneja la detección y corrección de errores, y asegura un acceso ordenado al medio de transmisión física.
Esta capa se divide en dos subcapas:
Control de Enlace Lógico (LLC): Gestiona la sincronización de tramas y la verificación de errores.
Control de Acceso al Medio (MAC): Controla cómo los dispositivos acceden y comparten el medio de red.
Las tecnologías comunes aquí incluyen Ethernet (IEEE 802.3), Wi-Fi (IEEE 802.11) y PPP (Point-to-Point Protocol). Los conmutadores de red y puentes funcionan principalmente en esta capa.

Capa 1 – La capa física
----------------------------------------
La Capa Física forma la base del modelo OSI. Transmite datos binarios crudos (1s y 0s) sobre medios físicos como cables de cobre, fibras ópticas o frecuencias de radio inalámbricas. Define los estándares eléctricos, mecánicos y procedimentales para activar y mantener el enlace físico entre los dispositivos de red.
Los estándares y tecnologías clave en esta capa incluyen interfaces físicas Ethernet, RS-232, DSL, SONET y Bluetooth. Los componentes de hardware como concentradores, cables, repetidores, conectores y transceptores operan aquí, determinando la velocidad de transmisión real, la fuerza de la señal y la integridad del medio.

-----------------------
NETWORK LAYER
-----------------------
La capa de red es la encargada del direccionamiento lógico, enrutamiento y el forwarding de paquetes de datos entre hosts de diferentes redes.
Algunos ejemplos son por ejemplo, el protocolo IP(ipv4,ipv6) o el protocolo ICMP.
- El protocolo IP, sirve como direccionamiento lógico de direcciones de interfaces de red, para identificar a cada host dentro de una red.
Las direcciones IP son jerárquicas y están estructuradas en redes, subredes y CIDR(Enrutamiento entre dominios de clase).
La estructura del paquete dentro del protocolo IP, divide el mismo en header(información esencial como destino e fuente, versión, time-to-live y tipo de protocolo) y payload.
El protocolo IP divide el paquete en segmentos pequeños que serán enviados a través de la red de manera desordenada, para que el host de destino lo reensamble y construya el paquete.
La comunicación de direcciones IP puede ser de 3 tipos: unicast(uno a uno), boradcast(difusión) o multicast(uno a cierto grupo elegido de hosts).
- Subneting es una técnica que divide una red IP en una más pequeña, mejorando la eficiencia y la seguridad.
***OJO*** ASOCIADOS CON EL PROTOCOLO IP ESTAN EL PRTOCOLO ICMP(ECHO PING/ECHO REQUEST) Y EL DHCP(CONFIGURACIÓN DINAMICA DE HOSTS).

----------
IP HEADER
----------
El tamaño mínimo deL HEADER (ip_pci) es de 20 bytes mientras que el máximo es 60 bytes.

- Versión: 4 bits
Puede variar entre (0100) o (0110) dependiendo si se utiliza IP versión 4 (IPv4) o IP versión 6 (IPv6). Este campo describe el formato de la cabecera utilizada. En la tabla se describe la versión 4.

- Tamaño Cabecera (IHL): 4 bits
Longitud de la cabecera, en palabras de 32 bits. Su valor mínimo es de 5 palabras (5x32 = 160 bits, 20 bytes) para una cabecera correcta, y el máximo de 15 palabras (15x32 = 480 bits, 60 bytes).

- Tipo de Servicio: 8 bits
Indica una serie de parámetros sobre la calidad de servicio deseada durante el tránsito por una red. Algunas redes ofrecen prioridades de servicios, considerando determinado tipo de paquetes "más importantes" que otros (en particular estas redes solo admiten los paquetes con prioridad alta en momentos de sobrecarga). Estos 8 bits se agrupan de la siguiente manera:

- Los 3 primeros bits están relacionados con la precedencia de los mensajes, un indicador adjunto que indica el nivel de urgencia basado en el sistema militar de precedencia (véase Message Precedence) de la CCEB, una organización de comunicaciones electrónicas militares formada por 5 naciones. La urgencia de estos estados aumenta a medida que el número formado por estos 3 bits lo hace, y responden a los siguientes nombres.
000: De rutina.
001: Prioritario.
010: Inmediato.
011: Relámpago.
100: Invalidación relámpago.
101: Procesando llamada crítica y de emergencia.
110: Control de trabajo de Internet.
111: Control de red.
Los 5 bits de menor peso son independientes e indican características del servicio.
Desglose de bits
Bits 0 a 2: Prioridad.
Bit 3: Retardo. 0 = normal ; 1 = bajo.
Bit 4: Rendimiento. 0= normal; 1= alto.
Bit 5: Fiabilidad. 0=normal; 1= alta.
Bit 6-7: No usados. Reservados para uso futuro.
Longitud Total: 16 bits
Es el tamaño total, en octetos, del datagrama, incluyendo el tamaño de la cabecera y el de los datos. Todos los hosts deben estar preparados para aceptar datagramas de al menos 576 octetos (pueden llegar completos o fragmentados). Se elige 576 para permitir bloques de datos de tamaño razonable (512 octetos datos + 64 octetos de cabecera) aunque el máximo tamaño para la cabecera es 60 octetos, se permite un margen para cabeceras de protocolos de nivel superior.

No se recomienda mandar datagramas mayores a 576 octetos si no se tiene la certeza de que el destino está preparado para aceptarlos.

- Identificador: 16 bits
Identificador único del datagrama. Se utilizará, en caso de que el datagrama deba ser fragmentado, para poder distinguir los fragmentos de un datagrama de los de otro. El originador del datagrama debe asegurar un valor único para la pareja origen-destino y el tipo de protocolo durante el tiempo que el datagrama pueda estar activo en la red. El valor asignado en este campo debe ir en formato de red.

- Flags: 3 bits
Actualmente utilizado solo para especificar valores relativos a la fragmentación de paquetes. Los 3 bits (por orden de mayor a menor peso) son:

bit 0: Reservado; debe ser 0
bit 1: 0 = Divisible, 1 = No Divisible (DF)
bit 2: 0 = Último Fragmento, 1 = Fragmento Intermedio (le siguen más fragmentos) (MF)
La indicación de que un paquete es indivisible debe ser tenida en cuenta bajo cualquier circunstancia. Si el paquete necesitara ser fragmentado, no se enviará.
Posición de Fragmento: 13 bits
En paquetes fragmentados indica la posición, en unidades de 64 bits, que ocupa el paquete actual dentro del datagrama original. El primer paquete de una serie de fragmentos contendrá en este campo el valor 0.

- Tiempo de Vida (TTL): 8 bits
Artículo principal: Tiempo de vida (informática)
Indica el máximo número de enrutadores que un paquete puede atravesar. Cada vez que algún nodo procesa este paquete disminuye su valor en, como mínimo, una unidad. Cuando llegue a ser 0, el paquete será descartado. Típicamente toma el valor 64 o 128 en los datagramas.

- Protocolo: 8 bits
Indica el protocolo de las capas superiores al que debe entregarse el paquete Vea Números de protocolo IP para comprender como interpretar este campo.

Suma de Control de Cabecera: 16 bits
Suma de control de cabecera. Se recalcula cada vez que algún nodo cambia alguno de sus campos (por ejemplo, el Tiempo de Vida). El método de cálculo -intencionadamente simple- consiste en sumar en complemento a 1 cada palabra de 16 bits de la cabecera (considerando valor 0 para el campo de suma de control de cabecera) y hacer el complemento a 1 del valor resultante.
- Dirección IP de origen: 32 bits
Ver Direcciones IP. Debe ser dada en formato de red.
- Dirección IP de destino: 32 bits

---------------
TRANSPORT LAYER
---------------
La capa de transporte tiene un rol crucial en la comunicaciíon entre dispositivos a través de una red.
Es la responsable de proveer el final de comunicación y asegurar el orden correcto de los datos entre dispositivos.
Los protocolos más conocidos son TCP Y UPD.

- CARACTERÍSTICAS TCP -
Establece la conexión entre host antes de que se envie ningún dato.
Garantiza la llegada de los datos, si algún paquete se pierde o es corrupto, TCP automaticamente lo reenvia(ACK).
Garantiza que llega en el orden correcto, es decir, reordena el paquete antes de pasarlo a una capa del modelo OSI superior.
- El TCP 3-WAY-HANDSHAKE, son procesos que establecen que la conexión es estable:
SYN: comienza con el envío de una bandera SYN, que indica la intención positiva de establecer conexión, incluye un ISN(secuencia de números inicial), que es aleatorio.
SYN-ACK: una vez que se ha recibido el segmento SYN, el servidor responde con un segmento que contiene tano SYN como ACK banderas. El ACK, que es la confirmación y el SYN
que es otra secuencia de números aleatoria creada por el servidor.
ACK: finalmente el cliente envía otro ACK de confirmación al servidor.
EN ESTE PUNTO LA CONEXIÓN YA ESTÁ ESTABLECIDA y ambos dispositivos ya pueden transmitir datos.
- TCP usa puertos para distinguir entre sus diferentes servicios o aplicaciones.
Están comprendidos entre 0 y 65535.
0-1023: servicios y protocolos conocidos.
1024-49151: servicios o aplicaciones específicas.

- CARACTERÍSTICAS UDP -
Establece una conexión simple y más minimalista de transmitir datos.
No establece conexión antes de enviar los datos y no garantiza el mismo nivel de garantia y ordenación.
Se centra en la eficiencia y la simplicidad.
- Comunicación sin conexión: no necestia establecer la conexión antes de enviar datos.
- No garantiza ni la llegada ni el reenvio de paquetes perdidos, lo que lo hace más rápido pero menos aconsejable para ciertas aplicaciones.
- Esta hecho para aplicaciones en tiempo real, donde la baja latencia es crucial.(streaming, VoIP,etc...)
- Sin estado: no necesita ninguna información para la comunicación. Cada paquete es independiente del anterior o del siguiente.

