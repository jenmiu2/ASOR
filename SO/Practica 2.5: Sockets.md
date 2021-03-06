
# Practica 2.5: Sockets

# Objetivos
En esta práctica, nos familiarizaremos con la interfaz de programación de sockets como base para la programación de aplicaciones basadas en red, poniendo de manifiesto las diferencias de programación entre los protocolos UDP y TCP. Además, aprenderemos a programar aplicaciones independientes de la familia de protocolos de red (IPv4 o IPv6) utilizados.

# Preparación del entorno de la práctica
```c
netprefix inet
machine 1 0 0
machine 2 0 0
```

|  Maquina | Interfaz | Red Interna | Dir. IP
|--|--|--|--|
| VM1  | eth0  | inet0 | 192.168.0.1/24  fd00::a:0:0:0:1/64 |
| VM2  | eth0  | inet0 | 192.168.0.100/24    fd00::a:0:0:0:100/64 |

# Gestion de direcciones
El uso del API BSD requiere la manipulación de direcciones de red, y traducción de estas entre las tres representaciones básicas: nombre de dominio, dirección IP (versión 4 y 6) y binario (para incluirla en la cabecera del datagrama IP).

### Ejercicio 1

Escribir un programa que obtenga todas las posibles direcciones con las que se podría crear un socket asociado a un host dado como primer argumento del programa. Para cada dirección, mostrar la IP numérica, la familia de protocolos y tipo de socket. Comprobar el resultado para:

- Una dirección IPv4 válida (ej. “147.96.1.9”).

- Una dirección IPv6 válida (ej. “fd00::a:0:0:0:1”).

- Un nombre de dominio válido (ej. “www.google.com”).

- Un nombre en /etc/hosts válido (ej. “localhost”).

- Una dirección o nombre incorrectos en cualquiera de los casos anteriores.

El programa se implementará usando getaddrinfo(3) para obtener la lista de posibles direcciones de socket (**struct** sockaddr). Cada dirección se imprimirá en su valor numérico, usando getnameinfo(3) con el  _flag_ NI_NUMERICHOST, así como la familia de direcciones y el tipo de socket.

**Nota:** Para probar el comportamiento con DNS, realizar este ejercicio en la máquina física.

```bash
# Las familias 2 y 10 son AF_INET y AF_INET6, respectivamente (ver socket.h)_

# Los tipos 1, 2, 3 son SOCK_STREAM, SOCK_DGRAM y SOCK_RAW, respectivamente_

>  **./gai www.google.com**  
66.102.1.147  2  1  
66.102.1.147  2  2  
66.102.1.147  2  3  
2a00:1450:400c:c06::67  10  1  
2a00:1450:400c:c06::67  10  2  
2a00:1450:400c:c06::67  10  3  
>  **./gai localhost**  
::1  10  1  
::1  10  2  
::1  10  3  
127.0.0.1  2  1  
127.0.0.1  2  2  
127.0.0.1  2  3  
>  **./gai ::1**  
::1  10  1  
::1  10  2

::1  10  3  
**> ./gai 1::3::4**  
Error getaddrinfo(): Name or service not known

**> ./gai noexiste.ucm.es**  
Error getaddrinfo(): Name or service not known
```
# Protocolo UDP - Servidor de hora

### Ejercicio 2 

Escribir un servidor UDP de hora de forma que:

- La dirección y el puerto son el primer y segundo argumento del programa. Las direcciones pueden expresarse en cualquier formato (nombre de host, notación de punto…). Además, el servidor debe funcionar con direcciones IPv4 e IPv6 .

- El servidor recibirá un comando (codificado en un carácter), de forma que ‘t’ devuelva la hora, ‘d’ devuelve la fecha y ‘q’ termina el proceso servidor.

- En cada mensaje el servidor debe imprimir el nombre y puerto del cliente, usar getnameinfo(3).

Probar el funcionamiento del servidor con la herramienta Netcat (comando nc o ncat) como cliente.

**Nota:** Dado que el servidor puede funcionar con direcciones IPv4 e IPv6, hay que usar **struct** sockaddr_storage para acomodar cualquiera de ellas, por ejemplo, en recvfrom(2).

Ejemplo:
| Servidor| Output |
|--|--|
|```./time_server :: 3000 ```| 2 bytes de ::FFFF:192.168.0.100:58772 2 bytes de ::FFFF:192.168.0.100:58772  2 bytes de ::FFFF:192.168.0.100:58772  Comando no soportado X  2 bytes de ::FFFF:192.168.0.100:58772 Saliendo... |
| **Cliente** |
|``` nc -u 192.168.0.1 3000``` | **t** 10:30:08 PM d  2014-01-14 X  **q**  **^C** |

**Nota:** El servidor no envía ‘\n’, por lo que se muestra la respuesta y el siguiente comando (en negrita en el ejemplo) en la misma línea.

### Ejercicio 3

Escribir el cliente para el servidor de hora. El cliente recibirá como argumentos la dirección del servidor, el puerto del servidor y el comando. Por ejemplo, para solicitar la hora, ./time_client 192.128.0.1 3000 t.

### Ejercicio 4

Modificar el servidor para que, además de poder recibir comandos por red, los pueda recibir directamente por el terminal, leyendo dos caracteres (el comando y ‘\n’) de la entrada estándar. Multiplexar el uso de ambos canales usando select(2).

### Ejercicio 5
Convertir el servidor UDP en multi-proceso siguiendo el patrón  _pre-fork_. Una vez asociado el socket a la dirección local con bind(2), crear varios procesos que llamen a recvfrom(2) de forma que cada uno atenderá un mensaje de forma concurrente. Imprimir el PID del proceso servidor para comprobarlo.

# Protocolo TCP - Servidor ECO
TCP nos ofrece un servicio orientado a conexión y fiable. Una vez creado el socket, debe ponerse en estado LISTEN (apertura pasiva, listen(2) ) y a continuación quedarse a la espera de conexiones entrantes mediante una llamada accept(2).

### Ejercicio 6
Crear un servidor TCP de eco que escuche por conexiones entrantes en una dirección (IPv4 o IPv6) y puerto dados. Cuando reciba una conexión entrante, debe mostrar la dirección y número de puerto del cliente. A partir de ese momento, enviará al cliente todo lo que reciba desde el mismo (eco). Comprobar su funcionamiento empleando la herramienta Netcat como cliente. Comprobar qué sucede si varios clientes intentan conectar al mismo tiempo.

| Servidor| Output |
|--|--|
|```./echo_server :: 2222 ```| Conexión desde fd00::a:0:0:0:1 53456 Conexión terminada  |
| **Cliente** |
|``` nc -6 fd00::a:0:0:0:1 222``` | Hola **Qué tal** Qué tal **^C** |

### Ejercicio 7
Escribir el cliente para conectarse con el servidor del ejercicio anterior. El cliente recibirá la dirección y el puerto del servidor como argumentos y, una vez establecida la conexión con el servidor, le enviará lo que el usuario escriba por teclado. Mostrará en la consola la respuesta recibida desde el servidor. Cuando el usuario escriba el carácter ‘Q’ como único carácter de una línea, el cliente cerrará la conexión con el servidor.

Ejemplo:

| Servidor| Output |
|--|--|
|```./echo_server :: 2222 ```| Conexión desde fd00::a:0:0:0:1 53445 Conexión terminada  |
| **Cliente** |
|```./echo_client fd00::a:0:0:0:1 222``` | Hola **Hola** Q |

### Ejercicio 8

Modificar el código del servidor para que acepte varias conexiones simultáneas. Cada petición debe gestionarse en un proceso diferente, siguiendo el patrón  _accept-and-fork_. El proceso padre debe cerrar el socket devuelto por accept(2).

### Ejercicio 9
Añadir la lógica necesaria en el servidor para que no quede ningún proceso en estado  _zombie_. Para ello, se deberá capturar la señal SIGCHLD y obtener la información de estado de los procesos hijos finalizados.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTExNTU4NzkxNTZdfQ==
-->