---
title: "HackTheBox: Máquina Backdoor"
published: true
date: 2024-04-14 20:37:00
tags: [CTF, HTB, Metasploit, Ethical Hacking, Cybersecurity]
image: /blog/assets/htb_backdoor/portada.png
---

<br>
![](/blog/assets/htb_backdoor/portada.png)
<br>

<br>
### Descripción
* * *
<br>

En esta ocasión, realizaremos un juego CTF a una máquina extremadamente sencilla, o eso parece, la cual se llama “Backdoor”. Usaremos herramientas como `Metasploit Framework` (MSF). 

**Nota**: No recomendamos usar MSF en ambientes reales, porque son herramientas automáticas de las cuales no tenemos control y pueden, durante un pentesting, corromper los archivos de la tecnología que estamos probando.


### Enlace de Lab:
[HTB: Backdoor](https://www.hackthebox.com/machines/backdoor)


### Dificultad:
FÁCIL


### Solución
* * *
<br>

Lo primero es conectarnos a HTB VPN con el siguiente comando y presionando ENTER.

<br>
![](/blog/assets/htb_backdoor/paso_1.png)
<br>


Verificamos que efectivamente nos hemos conectado a HTB.

<br>
![](/blog/assets/htb_backdoor/paso_2.png)
<br>


Vemos la IP de la máquina a la cual realizaremos un CTF.

<br>
![](/blog/assets/htb_backdoor/paso_3.png)
<br>

Observamos nuestra IP dentro del entorno HTB (10.10.14.42).

<br>
![](/blog/assets/htb_backdoor/paso_4.png)
<br>

Ahora haremos un PING a la máquina, en principio, por dos razones:
1. Ver si efectivamente estamos conectados.
2. Reconocer el sistema operativo que atacaremos.

<br>
![](/blog/assets/htb_backdoor/paso_5.png)
<br>

Efectivamente, estamos recibiendo paquetes de la 10.10.11.125, que es nuestra máquina a atacar. Además, nos fijamos en un segundo campo, el cual es TTL (Time To Live), con ese valor, se puede identificar el sistema operativo gracias a este rango:

`64` -> pertenece a Linux

`128` -> pertenece a Windows

En este caso, arroja un 63, esto debido al aislamiento de HTB que se pierde un valor, pero por proximidad podemos determinar que es Linux.

Ahora vamos a identificar los puertos de nuestra máquina a atacar con nmap con el siguiente comando en modo root:

```bash
nmap -n -p- -T5 -v -sCV 10.10.11.125
```

Donde cada flag significa:

`-n`: evitar name resolution, en este caso no es necesario y se tarda bastante.

`-p-`: analiza todos los puertos existentes, por defecto nmap sin este flag analiza el TOP 1000 ports.

`-T5`: velocidad de escaneo, siendo 1 el modo paranoico “indetectable” y 5 el más rápido, a mayor velocidad más ruidoso y detectable eres. Aprovechando que estamos en un espacio controlado, lo ponemos al máximo.

`-v`: va mostrando resultados mientras vamos escaneando. Usado para agilizar el proceso, ya que cada vez que detecta un puerto, no espera hasta al final para dar resultados, sino al instante en que los encuentra.

El nmap nos retorna estos resultados:

<br>
![](/blog/assets/htb_backdoor/paso_6.png)
<br>

`Puerto 80`: mayormente es el puerto estándar para HTTP, es decir, páginas web. Si en el navegador colocamos la IP de la máquina, nos llevará a un WordPress, el cual no luce interesante, es un callejón sin salida. Abundaríamos más, pero no es una máquina de tecnologías webs, sino de backdoors.

`Puerto 22`: es el SSH, en palabras sencillas es acceder de forma remota al ordenador, con todos los permisos y facilidades de como si estuviésemos físicamente en el ordenador.

`Puerto 1337`: este luce interesante, en realidad solo dice waste? No identifica nada, así que buscaremos ese puerto en internet y veremos de qué se trata y para qué se utiliza.

Después de buscar en internet esta tecnología descubrimos que se podía hacer backdoor, así que este es nuestro objetivo. Precisamente, el exploit se llama `gdb_server_exec` y está disponible en `MetaSploit Framework` (MSF).

Así que iniciaremos MSF con el siguiente comando. 

<br>
![](/blog/assets/htb_backdoor/paso_7.png)
<br>


Una vez iniciado, ejecutaremos el siguiente comando que nos arroja un resultado:

<br>
![](/blog/assets/htb_backdoor/paso_8.png)
<br>

```bash
use 0
```
Ya que en la columna # está identificado como un 0 y es el que queremos usar.

Lo primero es conocer el exploit y ver qué pide con el comando:

```bash
show options
```

Donde pide los siguientes valores

<br>
![](/blog/assets/htb_backdoor/paso_9.png)
<br>

Los rellenaremos todos con el siguiente formato:

```bash
set [campo] [valor]
```
Teniendo en cuenta que la LHOST, que es la `máquina nuestra`, no es la IP local (192.168.x.x) sino la que se nos asignó en la network interface tun0.

Existen dos comandos especiales que siempre, o casi siempre (en el 99,9% de los casos) vendrán con los exploits y hay que configurarlos, los cuales son payloads y targets. Para acceder a este menú debemos escribir:

```bash
show payloads
```

y desplegará una lista de payloads. En este caso usaremos el payload `Linux/x64/meterpreter/reverse_tcp` porque la máquina es Linux (TTL->63) y por prueba de ensayo y error pudimos ver que era de 64 bits su arquitectura. Usaremos una consola meterpreter, es la Shell por excelencia para realizar exploits, con opciones a post explotación, facilidad para escalar privilegios y mucha flexibilidad.

Y lo mismo realizaremos con targets:

```bash
show targets
```
y desplegará una lista, en este caso más pequeña, y usaremos la `x86_64 (64-bit)`, es decir, nuestro objetivo será una máquina con una arquitectura de 64 bits.

<br>
![](/blog/assets/htb_backdoor/paso_10.png)
<br>

Ejecutaremos con `run` o `exploit` y veremos los siguientes resultados, realizaremos un `ls` para que despliegue los archivos del directorio y veremos la flag `user.txt`.

<br>
![](/blog/assets/htb_backdoor/paso_11.png)
<br>

Luego, para tener una Shell en condiciones haremos lo siguiente:

```bash
shell
python3 -c "import pty; pty.spawn('/bin/bash')"
```
y si aplicamos un `whoami` obtendremos una respuesta `user`.

Ahora es momento de obtener root, para eso, simplemente bastan los siguientes comandos.

```bash
export TERM=xterm
/usr/bin/screen -x root/root
```

Luego un `whoami` y ya veremos que somos root, solo nos queda ver la flag y terminamos.

<br>
![](/blog/assets/htb_backdoor/paso_12.png)
<br>