---
title: "dockerlab.es: máquina Fileception"
published: true
date: 2024-04-27 16:37:00
tags: [Esteganografia, transferencia de archivos, ftp, encoding y decoding.]
image: /blog/assets/dockerlab.es_fileception/portada.png
---

<br>
![](/blog/assets/dockerlab.es_fileception/portada.png)
<br>

### Descripción
* * *
<br>
En esta ocasión, realizaremos un hack a una máquina de dificultad media en docker llamada `Fileception`, y tal vez, sepamos por que se llama así. Aprenderemos sobre `esteganografía`, la cual gira completamente esta máquina.


### Enlace de Lab:
[Dockerlabs: Fileception](https://www.dockerlabs.es/).

### Dificultad:
MEDIO.



### Solución
* * *
<br>
Estas máquinas están basadas en Docker y mediante un archivo llamado ´auto_loader.sh´ las montaremos de una forma automática, lo primero que debemos hacer es el siguiente comando:

```shell
./auto_deploy.sh fileception.tar
```
![](/blog/assets/dockerlab.es_fileception/paso_1.JPG)

Observamos que nuestra máquina está corriendo sobre la IP `172.17.0.2`, esto porque al estar en Docker por defecto corren sobre la interfaz 172.17.0.x

Dicho esto, es momento de hacer un escaneo con `nmap`, más que nada para ver con que nos estamos enfrentando. 

``` shell
nmap -p- -Pn  -sS --min-rate 5000 --open 172.17.0.2  -oX report.xml -v
```

![](/blog/assets/dockerlab.es_fileception/paso_2.JPG)

Donde: 

**`-p-`**: Esta opción le dice a `nmap` que escanee todos los puertos de 1 a 65535 en el objetivo. 
    
**`-Pn`**: Esta opción omite el descubrimiento de hosts.
    
**`-sS`**: Realiza un escaneo SYN. Este es un tipo de escaneo sigiloso que es menos probable que sea registrado por sistemas de detección de intrusos. 
    
**`--min-rate 5000`**: Esta opción configura `nmap` para enviar paquetes a una tasa mínima de 5000 paquetes por segundo.
    
**`--open`**: Le indica a `nmap` que sólo muestre los puertos que están abiertos.
      
**`-v`**: Aumenta la verbosidad del comando, lo que significa que `nmap` proporcionará más detalles sobre lo que está haciendo.

Obtenemos 3 puertos abiertos, `21 -> ftp` `22 -> ssh` y `80 -> http`.

Ahora es momento de ver con que versiones nos estamos enfrentando de cada servicio. Para ello implementamos el código:

```shell
nmap -sCV -p21,22,80 172.17.0.2 -oN ports_version
```

![](/blog/assets/dockerlab.es_fileception/paso_3.JPG)

Donde:

**`-sC`**: Esta opción le dice a `nmap` que ejecute scripts predeterminados contra los puertos encontrados. 
    
**`-sV`**: Le indica a `nmap` que intente determinar la versión de los servicios que se están ejecutando en los puertos abiertos. 

>*sCV es la combinacion de ambos comandos*.

Nos percatamos que ocurre algo interesante con el ftp, más especifico `ftp-anon: Anonymous FTP login allowed`.

Esto no es más que una característica que, gracias a FTP, sin necesidad de proporcionar contraseña podemos acceder a este servicio con el usuario `anoymous`.

Pues, eso hacemos. Accedemos al ftp con el comando:

```ssh
ftp 172.17.0.2
```

Al momento que nos pida un usuario, escribimos:

`anonymous`

Y perfecto, ya tendríamos acceso. Dentro del directorio si hacemos un `ls` veremos que hay una imagen que se llama hello_peter.jpg.

La descargamos con el comando `get`

![](/blog/assets/dockerlab.es_fileception/paso_4.JPG)

Vemos la imagen y es simplemente esto:

![](/blog/assets/dockerlab.es_fileception/paso_5.jpg)


Pues, si es todo lo que podemos extraer del servicio `FTP`, pues pasaríamos al siguiente. En el puerto 22, como suponemos que tenemos el usuario Peter podríamos hacer un ataque de fuerza bruta, pero mejor usarlo como última opción una vez hemos agotado todos los recursos.

Vamos al puerto `80` que estaba corriendo un servicio `http` de `apache`.

Si nos vamos al código fuente de la página, hay una sección comentada que dice lo siguiente:

![](/blog/assets/dockerlab.es_fileception/paso_6.JPG)


De aquí extraemos esta contraseña `@UX=h?T9oMA7]7hA7]:YE+*g/GAhM4` en un `enconding` `baseXX`. Investigando entre los diferentes encoding de base, el que funciona es el `base85` el cual nos arroja el siguiente resultado:


```text
base_85_decoded_password
```

Pero si seguimos leyendo el texto, vemos que habla de `esteganografía`. La cual según Wikipedia es:

> La esteganografía se trata del estudio y aplicación de técnicas que permiten ocultar mensajes u objetos, dentro de otros, llamados _portadores_, para que no se perciba su existencia.

Partiendo de esta definición, y como tenemos una imagen podemos ver si a lo mejor hay un archivo dentro de la imagen.

Usaremos una herramienta para estos fines; se llama `steghide` y sirve tanto para extraer archivos como para introducir uno dentro del otro.

Corremos el siguiente comando:

```shell
steghide extract -sf hello_peter.jpg
```

Al momento de que nos pida el `salvoconducto` proporcionamos la contraseña encontrada anteriormente `base_85_decoded_password`

Y vemos que efectivamente, se encontraba un archivo llamado `you_find_me.txt` dentro de la imagen.

![](/blog/assets/dockerlab.es_fileception/paso_7.JPG)

Si vemos el contenido de ese archivo. Inmediatamente veremos que tiene un formato bastante extraño, repletos de `ook`.

```shell
Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook?  Ook. Ook?  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook? Ook.  Ook! Ook!  Ook? Ook!  Ook. Ook?  Ook. Ook?  Ook. Ook?  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook?  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.  Ook. Ook?  Ook! Ook.  Ook! Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook. Ook.  Ook! Ook.  Ook! Ook.  Ook? Ook.  Ook! Ook!  Ook! Ook.  Ook. Ook.  Ook! Ook.
```

Buscamos esto en ``Google``, y vemos que pertenece a la misma familia de `lenguajes de programación esotéricos` y se llama concretamente `ook!`.

Vamos a traducirlo a ASCII y veremos que ya sí que es un texto más legible y lógico.

>***tener muy en cuenta los espacios al momento de copiarlo para traducirlos***

![](/blog/assets/dockerlab.es_fileception/paso_9.JPG)

```text
9h889h23hhss2
```

Ya tenemos el usuario `Peter` y también la clave. Ahora es momento de acceder al ``puerto 22`` mediante ``ssh`` con el siguiente código y verificar si en realidad funciona.

```shell
ssh peter@172.17.0.2
```

Ponemos la clave cuando nos la pida, y vemos que efectivamente tenemos acceso como Peter al servidor.

![](/blog/assets/dockerlab.es_fileception/paso_10.JPG)

Es momento de ver que podemos y no podemos hacer. Y como lamentablemente, no pertenecemos al grupo `sudoers` estamos sumamente limitados.

Pero si hacemos un ``ls`` en la ruta donde nos abrio el ssh `/home/peter/` veremos que hay un archivo llamado `nota_importante.txt` que dice lo siguiente:

>NO REINICIES EL SISTEMA !!
>
>HAY UN ARCHIVO IMPORTANTE EN TMP

![](/blog/assets/dockerlab.es_fileception/paso_11.JPG)

Pues, si nos vamos a la ruta `/tmp` hay dos archivos 

>importante_octopus.odt
>
>recuerdos_del_sysadmin.txt

Vemos el ``txt `` el cual nos narra lo siguiente:

>Cuando era niño recuerdo que, a los videos, para pasarlos de flv a mp4, solo cambiaba la extensión. Que iluso.

![](/blog/assets/dockerlab.es_fileception/paso_12.JPG)

El siguiente archivo con extensión `odt` es un archivo de ``libreoffice`` el equivalente a ``microsoft office`` para `Linux`

Como no podemos hacer mucho en el servidor, nos lo descargamos con el comando de ``scp (Secure Copy Protocol)``

```bash
scp peter@172.17.0.2:/tmp/importante_octopus.odt ./
```

![](/blog/assets/dockerlab.es_fileception/paso_13.JPG)

Si tenemos ``libreoffice `` instalado y lo abrimos, nos aparecerá el siguiente aviso:

![](/blog/assets/dockerlab.es_fileception/paso_14.JPG)

> **nota**: Este aviso no aparece en plataforma Windows

Su contenido es el mismo de ``recuerdos_del_sysadmin.txt``, lo cual no aporta nada.

Pero, y si la anécdota de cambiar la extensión de los archivos es una pista.

Si buscamos por internet, nos cuenta que:

>Un archivo _.docx_ se comporta como un archivo ZIP que **almacena** el contenido (texto e imágenes) **en datos XML y CSS** y, a continuación, los **comprime**. Al abrir el archivo _.docx_, el contenido se vuelve a descomprimir.[^1] 
>
>[....]
>
**un documento de Word con extensión DOCX no es más que un contenedor de archivos**.[^2]

Partiendo de estas afirmaciones, y sabiendo que libreoffice y word son totalmente compatibles, pues da a pensar que, tanto word como libreoffice usan la misma técnica para crear sus archivos. 

cambiamos de extensión el archivo con el comando

```shell
mv importante_octopus.odt importante_octopus.zip 
```

Vemos que lo extraemos sin problema y parece que todo está en orden. Hasta que encontramos un archivo oculto llamado `leerme.xml`.

![](/blog/assets/dockerlab.es_fileception/paso_16.JPG)

Y si lo vemos, este es su contenido

![](/blog/assets/dockerlab.es_fileception/paso_17.JPG)

Vemos, que son las credenciales del ``sysadmin``, por lo cual, ya deberíamos tener acceso ``root``.

La clave se encuentra en `base64` si le hacemos un `decode`, nos dará el siguiente texto:

```text
80h2380h34uouo3h4
```

Y si nos conectamos por ssh con el comando

```shell
ssh octopus@172.17.0.2
```

![](/blog/assets/dockerlab.es_fileception/paso_18.JPG)

Perfecto! Ya somos usuario root!

Y, para eliminar el laboratorio, simplemente con `ctrl+c` el bash script se encargaría de todo.

![](/blog/assets/dockerlab.es_fileception/paso_19.png)

<br>
### _Referencias_
* * *
[^1]: [.docx: todo sobre la extensión de Microsoft Word](https://www.ionos.es/digitalguide/online-marketing/vender-en-internet/que-es-un-archivo-docx/)
[^2]: [Lo que la extensión DOCX esconde](https://www.gonduana.com/lo-que-la-extension-docx-esconde/)