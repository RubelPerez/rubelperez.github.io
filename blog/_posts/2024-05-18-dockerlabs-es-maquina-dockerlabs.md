---
title: "dockerlabs.es: máquina Dockerlabs"
published: true
date: 2024-05-18 10:10:00
tags: [DirBuster, Web Application Brute Forcing, Directory Enumeration, File Upload Vulnerability, Reverse Shell, Netcat, Privilege Escalation, Linux, Sudo Exploit, CTF, Capture The Flag, Penetration Testing, Kali Linux, Information Security, Cybersecurity, Post-Exploitation, Root Access, File Enumeration, Command Injection, Brute Force Attack, CTF Writeup, Exploit Development, Shell Access, Linux Privilege Escalation]
image: /blog/assets/dockerlabs.es_dockerlabs/portada.png
---




<br>
![](/blog/assets/dockerlabs.es_dockerlabs/portada.png)
<br>




### Descripción
* * *
<br>
En este writeup, detallamos nuestro proceso de vulneración de una máquina en nuestro laboratorio de pentesting. Utilizamos herramientas como DirBuster para realizar una enumeración de directorios y archivos sensibles. Descubrimos una vulnerabilidad en la subida de archivos que nos permitió ejecutar un shell reverso con Netcat. Posteriormente, logramos una escalada de privilegios en un sistema Linux mediante la explotación de configuraciones incorrectas de sudo. Este writeup cubre todas las etapas del ataque, desde la recolección de información hasta la obtención de acceso root, proporcionando una visión completa de las técnicas y herramientas empleadas en el proceso.


### Enlace de Lab:
[dockerlabs.es: Dockerlabs](https://www.dockerlabs.es/).




### Dificultad:
Fácil.












### Solución
* * *
<br>




Para iniciar el proceso de `vulnerar` esta máquina, lo primero que haremos es con ayuda de `nmap` ver a qué nos estamos enfrentando.




``` shell
nmap -p- -Pn  -sS --min-rate 5000 --open 172.17.0.2 -oX report.xml -v
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
![](/blog/assets/dockerlabs.es_dockerlabs/paso_1.JPG)
<br>




Obtenemos 1 puerto abierto, `80 -> http`  .




Ahora es momento de ver con qué versiones nos estamos enfrentando de cada servicio. Para ello implementamos el código:




```shell
nmap -sCV -p80 172.17.0.2-oN ports_version
```




Donde:




**`-sC`**Esta opción le dice a `nmap` que ejecute scripts predeterminados contra los puertos encontrados.
 **`-sV`**: Le indica a `nmap` que intente determinar la versión de los servicios que se están ejecutando en los puertos abiertos.




>*sCV es la combinación de ambos comandos*.


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_2.JPG)
<br>


Nos fijamos que tiene un puerto abierto.




`puerto 80`: servicio http








Lo primero que haremos es ver los vectores de entrada que tiene el `puerto 80`, vemos que el servicio `Apache` a dia de hoy no tiene ninguna vulnerabilidad. Así que nos tocará navegar en la web para ver la fuga de información.






<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_3.JPG)
<br>


Vemos que es una web bastante similar a la web principal de `dockerlabs.es`. No hay nada relevante en el `source code`, así que nos tocaría hacer un fuzzing web.


En este caso utilizaremos `dirb buster`, una herramienta para hacer fuzzing web de una manera un tanto visual, simplificando nuestra actividad.




Lo configuramos de la siguiente forma e iniciamos el escaneo


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_4.JPG)
<br>


Encontramos varias rutas interesantes:




<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_5.JPG)
<br>




`uploads`: Es una carpeta donde se almacenan archivos que posiblemente se suban por alguna ruta.


`uploads.php`: Es un archivo php el cual no puede ser leído, es posiblemente el script intermediario encargado de subir archivos a `uploads`


`machine.php`: El más interesante de los 3, consta de un formulario en el cual se pueden subir archivos que se almacenarán en el directorio `uploads`


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_6.JPG)
<br>


Este es el contenido de `machine.php` el cual nos pide que ingresemos un archivo a subir. Intentamos subir un archivo `txt` y nos dira el error de que `solo se permiten archivos zip`




Como evidentemente estamos frente a una posible `Arbitrary File Upload Vulnerability`.


Abrimos `burp suite` para simplificar la tarea, subimos un archivo. Capturamos la request y la enviamos al `repeater`. Allí modificaremos extensiones hasta que encontremos una diferente a `zip` que pueda ser interpretada por el servidor y a la vez se salte las validaciones.


Para `PHP` tenemos las siguientes `extensiones bypass` encontradas en hacktricks [^1]


`PHP`: .php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc, .hphp, .ctp, .module




Damos con la extensión correcta que es `phar` y nos creamos una `reverse shell` y la enviamos mediante `burp suite` con las siguientes características:




<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_7.JPG)
<br>




Abrimos una `shell` en modo de escucha con `nc` mediante el código:


```shell
nc -lvnp 443
```


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_8.JPG)
<br>




Vamos al directorio `uploads` y presionamos sobre el archivo `reverse.php` y cuando vayamos a nuestro `nc` en modo de escucha nos aparecerá lo siguiente:


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_9.JPG)
<br>


Ya tenemos acceso como el usuario `www-data` al servidor


Ahora es momento de tratar la bash, más que nada para poder hacer `ctrl+c` sin matar la `reverse shell` y `ctrl+l` para poder limpiar la pantalla:


```bash
script /dev/null -c bash
```


`ctrl+z`


```bash
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```


Si hacemos nano, vi o cat, la pantalla nos aparecerá pequeña y no podremos crear archivos con total libertad. Para ello, corremos lo siguiente:


Abrimos una terminal nueva y corremos:


```bash
stty size
```


<br>
![](/blog/assets/dockerlab.es_upload/bash.png)
<br>
Nos aparecerán dos números, el primero corresponde a las `rows` y el segundo a `columns`.


Abrimos la `reverse shell` y corremos el siguiente código:


```bash
stty rows 24 columns 128
```


Buscando formas para `escalar privilegios` corremos el comando `sudo -l`.


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_12.JPG)
<br>


vemos que tenemos acceso como `sudo` a unos binarios que pueden leer archivos con permisos de `root`.




Vamos a las rutas más comunes donde puede haber algo de información y en `/opt/` encontramos algo interesante:
<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_11.JPG)
<br>




Vamos a GTFObins[^2] y vemos cómo explotar estos binarios


```shell
LFILE=file_to_read
sudo cut -d "" -f1 "$LFILE"
```


Pues corremos el comando


```shell
sudo /usr/bin/cut -d "" -f1 "/root/clave.txt"
```


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_13.JPG)
<br>


Obtenemos la clave de `root` la cual es `dockerlabsmolamogollon123`




Intentamos hacer login como `root` y la `password` encontrada


<br>
![](/blog/assets/dockerlabs.es_dockerlabs/paso_14.JPG)
<br>




y somos usuarios `root`




### _Referencias_
* * *
[^1]: [hacktricks: File extension Bypass](https://book.hacktricks.xyz/pentesting-web/file-upload)


[^2]: [GTFObins](https://gtfobins.github.io/)

