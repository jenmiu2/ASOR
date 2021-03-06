# Practica 1.5: RIP y BGP

# Objetivos
En esta práctica se afianzan los conceptos elementales del encaminamiento IP. En particular, se estudia un protocolo de encaminamiento interior y otro exterior: RIP (_Routing Information Protocol_) y BGP (_Border Gateway Protocol_).

Existen muchas implementaciones de los protocolos de encaminamiento. En esta práctica vamos a utilizar Quagga, que actualmente implementa RIP (versiones 1 y 2), RIPng, OSPF, OSPFv3, IS-IS y BGP. Quagga está estructurado en diferentes servicios (uno para cada protocolo) controlados por un servicio central (Zebra) que hace de interfaz entre la tabla de encaminamiento del  _kernel_  y la información de encaminamiento de los protocolos individuales.

Todos los ficheros de configuración han de almacenarse en el directorio /etc/quagga. La sintaxis de estos ficheros es sencilla y está disponible en  [http://quagga.net](http://quagga.net/). Revisar especialmente la correspondiente a RIP y BGP en  [https://www.quagga.net/docs/quagga.html](https://www.quagga.net/docs/quagga.html). Además, en /usr/share/doc/quagga-0.99.22.4 hay ficheros de ejemplo.

# Protocolo interior: RIP
## Preparación del entorno
```c
netprefix inet
machine 1 0 0 1 3 2 4
machine 2 0 0 1 1 2 5
machine 3 0 2 1 1 2 6
machine 4 0 2 1 3 2 7
```
|  Maquina | Interfaz | Red Interna | Dir. de red | Dir. IP
|--|--|--|--|--|
| Router1  | eth0 eth1 eth2  | inet0 inet3 inet4 | 172.16.0.0/16 172.19.0.0/16 192.168.0.0/24 | 172.16.0.1 172.19.0.1 192.168.0.1 |
| Router2  | eth0 eth1 eth2  | inet0 inet1 inet5 | 172.16.0.0/16 172.17.0.0/16 192.168.1.0/24 | 172.16.0.2 172.17.0.2 192.168.1.2 |
| Router3  | eth0 eth1 eth2  | inet2 inet1 inet6 | 172.18.0.0/16 172.17.0.0/16 192.168.2.0/24 | 172.18.0.3 172.17.0.3 192.168.2.3 |
| Router4  | eth0 eth1 eth2  | inet2 inet3 inet7 | 172.18.0.0/16 172.19.0.0/16 192.168.3.0/24 | 172.18.0.4 172.19.0.4 192.168.3.4 |

### Ejercicio 1
Configurar todos los encaminadores según la figura y tabla anterior. Después, comprobar:

- Que los encaminadores adyacentes son alcanzables, por ejemplo, Router1 puede hacer  _ping_  a Router2 y Router4.

- Que la tabla de encaminamiento de cada encaminador es la correcta e incluye una entrada para cada una de las tres redes a las que está conectado.

Además, activar el  _forwarding_  de paquetes IPv4 igual que en la práctica 1.1.

## Configuración del protocolo RIP
### Ejercicio 2
Configurar RIP en todos los encaminadores para que intercambien información:
- Crear un fichero ripd.conf en /etc/quagga con el contenido que se muestra a continuación.
- Iniciar el servicio RIP (y Zebra) con service ripd start.
Contenido del fichero /etc/quagga/ripd.conf:
```c
#Activar el encaminamiento por RIP
router rip
#Definir la versión del protocolo que se usará
version 2
#Habilitar información de encaminamiento en redes asociadas a los interfaces
network eth0
network eth1
network eth2
```

### Ejercicio 3
Consultar la tabla de encaminamiento de RIP y de Zebra en cada encaminador con el comando vtysh. Comprobar también la tabla de encaminamiento del  _kernel_  con el comando ip.
```bash
sudo vtysh -c "show ip rip"
sudo vtysh -c "show ip route"
ip route
```
### Ejercicio 4
Con la herramienta wireshark, estudiar los mensajes RIP intercambiados, en particular:
- Encapsulado.
- Direcciones origen y destino.
-  Campo de versión.
- Información para cada ruta: dirección de red, máscara de red, siguiente salto y distancia.

### Ejercicio 5
Eliminar el enlace entre Router1 y Router4 (por ejemplo, desactivando el interfaz eth1 en Router4). Comprobar que Router1 deja de recibir los anuncios de Router4 y que, pasados aproximadamente 3 minutos (valor de  _timeout_  por defecto para las rutas), ha reajustado su tabla.

### Ejercicio 6 (Opcional)
Los servicios de Quagga pueden configurarse de forma interactiva mediante un terminal (telnet), de forma similar a los encaminadores comerciales. Configurar ripd vía VTY:

- Añadir “password asor” al fichero ripd.conf, desactivar el protocolo (no router rip) y comentar el resto de entradas. Una vez cambiado el fichero, reiniciar el servicio.

- Conectar al VTY de ripd y configurarlo. En cada comando se puede usar ? para mostrar la ayuda asociada.
<Insertar foto>
**Nota:** Para poder escribir la configuración en ripd.conf, el usuario quagga debe tener los permisos adecuados sobre el fichero. Para cambiar el propietario del fichero, ejecutar el comando:
```c
chown quagga:quagga /etc/quagga/ripd.conf
```

# Protocolo exterior: BGP

## Preparación del entorno
```c
netprefix inet
machine 1 0 0
machine 2 0 0 1 1
machine 3 0 1
```
|  Maquina | Interfaz | Red Interna | Dir. de red | Dir. IP
|--|--|--|--|--|
| Router1  | eth0 | inet0 | 2001:db8:200:1::/64 | 2001:db8:200:1::1 |
| Router2  | eth0 eth1  | inet0 inet1| 2001:db8:200:1::/64 2001:db8:200:2::/64 | 2001:db8:200:1::2  2001:db8:200:2::2 |
| Router3  | eth0 | inet1 | 2001:db8:200:2::/64 | 2001:db8:200:2::3 

## Configuración del protocolo BGP
### Ejercicio 8
Consultar la documentación de las clases de teoría para determinar el tipo de AS (_stub_,  _multihomed_  o  _transit_  ) y los prefijos de red que debe anunciar. Suponed que el RIR ha asignado a cada AS prefijos de longitud 48 y que los prefijos anunciados deben agregarse al máximo.
|Numero de AS| Tipo  | Prefijos agregados |
|--|--|--|
|  |  |  |
### Ejercicio 9
Configurar BGP en los encaminadores para que intercambien información:

- Crear un fichero bgpd.conf en /etc/quagga usando como referencia el que se muestra a continuación.

- Iniciar el servicio BGP (y Zebra) con service bgpd start.

Por ejemplo, el contenido del fichero /etc/quagga/bgpd.conf de Router1 en el AS 100 sería:
```bash
# Activar el encaminamiento BGP en el AS 100

router bgp 100

# Establecer el identificador de encaminador BGP

bgp router-id 0.0.0.1

# Añadir el encaminador BGP vecino en el AS 200

neighbor 2001:db8:200:1::2 remote-as 200

# Empezar a trabajar con direcciones IPv6

address-family ipv6

# Anunciar un prefijo de red agregado

network 2001:db8:100::/47

# Activar IPv6 en el encaminador BGP vecino

neighbor 2001:db8:200:1::2 activate

# Dejar de trabajar con direcciones IPv6

exit-address-family
```

### Ejercicio 10
Consultar la tabla de encaminamiento de BGP y de Zebra en cada encaminador con el comando vtysh. Comprobar también la tabla de encaminamiento IPv6 con el comando ip.
```bash
sudo vtysh -c "show ip rip"
sudo vtysh -c "show ip route"
ip route
```

### Ejercicio 11
Con ayuda de la herramienta wireshark, estudiar los mensajes BGP intercambiados (OPEN, KEEPALIVE y UPDATE).

<!--stackedit_data:
eyJoaXN0b3J5IjpbODc2OTE4MDMxXX0=
-->