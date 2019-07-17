# Ch.3 Capa de Transporte

- **Reside entre las capas de aplicaciones y redes**
- Provee servicios de comunicación directamente a las aplicaciones en ejecución en distintos hosts
- _Mas simple es verlo con ejemplos de protocolos implementados_

## Introducción y servicios de la capa de transporte

_Provee **comunicación lógica entre aplicaciones ejecutándose en distintos hosts**, simulando una conexión directa entre un host (proceso específicamente) y otro_

- Abstrae cuan compleja la infraestructura es
- **Implementado en end points**, pero no routers
- Transforma los paquetes de las aplicaciones en paquetes de la capa de transporte (segmentos)
    - Esto es posible al quebrar la información en trozos y añadir un header de transporte a cada uno de ellos, generando un segmento. Este segmento es enviado a la capa de redes, la cual crea un datagrama con la información recibida, el cual es enviado a destino
- La capa de transporte puede asimilarse al punto final de la capa de red en cada extremo, pero la capa de transporte solo vive en los end points

## Vision general de la capa de transporte

_Internet o las redes TCP/IP, tienen disponibles protocolos de la capa de transporte para la capa de aplicaciones_

- **UDP (User Datagram Protocol)**
    - No confiable
    - Sin conexión a la aplicación a invocar
- **TCP (Transsmission Control Protocol)**
    - Confiable
    - Crea una conexión a la aplicación a invocar
- Al diseñar una aplicación se debe determinar cual protocolo en la capa de transporte sera usado al crear sus sockets
- **IP (Internet Protocol) [Protocolo de la capa de red en Internet]**
    - Provee comunicación lógica entre hosts
    - Hace su "mejor esfuerzo" para entregar los segmentos entre hosts que intentan comunicarse
    - No da garantías en la entrega [Lo cual nos dice que es poco confiable]
    - No garantiza el orden de los segmentos
    - No garantiza la integridad de la información
- La responsabilidad de UDP y TCP es extender el servicio de entrega de IP (el protocolo) a dos end systems que desean comunicarse, esto **extiende el envio de host-a-host a proceso-a-proceso**
    - Multiplexing y demultiplexing en la capa de transporte [Arqui :)]
- Tanto UDP como TCP proveen mecanismos para la detección de errores para chequear la integridad de los datos
- **UDP no asegura que los datos enviados llegarán intactos o del todo hacia su destino**, por lo cual se denota como un servicio poco confiable
- TCP ofrece varios servicios adicionales a las aplicaciones, los cuales aseguran que los datos sean entragados al proceso que los solicita, correctamente y en orden
    - Control de flujo
    - Números de secuencia
    - Acknowledgments (ACK)
    - Timers
    - Control de congestión (más que un servicio a la aplicación, es un servicio otorgado a internet en general como un todo), el cual evita el tráfico excesivo de una conexión en particular, el cual **regula las tasas de las conexiones en ambas direcciones para no saturar la red** (UDP es no regulado)

## Multiplexing y Demultiplexing

_El método que extiende el servicio de host-a-host facilitado por la capa de redes a uno de procesos-a-procesos por la capa de transporte enfocado a las aplicaciones_

- El host de destino recibe segmentos de la capa de redes, luego la capa de transporte tiene la responsabilidad de enviar los datos en los segmentos a las aplicación apropiada que se está ejecutando en el host

_Rompamos el proceso con un ejemplo_

1.- Supongamos que estas descargando páginas web mientras corres una sesión FTP y dos sesiones Telnet (teletype network, similar a un ssh, véase el detalle en [RFC 15](https://tools.ietf.org/html/rfc15)), **esto genera cuatro aplicaciones (procesos) de red en ejecución** dos procesos Telnet, un proceso FTP y un proceso HTTP.

2.- Cuando la capa de transporte de nuestro PC recibe datos de la capa de red, **necesita direccionar los datos a la aplicación (proceso) correspondiente**, alguna de las 4 activas.

3.- Recordando sabemos que un proceso (como parte de una aplicación de red) **puede tener uno o más sockets**, la capa de transporte no envía la información de forma directa a la aplicación, sino al intermediario, el socket (cada socket tiene un identificador único, el formato depende del protocolo UDP/TCP)

4.- **La capa de transporte toma un segmento dirigido a ella y lee la cabecera correspondiente**, obteniendo la información del socket al cual esta dirigido los datos y los dirige hacia el, este proceso es llamado **demultiplexing**, de forma contraria, la acción de obtener pedazos de datos en la fuente de diferentes sockets, encapsularlos con un header creando un segmento, y pasándolo hacia la capa de red es llamado **multiplexing**

Con esta información podemos recalcar dos cosas, **los sockets tienen identificadores únicos** y **cada segmento tiene campos especiales que indican el socket a donde deben ir dirigidos**. 

_TCP y UDP tienen más campos que tan solo estos_

Los campos que nombremos anteriormente son el **número de puerto de destino** y **número de puerto de origen**, cada campo es un número de 16 bits desde 0 hasta el 65535

_Los primeros 1024 números estan restringidos para servicios conocidos (HTTP por ejemplo usa el 80 y Telnet el 23), la lista esta en el [RFC 1700](https://tools.ietf.org/html/rfc1700) y ha sido actualizada en el [RFC 3232](https://tools.ietf.org/html/rfc3232)_

### **Multiplexing y demultiplexing en UDP**

>clientSocket = socket(socket.AF_INET, socket.SOCK_DGRAM)

_Forma de generar un socket en python_

Cuando un socket es creado la capa de transporte inmediatamente asigna un puerto dentro del rango de 1024 hasta 65535 que no este siendo usado, aunque también podemos decidir que puerto usar

>clientSocket.bind((‘’, 19157))

_Importante denotar que un socket UDP es identificado totalmente con la tupla IP-Puerto_

### **Multiplexing y demultiplexing en TCP**

Primero debemos notar la diferencia con UDP, esta se basa inicialmente en como se identifica un socket TCP, el cual se caracteriza por una tupla con 4 elementos (IP origen, puerto origen, IP destino, puerto destino)

En el caso que un segmento TCP llega a destino, este usa los cuatro campos para _demultiplex_ y dirigir el segmento al socket apropiado

_Un caso particular (en contraste con UDP) es cuando dos segmentos TCP llegan con diferentes IPs de origen o numeros puertos serán dirigidos por dos sockets distintos_

## Servidores web y TCP

_Usualmente un servidor web usa el puerto 80 para comunicarse (este es un puerto reservado vease [RFC 3232](https://tools.ietf.org/html/rfc3232))_

Cuando un cliente (por ejemplo, un navegador) envía segmentos al servidor, **todos los segmentos** tendrán como destino el puerto 80. En particular, el segmento de establecimiento de la conexión y el segmento que contiene la solicitud HTTP tendrán como destino el puerto 80

El servidor identifica los segmentos de distintos clientes usando la dirección IP y puerto en la cabecera del segmento recibido

_Si el cliente y servidor usan HTTP persistente, entonces durante la conexión el cliente y el servidor intercambiarán los mensajes HTTP en el mismo socket en el cual se inicio, o sea, se mantienen los sockets originales. En el caso de conexiones no persistentes, una nueva conexión TCP es creada por cada solicitud/respuesta, ergo, un nuevo socket es creado y luego cerrado por cada solicitud/respuesta_

## UDP: Transporte sin conexión ([RFC 768](https://tools.ietf.org/html/rfc768))

Sabemos actualmente que en orden de trasnportar datos entre aplicaciones y la capa de red en una manera correcta debemos proveer **multiplexing/demultiplexing**

UDP como protocolo de transporte hace lo mínimo posible que puede hacer. Además de proveer multiplexing/demultiplexing y un ligero chequeo de errores (Cheksum que se genera con la infoprmación del header) no agrega nada mas al protocolo IP

El funcionamiento es el siguiente:

1.- Toma el mensaje de la aplicación, añade los puertos de origen y destino, además agrega dos valores adicionales (largo y checksum)

2.- El resultado es enviado a la capa de red, esta encapsula el segmento en un datagrama IP y hace su mejor esfuerzo al intentar enviar el segmento a su destino

3.- Si el segmento llega al destino, UDP usa el número de puerto de destino para entregar el segmento a la aplicación correspondiente

_Notar que UDP no genera un handshaking entre hosts en la capa de transporte antes de enviar el segmento. Por esto decimos que UDP es un protocolo connectionless_

### Cuando elegir UDP sobre TCP

**Mayor control al nivel de aplicación sobre que datos son enviados y cuando**: Debido a que UDP no tiene la complejidad de TCP ni mecanismos de congestión al enviar un paquete por UDP este es enviado inmediatamente a la capa de red

**No hay necesidad de establecer una conexión**: UDP no tiene un delay al establecer una conexión, al contrario con TCP que tiene un handshake de tres vías antes de enviar datos

**No hay estado de la conexión**: TCP mantiene el estado de la conexión en los end systems, por otra parte, UDP no lo hace podiendo mantener una mayor cantidad clientes al correr una aplicación en comparación con TCP

**Overhead por cabeceras en paquetes pequeños**: TCP tiene un encabezado de 20 bytes, versus UDP que tan solo tiene 8 bytes

### Estructura UDP

La cabecera tiene sólo cuatro campos, cada uno de dos bytes. Dos corresponden a los puertos de origen y destino, un campo que contiene la cantidad de bytes del segmento (incluyendo la cabecera) y un campo con el checksum de las cabeceras que sirve para ña comprobación de errores (su cálculo se verá más adelante)

#### UDP Checksum ([RFC 1071](https://tools.ietf.org/html/rfc1071))

Este cálculo provee detectar errores en el envío de segmentos via UDP. Esto se logra en el lado del que recibe el segmento al sumar todas las palabras de 16 bits en el segmento con complemento a uno

_Notar que esta suma tiene overflow, si la suma nos da 1111111111111111, el paquete no contiene errorres, si contiene al menos un cero el paquete no es correcto en su contenido_

Debemos notar que UDP solo nos da información si tenemos un error, pero no nos dice como recuperarnos del error

## Principios de transferencia de datos confiables

**Con un canal confiable de trnasferencia de datos, no hay pérdidas o corrupción en los datos enviados**

_Notar que una capa puede ser confiable en el envío de datos pero otras no necesariamente, por ejemplo tomar TCP en la capa de transporte es un protocolo confiable, pero puede estar implementado sobre IP (técnicamente UDP), un protocolo no confiable, en la capa de red_

### Construyendo un protocolo de transferencia de datos confiable

#### Caso inicial

Construyamos el caso más simple posible:

- El lado que envía simplemente acepta datos desde la capa de más arriba vía rdt_send(data), enviando el paquete dentro del canal
- En el lado que recibe, mediante el canal obtenemos el paquete via rdt_rcv(packet), moviendo los datos a la capa superior

_Este es un protocolo simple, no hay diferencia entre una unidad de datos y un paquete. Tambien, todos los paquetes se mueven de manera unidirecional (origen -> destino), por lo cual no hay necesidad de feedback, suponiendo que no hat errores o congestión_

#### Envio con bit de error

Dando más realismo al modelo anterior, supongamos que los paquetes pueden corromperse en el trayecto (dado la naturaleza física de la trasnmisión de los paquetes). Para esto necesitaremos desarrollar mensajes de control para que desde el origen haya conocimiento de el estado de la comunicación y los paquetes enviados

Para esto nos basaremos en el protocolo ARQ (**Automatic Repeat reQuest**), el cual nos pide tres capacidades en nuestro modelo:

1.- **Detección de errores**: esta debe ser capaz de detectar y posiblemente corregir bit de error en los paquetes (por ahora sabemos que necesitamos bits extras para estas funcinalidades en nuestra cabecera, similar a)

2.- **Feedback del destino**: Dado que el envío de datos es usualmente dirigido a otro end system, la forma de saber el resultado de la operación es mediante un feedback explícito de la máquina que recibió o estaba dirigida el mensaje. Para esto usaremos como feedback positivo **ACK** y como feedback negativo **NAK**

3.- **Retransmisión**: Un paquete el cual a sido recibido con errores será retransmitido por el origen

El funcionamiento integrando el protocolo ARQ sería el siguiente:

- Se hace el envío del paquete con un checksum (similar a UDP) y esperamos respuesta del otro lado de la conexión (esperamos por un ACK o NAK por parte del destino)
- Si recibimos un ACK, el paquete ha sido exitosamente enviado y esperamos por futuros paquetes desde la capa superior
- Si recibimos un NAK, volvemos a enviar el paquete y volvemos a esperar por un ACK o NAK desde el destino

_Notar que si recibimos un NAK no podemos recibir datos para enviar desde la capa superior puesto que estamos esperando una respuesta sea ACK o NAK, esre tipo de protocolos son conocidos como **stop-and-wait**. Además notar que este protocolo como esta declarado tiene un error el cual no incluye que el paquete de feedback (ACK o NAK) pueda estar corrupto_

Para resolver el problema de feedback corrupto tenemos un par de formas:

- Responder devuelta que fue lo que dijo el destino (agregando otro tipo de paquete a nuestro protocolo), este enfoque contiene el mismo problema el cual queremos resolver (u.u) que sucede si el paquete que enviamos se corrompe estariamos dando vueltas en círculos
- Lo que podriamos hacer es tener un checksum mas complejo que no solo sea capaz de saber si hay un error, pero también recuperar la información que se ha corrompido directamente en el destino
- Otro enfoque sería que el origen volviera a enviar el paquete que recibió una respuesta corrompida. Pero esto tiene dos problemas, generaríamos paquetes duplicados, pero aún más problemástico es que el destino no sabe cual feedback fue el último recibido correctamente por el origen, ergo, no sabe a priori cuando un paquete contiene datos nuevos o una retransmisión

Una solución simple a este problema (adoptado por los protocolos actuales, TCP incluido) es añadir un nuevo campo al paquete y hacer que origen enumere los paquete mediante **un número de sequencia** en el campo creado

#### Envio con posibilidad de perder paquetes

### Pipeline vs stop-and-wait

#### Go-Back-N

#### Selective repeat