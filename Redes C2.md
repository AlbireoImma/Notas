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

## UDP: Transporte sin conexión

Sabemos actualmente que en orden de trasnportar datos entre aplicaciones y la capa de red en una manera correcta debemos proveer **multiplexing/demultiplexing**
