---
title: "dockerlabs.es: máquina Agua de Mayo"
published: true
date: 2024-05-13 22:10:00
tags: [Nmap, Vulnerability Assessment, Kali Linux, SSH, Apache Server, Network Scanning, Bettercap, Privilege Escalation, Security Best Practices, Debian Linux, Penetration Testing Tools, Web Server Security, Cybersecurity, Ethical Hacking]
image: /blog/assets/dockerlabs.es_agua_de_mayo/portada.png
---


<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/portada.png)
<br>


### Descripción
* * *
<br>
Esta máquina virtual creada con `Docker` tiene de todo un poco para que practiques tus habilidades de hacking. Está equipada con un servidor web Apache, más algunos servicios básicos como SSH en el puerto 22 y HTTP en el puerto 80, listos para ser desafiados.


### Enlace de Lab:
[dockerlabs.es: Agua de Mayo](https://dockerlabs.es/).




### Dificultad:
Fácil.












### Solución
* * *
<br>




Para iniciar el proceso de `vulnerar` esta máquina, lo primero que haremos es con ayuda de `nmap` ver a qué nos estamos enfrentando.




``` shell
nmap -p- -Pn  -sS --min-rate 5000 --open 172.17.0.2  -oX report.xml -v
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
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_2.JPG)
<br>




Nos fijamos que tiene varios puertos abiertos.




`puerto 22`: servicio SSH




`puerto 80`: servicio http






Ahora es momento de ver con qué versiones nos estamos enfrentando de cada servicio. Para ello implementamos el código:




```shell
nmap -sCV -p21,22,80 172.17.0.2 -oN ports_version
```




Donde:




**`-sC`**: Esta opción le dice a `nmap` que ejecute scripts predeterminados contra los puertos encontrados.
 **`-sV`**: Le indica a `nmap` que intente determinar la versión de los servicios que se están ejecutando en los puertos abiertos.




>*sCV es la combinación de ambos comandos*.






<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_3.JPG)
<br>




Lo primero que haremos es ver los posibles vectores de ataque. Lamentablemente el `puerto 22` perteneciente a `ssh` no es vulnerable. Nos queda ver si hay alguna fuga de información en el servicio `http`


Accedemos a la página web del `puerto 80` aparenta ser una configuración por defecto de `Apache2` pero si vamos al código fuente nos percatamos de que hay un código que pertenece a `Brainfuck`.




```brainfuck
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
```


La pasamos por un traductor de código `brainfuck` a `ascii` y este será el resultado.


```text
bebeaguaqueessano
```


Aparentemente ya tenemos un usuario para ingresar mediante `ssh`.


Ahora hacemos un fuzzing web con ayuda de `dirbuster` y obtenemos los siguientes resultados


<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_6.JPG)
<br>


Descargamos la imagen,y aplicando técnicas de `esteganografía` para asegurarnos. Tratamos de ver si tiene alguna información o un `txt` dentro de esta. Pero no, tomamos el nombre de la imagen `agua_ssh` y tomamos la parte de `agua` como usuario y `bebeaguaqueessano` como clave y vemos que funciona


<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_7.JPG)
<br>


Hacemos un `id` y vemos que pertenecemos al grupo `lxd`. Posiblemente explotando esta vulnerabilidad podamos escalar privilegios.


Si hacemos un `ls` en el directorio veremos que tenemos una archivo llamado `alpine-v3.13-x86_64-20210218_0139.tar.gz` que hace que sea casi seguro que debamos explotar el lxd. Pero no, esto no es más que un `rabbit hole` [^1]


Si realizamos un `sudo -l` vemos que podemos ejecutar como `root` el `/usr/bin/bettercap`.


<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_8.JPG)
<br>


Abrimos este binario como sudo y ejecutamos el comando `?`. En la ayuda veremos que si corremos un comando iniciando con `!` correrá comandos del sistema. Entonces corremos


```bash
! whoami
```


Y recibiremos un `root`


Pues, en este momento para escalar de una forma más profesional corremos lo siguiente:


```bash
! chmod +s /bin/bash
```

<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_9.JPG)
<br>


Salimos de `bettercap`, luego, hacemos un `/bin/bash -p` y ya seremos usuarios `root`

<br>
![](/blog/assets/dockerlabs.es_agua_de_mayo/paso_10.JPG)
<br>


Y eso sería todo en esta máquina.


### _Referencias_
* * *
[^1]: [¿Que es un Rabbit Hole?](https://medium.com/@luke.williams1248/on-pen-testing-rabbit-holes-and-how-to-avoid-them-ed7b93cdfaa5)

