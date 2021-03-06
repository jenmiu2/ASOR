
# Practica 2.4: Tuberias

# Objetivos
Las tuberías ofrecen un mecanismo sencillo y efectivo para la comunicación entre procesos en un mismo sistema. En esta práctica veremos los comandos e interfaz para la gestión de tuberías, y los patrones de comunicación típicos.

# Tuberias sin nombre
Las tuberías sin nombre son entidades gestionadas directamente por el núcleo del sistema y son un mecanismo de comunicación unidireccional eficiente para procesos relacionados (padre-hijo). La forma de compartir los identificadores de la tubería es por herencia (en la llamada fork(2)).

### Ejercicio 1

Escribir un programa que emule el comportamiento de la shell en la ejecución de una sentencia en la forma: comando1 argumento1 | comando2 argumento2. El programa creará una tubería sin nombre y creará un hijo:

- El proceso padre redireccionará la salida estándar al extremo de escritura de la tubería y ejecutará comando1 argumento1.

- El proceso hijo redireccionará la entrada estándar al extremo de lectura de la tubería y ejecutará comando2 argumento2.

Probar el funcionamiento con una sentencia similar a: ./ejercicio1 echo 12345 wc -c

**Nota:** Antes de ejecutar el comando correspondiente, deben cerrarse todos los descriptores no necesarios.

### Ejercicio 2 

Para la comunicación bi-direccional, es necesario crear dos tuberías, una para cada sentido: p_h y h_p. Escribir un programa que implemente el mecanismo de sincronización de parada y espera:

- El padre leerá de la entrada estándar (terminal) y enviará el mensaje al proceso hijo, escribiéndolo en la tubería p_h. Entonces permanecerá bloqueado esperando la confirmación por parte del hijo en la otra tubería, h_p.

- El hijo leerá de la tubería p_h, escribirá el mensaje por la salida estándar y esperará 1 segundo. Entonces, enviará el carácter ‘l’ al proceso padre, escribiéndolo en la tubería h_p, para indicar que está listo. Después de 10 mensajes enviará el carácter ‘q’ para indicar al padre que finalice.


# Tuberias con nombre
Las tuberías con nombre son un mecanismo de comunicación unidireccional, con acceso de tipo FIFO, útil para procesos sin relación de parentesco. La gestión de las tuberías con nombre es igual a la de un archivo ordinario (open, write, read…). Revisar la información en fifo(7).

### Ejercicio 3
Usar la orden mkfifo para crear una tubería con nombre. Usar las herramientas del sistema de ficheros (stat, ls…) para determinar sus propiedades. Comprobar su funcionamiento usando utilidades para escribir y leer de ficheros (ej. echo, cat, tee...).
### Ejercicio 4

Escribir un programa que abra la tubería con el nombre anterior en modo sólo escritura, y escriba en ella el primer argumento del programa. En otro terminal, leer de la tubería usando un comando adecuado.
# Multiplexación síncrona de entrada/salida

Es habitual que un proceso lea o escriba de diferentes flujos. La llamada select(2) permite multiplexar las diferentes operaciones de E/S sobre múltiples flujos.
### Ejercicio 5
Crear otra tubería con nombre. Escribir un programa que espere hasta que haya datos listos para leer en alguna de ellas. El programa debe mostrar la tubería desde la que leyó y los datos leídos. Consideraciones:

- Para optimizar las operaciones de lectura usar un  _buffer_  (ej. de 256 bytes).

- Usar read(2) para leer de la tubería y gestionar adecuadamente la longitud de los datos leídos.

- Normalmente, la apertura de la tubería para lectura se bloqueará hasta que se abra para escritura (ej. con echo 1 > tuberia). Para evitarlo, usar la opción O_NONBLOCK en open(2).

- Cuando el escritor termina y cierra la tubería, read(2) devolverá 0, indicando el fin de fichero, por lo que hay que cerrar la tubería y volver a abrirla. Si no, select(2) considerará el descriptor siempre listo para lectura y no se bloqueará.
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDEyNjIzNDg5XX0=
-->