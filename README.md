# Laboratorio #1 - Administración de Redes 
## Juan José Cifuentes Cuellar
## Tomas Candelo Montoya
## Carlos Farouk Abdalá Rincón 

# Planificación
## Preguntas

**¿Cuántas subredes se necesitan?**  

**¿Cuántos host/interfaces necesita la subred, y que dispositivos se encuentran en ella?**  

**¿Qué partes de la red utilizan direcciones publicas y que partes direcciones privadas?**    

**¿Dónde deberían estar conservadas estas direcciones?** 

**¿Se requiere una asignación dinámica y/o estática? ¿Donde?**

**¿En que terminal se configuraron los servicios requeridos?**

**¿Qué servicio de IPv4 se debe configurar para para limitar el acceso al servidor web por el puerto 80 y 443 en las vlans especificas?**

**¿En que interfaces se deben configurar las OSPF o EIGRP (no RIP) y/o rutas estáticas?**

## Subneteo

La red empresarial cuenta con un total de 5 espacios de red

• La intranet de INTRANETotá 2001:1200:A1::/48 para un total de 3 vlans (50 Ing, 100 Tech, 99 native)

• La intranet de Madrid 2001:1200:B1::/48 para un total de 3 vlans (25 Tesoreria, 100 Vice, 1 native)

• La Conexión del Router de INTRANETotá con el ISP de la misma ciudad y el espacio DMZ de la red (que cuenta con un total de 6 ) 2001:1200:C1::/48 

• La Conexión del Router de España con el ISP del mismo pais 2001:1200:D1::/48 

• Las Conexiones entre los routers que hacen parte del espacio de internet, este espacio de direcciones a diferencia de los cuatro anteriores es un espacio Ipv4 con el siguiente espacio de red 190.85.201.0/24

Para el proceso de subneteo en las redes IPv6 no se tiene informacion de cuantos host se necesitan por cada subred por lo que su proceso de subneteo se basa en que estamos buscando direcciones con un inteface ID de longitud 64, mientras que para el subneteo del espacio IPv4 se tiene en cuenta que necesitamos 2 host por cada subred (conexion entre routers) y se tiene un total de 5 conexiónes. 

**Subneteo de la intranet de INTRANETotá**

![SUBNETEO DE LA INTRANET DE INTRANETOTÁ](/image/Subneto_inteNet_INTRANET.png)

**Subneteo de la intranet de Madrid**

![SUBNETEO DE LA INTRANET DE MADRID](/image/Subneto_inteNet_MAD.png)

**Subneteo del DMZ y WAN entre R1_INTRANET y ISP_INTRANET**

![SUBNETO DE LA RED DMZ Y WAN ENTRE R1_INTRANET Y ISP_INTRANET](/image/Subneteo_DMZ_R1-ISP.png)

**Subneteo del ISP_ESP**

![SUBNETO DE LA RED ISP DE ESPAÑA](/image/Subneteo_R2-ISP_MAD.png)


Para el subneteo de los espacios de red IPv6 se realizo el mismo proceso para los 4 espacios, como estandar buscamos una longitud de interfaz de 64 bits por lo que en este caso el número de bits que hacen falta para cumplir este estandar es de 16 bits (64-48=16) lo que seria igual a un cuarteto de la dirección el cual va a ser nuestro identificador de subred, una vez con esto sacamos las primeras 15 posibles subredes para cada espacio, unicamente para ver las primeras posiblidades que tenemos y solo seleccionamos la cantidad de reubredes necesarias por el espacio de red:

• La intranet de INTRANETota necesita un total de 3 subredes

• La intranet de Madrid de igual manera necesita 3 subredes

• El DMZ y la conexión WAN entre el R1_INTRANET y el ISP_INTRANET necesita 2 subredes 

•La conexión Wan entre el R2_ESP y el ISP_ESP necesita una sola subred

**Subneteo del espacio WAN**

![SUBNETO DEL ESPACIO WAN(INTERNET)](/Image/Subneto_internet.png)

Para este Subneteo el número de host pedidos por las especificaciones de la red es de 2 lo que da un total de 2 bits de reserva, lo que da una nueva mascara de subred con identificador 30 y un incremento de 4 en el cuarto octeto de la dirección. En este caso son necesarios 5 rangos de red para las conexiones entre los routers que pertenecen al internet

## Tabla de direccionamiento

Finalmente con los Subneteos terminados pasamos a la construcción de la tabla de enrutamiento en la cual asignamos las direcciones IP a los diferentes dispositivos de la red y en los puertos que deben de ser asignadas, el resultado de esas asignaciones fue la siguiente tabla:

![Tabla de direccionamiento en la red empresarial](/image/TABLA_DIRECCIONAMIENTO.png)

# Configuración

Para todos los dispositivos intermedios(Routers,switches) se hace la misma configuración básica, en la que se le asigna un nombre, un mensaje de bienvenida y contraseñas de acceso tanto para entrar a su configuración (por cable de consola o por telnet) como para acceder a sus diferentes niveles administrativos.

Los comandos usados para estas configuraciones basicas fueron los siguietes

```
Switch(config)#hostname SW1_INTRANET
SW1_INTRANET(config)#line console 0
SW1_INTRANET(config-line)#password cisco
SW1_INTRANET(config-line)#login
SW1_INTRANET(config-line)#exit
SW1_INTRANET(config)#
SW1_INTRANET(config-line)#password cisco
SW1_INTRANET(config-line)#login
SW1_INTRANET(config-line)#exit
SW1_INTRANET(config)#banner motd #Se encuentra configurando el switch 1 en Bogootá. Ingrese su contrasena#
SW1_INTRANET(config)#enable password cisco
```

Ademas de esto, tanto la intranet de Bogotá como la de Madrid, tenemos que permitir el paso y uso de las direcciones IPv6 para eso a los switches se les realizó la siguiente configuración 

```
SW1_INTRANET(config) SDM PREFER DUAL-IPV4-AND-IPV6 DEFAULT
SW1_INTRANET(config) END
SW1_INTRANET# reload
```

Con esta configuración se permite que los switches puedan utilizar y reconocer direcciones IPv6 

En el caso de los routers unicamente tenemos que hacer es habilitar el propio router como un router IPv6, el cual podra enviar y recibir paquetes IPV6, rutas estaticas o protocolos de enrutamiento IPV6, el comando es el siguiente:

```
R1_BOG(config) IPv6 unicast-routing
```

## Creacion de las vlans

Lo primero que se va a hacer para la configuración de las vlans es asiganr la default gateway, la cual es la direccion vinculada con la subinterfaz del router que tiene la vlan que usaremos de troncamiento (En el caso de Bogotá la vlan 99 y en el caso de Madrid la vlan 1) como se puede ver en la tabla de enrutamiento, El comando es el siguiente:

```
SW1_INTRANET(config) ipv6 route ::/0 2001:1200:A1:1::1
```

Ahora, apagamos todas sus interfaces.
```
SW1_INTRANET(config)#interface range fa0/1-24
SW1_INTRANET(config-if-range)#shutdown
SW1_INTRANET(config-if-range)#exit
```

Una vez desactivadas las interfaces desigmaos el rango de la fastEthernet 1 a la 5 como interfaces troncales las cuales estaran vinculadas a la vlan nativa de cada intranet y las encendemos 

```
SW1_INTRANET(config)#interface range fa0/1-5
SW1_INTRANET(config-if-range)#switchport mode trunk
SW1_INTRANET(config-if-range)#switchport trunk native vlan 99
SW1_INTRANET(config-if-range)#no shutdown 
SW1_INTRANET(config-if-range)#exit
```

Ahora creamos las vlans necesarias y asignamos su nombre a cada una

```
SW1_INTRANET(config)#vlan 50
SW1_INTRANET(config-vlan)#name Ing
SW1_INTRANET(config-vlan)#vlan 100
SW1_INTRANET(config-vlan)#name Tech
SW1_INTRANET(config-vlan)#vlan 99
SW1_INTRANET(config-vlan)#name Native
SW1_INTRANET(config-vlan)#exit
```

Con las vlans creadas determinamos a que rangos de interfaces le vamos a asignar la respectiva vlan, como la guia de laboratorio no especifica que interfaces corresponde a cada vlan, las asignamos de manera equitativa entre las restantes

```
SW1_INTRANET(config)#interface range fa0/6-15
SW1_INTRANET(config-if-range)#Switchport access vlan 50
SW1_INTRANET(config-if-range)#exit
SW1_INTRANET(config)#interface range fa0/16-24
SW1_INTRANET(config-if-range)#Switchport access vlan 100
SW1_INTRANET(config-if-range)#exit
SW1_INTRANET(config)#interface range fa0/1-5
SW1_INTRANET(config-if-range)#Switchport access vlan 99
SW1_INTRANET(config-if-range)#exit
```

Finalmente le asignamos la direccion IPv6 a las interfaces de la vlan 99 en cada uno de los switches con los siguientes comandos 

```
SW1_INTRANET(config)#interface vlan 99
SW1_INTRANET(config-if)#ipv6 address 2001:1200:A1:1::2/64
SW1_INTRANET(config-if)#exit
```

No olvidar que tenemos que reactivar todas las interfaces restantes del switch y guardar los cambios 

```
SW1_INTRANET(config)#interface range fa0/6-24
SW1_INTRANET(config-if-range)#no shutdown
SW1_INTRANET(config-if-range)#exit
SW1_INTRANET(config)#exit
SW1_INTRANET#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```

