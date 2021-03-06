# Practica 1.2: Conceptos Avanzados de TCP

# Objetivos
En esta práctica estudiaremos el funcionamiento del protocolo TCP. Además veremos algunos parámetros que permiten ajustar el comportamiento de las aplicaciones TCP. Finalmente se consideran algunas aplicaciones del filtrado de paquetes mediante iptables.

# Preparación del entorno

 1. Definir la máquina base de la asignatura

```bash
$sudo asorregenarte
```
 2. Crear un archivo pr1.topol con la topología de la red, que consta de 4 máquinas y dos redes. El contenido del fichero debe ser:
```c
netprefix inet
machine 1 0 0
machine 2 0 0
machine 3 0 0 1 1
machine 4 0 1
```
La sintaxis es:

```bash
machine <nº maquina> <interfaz n> <red conexion n>
```
 - Crear la topología de red que arrancará las 4 máquinas virtuales (VM1, VM2, Router y VM4).
```bash
$sudo vtopol pr2.topol
```
La tabla correspondiente:
|Maquina| IP | Comentarios
|--|--|--|
| VM1 | 192.168.0.1/24 |Añadir router como encaminador por defecto |
| VM2 | 192.168.0.2/24 | Añadir router como encaminador por defecto|
| VM3 - Router | 192.168.0.3/24 (eth0) 172.16.0.3/16 (eth1)| Añadir forwarding de paquetes |
| VM4 | 172.16.0.1/16 (eth1)| Añadir router como encaminador por defecto |

# Estados de una conexión TCP
En esta práctica usaremos la herramienta Netcat, que permite leer y escribir en conexiones de red. Netcat es muy útil para investigar y depurar el comportamiento de la red en la capa de transporte, ya que permite especificar un gran número de los parámetros de la conexión. Además para ver el estado de las conexiones de red usaremos la herramienta netstat (también puede usarse la herramienta ss, que es más moderna y completa).

### Ejercicio 1
Consultar las páginas de manual de nc y netstat. En particular, consultar las siguientes opciones de netstat: -a, -l, -n, -t y -o. Probar algunas de las opciones para ambos programas para familiarizarse con su comportamiento.

 **nc**
 ```latex
-a : ----
-l : enlaza y lista conexiones entrantes
-n : no resuelve nombres de hosts via DNS
-t : contexta a conexiones Telnet
-o : vuelca la sesión sobre un fichero
```

**ss**
```latex
-a : muestra todos los sockets 
-l : lista los sockets que escuchan
-n : no resuelve nombres de servicio
-t : tcp sockets
-o : muestra la información del timer
```

### Ejercicio 2
**(LISTEN)** Abrir un servidor TCP en el puerto 7777 en VM1 usando el comando nc -l 7777. Comprobar el estado de la conexión en el servidor con el comando ss -tln. Abrir otro servidor en el puerto 7776 en VM1 usando el comando nc -l 192.168.0.1 7776. Observar la diferencia entre ambos servidores usando ss. Comprobar que no es posible la conexión desde VM1 con localhost como dirección destino usando el comando nc localhost 7776.
**VM1**
```bash
 nc -l 7777
```

[imagen ejercicio 2 VM1](https://drive.google.com/file/d/1imtR64cbn9v-R8U_0czmhZUA0HcjOY71/view?usp=sharing)

```bash 
nc -l 192.168.0.1 7776
```
[imagen ejercicio 2 VM1](https://drive.google.com/file/d/1x-3HS5IkgSludkgiJoNEJuNFxeeF41K6/view?usp=sharing)

```latex
La diferencia en crear los anteriores puertos es que en el primer caso no asociamos una dirección IP al puerto y en el segundo caso si por lo que la dirección local de conexión puede ser cualquiera. \par En el caso de conectarse al localhost nos deniega la conexión en el caso del puerto 7776 porque se ha asociado una dir local IP.
```
```bash 
nc localhost 7776
```
[imagen ejercicio 2 VM2-localhost](https://drive.google.com/file/d/15Z1fI6lsdwehFHK4Q3bdBHl2PJkdSi2n/view?usp=sharing)

### Ejercicio 3 (VM2)
**(ESTABLISHED)** En VM2, iniciar una conexión cliente al servidor arrancado en el ejercicio anterior usando el comando nc 192.168.0.1 7777.
 ```bash
 nc 192.168.0.1 7777
 ```
 -  Comprobar el estado de la conexión e identificar los parámetros (dirección IP y puerto) con el comando netstat -tn.
 ```bash
 ss -tn
 ```
 - Iniciar una captura con Wireshark. Intercambiar un único carácter con el cliente y observar los mensajes intercambiados (especialmente los números de secuencia, confirmación y flags TCP) y determinar cuántos bytes (y número de mensajes) han sido necesarios.
[imagen ejercicio 3 - estado conexión](https://drive.google.com/file/d/15TJkltwmJ6sLq28Xix7DHRGcNfApHbGh/view?usp=sharing)
[imagen ejercicio 3 - wireshark](https://drive.google.com/file/d/15chPyTjEjkvOL9jj1wvQnVRDCK0YqDkw/view?usp=sharing)

### Ejercicio 4
**(TIMEWAIT)** Cerrar la conexión en el cliente (con Ctrl+C) y comprobar el estado de la conexión usando netstat. Usar la opción -o de netstat para observar el valor del temporizador TIMEWAIT.
[imagen ejercicio 4](https://drive.google.com/file/d/1eCL8QPgp4bwFp5fcTSaPfdaMWjVv-xFe/view?usp=sharing)

### Ejercicio 5
**(SYN-SENT y SYN-RCVD)** El comando iptables permite filtrar paquetes según los flags TCP del segmento con la opción --tcp-flags (consultar la página de manual iptables-extensions). Usando esta opción:
```bash
sudo iptables -F # para borrar las reglas anteriores
sudo iptables -L # para listar las reglas
```
- Fijar una regla en el servidor (VM1) que bloquee un mensaje del acuerdo TCP de forma que el cliente (VM2) se quede en el estado SYN-SENT. Comprobar el resultado usando netstat en el cliente.
**VM1**
```bash
sudo iptables -P INPUT DROP # todas los paquetes se rechazan
sudo iptables -A INPUT -p tcp --dport 7777 # todos los paquetes del tipo tcp del puerto 7777 se van a rechazar
ss -tan
sudo iptables -F 
```
[imagen ejercicio 5 - VM1](https://drive.google.com/file/d/1b6Ows2cTslBpfQAJAu_DPQiOXDfxEjB9/view?usp=sharing)
- Borrar la regla anterior y fijar otra en el cliente (VM2) que bloquee un mensaje del acuerdo TCP de forma que el servidor se quede en el estado SYN-RCVD. Comprobar el resultado con ss -tan en el servidor. Además, esta regla debe dejar al servidor también en el estado LAST-ACK después de cerrar la conexión en el cliente. Usar la opción -o de ss para determinar cuántas retransmisiones se realizan y con qué frecuencia.
**VM2**
```bash 
sudo iptables -P OUTPUT DROP # todas los paquetes se rechazan
sudo iptables -A OUTPUT -p tcp --dport 7777 --tcp-flags ALL ACK # todos los paquetes del tipo tcp del puerto 7777 se van a rechazar si tienen el flag/opción ACK activado
ss -tan
sudo iptables -F
```
[imagen ejercicio 5 - VM2](https://drive.google.com/file/d/1GRA4czs0c3QrhLHfyXQPgxg9HuB_eT96/view?usp=sharing)
### Ejercicio 6
Intentar una conexión a un puerto cerrado del servidor (ej. 7778) y, con ayuda de la herramienta wireshark, observar los mensajes TCP intercambiados, especialmente los flags TCP.
[imagen ejercicio 6 - VM2](https://drive.google.com/file/d/14bHNLBFArZ_E41ScURXDQQ404ZmQHIBE/view?usp=sharing)

# Introducción a la seguridad en el protocolo TCP
Diferentes aspectos del protocolo TCP pueden aprovecharse para comprometer la seguridad del sistema. En este apartado vamos a estudiar dos: ataques DoS basados en TCP SYN  _flood_  y técnicas de exploración de puertos.
### Ejercicio 7
El ataque TCP SYN  _flood_  consiste en saturar un servidor mediante el envío masivo de mensajes SYN.
-  **(Cliente VM2)** Para evitar que el atacante responda al mensaje SYN+ACK del servidor con un mensaje RST que liberaría los recursos, bloquear los mensajes SYN+ACK en el atacante con iptables.
```bash 
sudo iptables -A OUTPUT -p tcp --dport 7777 --tcp-flags ALL RST, ACK -j DROP # los paquetes que provienen del  puerte 7777 del tipo tcp con los flags/opciones RST y ACK activados van a rechazarse
```

- **(Cliente VM2)** Para enviar paquetes TCP con los datos de interés usaremos el comando hping3 (estudiar la página de manual). En este caso, enviar mensajes SYN al puerto 22 del servidor (ssh) lo más rápido posible (_flood_).
```bash
sudo hping3 -p 22 -S --flood 192.168.0.1
```
[Imagen ejercicio 7 - VM2 flood](https://drive.google.com/file/d/1bQLasqIxObjh3-7uxG1NuG-lyFKHjlTL/view?usp=sharing)
- **(Servidor VM1)** Estudiar el comportamiento de la máquina, en términos del número de paquetes recibidos. Comprobar si es posible la conexión al servicio ssh.
[imagen ejercicio 7 - VM1/VM2 wireshark](https://drive.google.com/file/d/1Uuaur1x9EjQRKMtLhHZiJQbyY_EdCaAG/view?usp=sharing)
Se intenta establecer conexión con localhost:
```bash
ssh localhost
```
[Imagen ejercicio 7 - VM2 localhost](https://drive.google.com/file/d/1rBAGANdMpf4WQz7N6IqWlNXYh3EOEOuP/view?usp=sharing)

### Ejercicio 8
**(Técnica CONNECT)** Netcat permite explorar puertos usando la técnica CONNECT que intenta establecer una conexión a un puerto determinado. En función de la respuesta (SYN+ACK o RST), es posible determinar si hay un proceso escuchando.
- **(Servidor VM1)** Abrir un servidor en el puerto 7777.
```bash
nc -l 7777
```
- **(Cliente VM2)** Explorar de uno en uno, el rango de puertos 7775-7780 usando nc, en este caso usar las opciones de exploración (-z) y de salida detallada (-v).  **Nota:**  La versión de nc no soporta rangos de puertos. Por tanto, se debe hacer manualmente, o bien, incluir la sentencia de exploración de un puerto en un bucle para automatizar el proceso.
Creamos un script bash para ejecutar el comando nc contra los diferentes puertos:
```bash
gedit ejercicio8.sh
```
Contenido del ejercicio8.sh:
```bash
#!/bin/bash
for i in {7775..7780}
do 
	echo -n "puerto: $i"
	printf "\n"
	nc -z -v 192.168.0.1 $i
done
	printf "\n"
```
Ejecutar el comando:
```bash
bash ejercicio8.sh
```
- Con ayuda de wireshark observar los paquetes intercambiados.
**Opcional:** La herramienta nmap permite realizar diferentes tipos de exploración de puertos, que emplean estrategias más eficientes. Estas estrategias (SYN  _stealth_, ACK  _stealth_, FIN-ACK  _stealth_…) son más rápidas que la anterior y se basan en el funcionamiento del protocolo TCP. Estudiar la página de manual de nmap (PORT SCANNING TECHNIQUES) y emplearlas para explorar los puertos del servidor. Comprobar con wireshark los mensajes intercambiados.
Tipos de escaneo:
 TCP syn scan: -sS
 TCP ack scan: -sA
 TCP FIN scan: -sF
Para ver más detalles sobre el escaneo usamos las siguientes opciones:
 -v: aumenta el nivel del mensaje detallado
 -A: nos indica el sistema operativo y la versión
Creamos un script que permita dar permisos root y que escanee todos los puertos 7775-7778 del host 192.168.0.1 en la maquina virtual deseada **VM2**.

```bash
gedit ejercicio8a.sh
```
Contenido del ejercicio8a.sh:
```bash
#!/bin/bash
printf "escaneo tipoi -sS: \n"
sudo nmap -v - A -sS -p 7775-7778 192.168.0.1
printf "escaneo tipoi -sA: \n" 
sudo nmap -v - A -sS -p 7775-7778 192.168.0.1
printf "escaneo tipoi -sF: \n" 
sudo nmap -v - A -sS -p 7775-7778 192.168.0.1
```

Ejecutar el comando:
```bash
bash ejercicio8a.sh
```
Resultados:
**Opción -sS**
[ejercicio 8 opcional](https://drive.google.com/file/d/1THihdD69R-2HxvbyrgQPOdAiLbiYurq0/view?usp=sharing)
[ejercicio 8 opcional-wireshark](https://drive.google.com/file/d/1mOIw9L2WHflLOTCVHdbN4_9kWuchZPB5/view?usp=sharing)
**Opción -sA**
[ejercicio 8 opcional](https://drive.google.com/file/d/1dOwCzyz-bCgjusZDa5YYKfZGLboT1iPk/view?usp=sharing)
[ejercicio 8 opcional-wireshark](https://drive.google.com/file/d/1TpbbwlWy9bW2Zbp6JvgV_IOC3uEB_YTe/view?usp=sharing)
**Opción -sF**
[ejercicio 8 opcional](https://drive.google.com/file/d/1gfJkB75LO22UGLzLec_DjUoc-K5jVsid/view?usp=sharing)
[ejercicio 8 opcional-wireshark](https://drive.google.com/file/d/1h7CeR6Q9BOYHwsSRfqcBrAn3PrX9y5_c/view?usp=sharing)

# Opciones y parámetros de TCP
El comportamiento de la conexión TCP se puede controlar con varias opciones que se incluyen en la cabecera en los mensajes SYN y que son configurables en el sistema operativo por medio de parámetros del kernel.

### Ejercicio 9
Con ayuda del comando sysctl y la bibliografía recomendada, completarar la  Carametro l tabla s que permiten modificar algunas opciones dee e sie nli ra ar ee cnresa al a de ruae  ua ru a la e  a otear la  TCP:
Para poder ver los valores de esos parámetros se utiliza la siguiente sentencia:
```bash
sudo cat /proc/sys/net/ipv4/tcp_window_scaling
```
|Parametro del kernel| Proposito | Valor por defecto |
|--|--|--|
| net.ipv4.tcp_window_scaling | Memoria reservada para enviar al buffer los paquetes TCP | 1 |
| net.ipv4.tcp_timestamps | Es un booleano que nos indica si queremos reducir o no, el rendimiento relacionado con las marcas del tiempo | 1 |
| net.ipv4.tcp_sack | Habilita el reconocimiento (SACKS) | 1 |

### Ejercicio 10 
Abrir el servidor en el puerto 7777 y realizar una conexión desde la VM cliente. Con ayuda de wireshark estudiar el valor de las opciones que se intercambian durante la conexión. Variar algunos de los parámetros anteriores (ej. no usar ACKs selectivos) y observar el resultado en una nueva conexión.
**VM2 (default value)**
[Imagen ejercicio 10](https://drive.google.com/file/d/1SgjlAYM41LLsuR33M9_U_3xj3B6JRFpW/view?usp=sharing)
**VM2 (opposite value)**
[Imagen ejercicio 10](https://drive.google.com/file/d/1UnwrALYwCjB-GVY0Sf4_8-kWo0IJ_RRJ/view?usp=sharing)

**Análisis del resultado**
Como se puede ver las ventanas que se estipulan son menor antes de habilitar las opciones:
-   Window size scaling factor (habilitado) 128 (deshabilitado) -2
-   Window size value (habilitado) 229 (deshabilitado) 29200
-   Options (habilitado) ninguno (deshabilitado) No Operation (NOP) Timestamps
### Ejercicio 11 
Con ayuda del comando sysctl y la bibliografía recomendada, completar la siguiente tabla con parámetros que permiten configurar el temporizador  _keepalive_:
Para poder ver los valores de esos parámetros se utiliza la siguiente sentencia:
```bash
sudo cat /proc/sys/net/ipv4/tcp_keepalive_time
```

|Parametro del kernel| Proposito | Valor por defecto |
|--|--|--|
| net.ipv4.tcp_keepalive_time | El intervalo que hay entre el último paquete ack enviado y el primer keepalive probes. | 7200 |
| net.ipv4.tcp_keepalive_probes | El número de paquetes no reconocidos que se envían antes de considerar una conexión por terminada | 9 |
| net.ipv4.tcp_keepalive_intvl | El intervalo entre la secuencia de keepalive probes independientemente de si se ha intercambiado algo entre medias. | 75 |

# Traducción de direcciones NAT y reenvío de puertos 
En esta sección supondremos que la red que conecta Router (VM3) con VM4 es pública y que no puede encaminar el tráfico 192.168.0.0/24. Además, asumiremos que la IP de Router es dinámica.

### Ejercicio 12
Configurar la traducción de direcciones dinámica en Router:

- **(VM3, Router)** Configurar Router para que haga SNAT (_masquerade_) sobre la interfaz eth1 usando el comando iptables.
```bash
sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

- **(VM1)** Comprobar la conexión entre VM1 y VM4 con la orden ping.
```bash
ping 172.16.0.1
```
-  **(Router)** Analizar con Wireshark el tráfico intercambiado, especialmente los puertos y direcciones IP origen y destino en ambas redes
**Antes de aplicar la regla**
[ejercicio 12 (VM1 eth0-VM3 eth0 y eth1-VM4 eth0) - antes](https://drive.google.com/file/d/15v9GjWXAcMGC3dWQR2cQOaOthmkf5HT5/view?usp=sharing)
**Después de aplicar la regla**
[ejercicio 12 (VM1 eth0-VM3 eth0 y eth1-VM4 eth0) - después](https://drive.google.com/file/d/1ePkBZidQlSeU6ZtwxcQF1O007D4pE4MR/view?usp=sharing)

### Ejercicio 13
¿Qué parámetro se utiliza, en lugar del puerto origen, para relacionar las solicitudes con las respuestas? Comprueba la salida del comando conntrack -L o, alternativamente, el fichero /proc/net/nf_conntrack.
```bash
sudo cat /proc/net/nf_conntrack
```
[Imagen ejercicio 13](https://drive.google.com/file/d/1IRg-_34L_84-4VF5FenzUUr5w2fqRMN4/view?usp=sharing)

### Ejercicio 14
Acceso a un servidor en la red privada:
Para mostrar las reglas iptables de prerouting:
```bash
sudo iptables -L -n -t nat
```
-   **(Router)** Usando iptables, reenviar las conexiones (DNAT) del puerto 80 de Router al puerto 7777 de VM1. Iniciar una captura de Wireshark en los dos interfaces.
```bash
sudo iptables -t nat -A PREROUTING -d 172.16.0.1 -p tcp --dport 80 -j DNAT --to 192.168.0.1:7777
```
-   **(VM1)** Arrancar el servidor en el puerto 7777 con nc.
```bash
 nc -l 7777
 ```
-  **(VM4)** Conectarse al puerto 80 de Router con nc y comprobar el resultado en VM1.
```bash 
nc -l 192.168.0.1 80
```
-  **(Router)** Analizar con Wireshark el tráfico intercambiado, especialmente los puertos y direcciones IP origen y destino en ambas redes.

La máquina VM3 (puerto 80) redirige los paquetes que la máquina VM4 envía a la máquina VM1 (puerto 7777)
[Imagen ejercicio 14 - router](https://drive.google.com/file/d/1HtmR3Fb7SdwAXz94-r3z6UWVMxEb4ICy/view?usp=sharing)

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3OTgwMDc3MTgsLTE1NTg0NDQ0NjMsMT
UwMzM2MjE0OSwtMTQwMjczMTc4OCwtMTg4MTg5NDQwNV19
-->