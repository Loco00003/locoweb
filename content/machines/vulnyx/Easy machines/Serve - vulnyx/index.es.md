+++
title = "Serve"
date = 2024-01-21T12:18:15-06:00
summary="Get ready for a long read, despite being an easy machine, took me a while to figure it out."
tags=["python","keepass","pentesting","vulnyx","easy","password bruteforcing","hydra","john"]
series=["Vulnyx Fácil"]
series_order=2
+++

## Informacion sobre la maquina
| Nombre | Serve |
| ---- | ---- |
| Sistema Operativo | Linux Debian |
| Autor | d4t4s3c |
| Sitio | [Vulnyx](https://vulnyx.com) |
| Dificultad | Fácil |

## Enumeración
Corremos nuestro escaneo de nmap:
`sudo nmap -oN serve_scan.txt -vv -sS -sC -sV -T4 -p- 192.168.25.18`

![](imagenes/Pasted%20image%2020240122185116.png)

Puerto 22 y 80 abiertos, revisemos la pagina web primero:

![](imagenes/Pasted%20image%2020240122185250.png)

Página predeterminada de apache2. Corremos nuestro escaneo de directorios con gobuster, a ver si encontramos algo de interés:
`gobuster dir -u http://192.168.25.18/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -x php,html,txt `

![](imagenes/Pasted%20image%2020240122185506.png)

Encontramos tres directorios interesantes a los que podemos acceder, vamos a proceder a checar cada uno de ellos manualmente, y luego correr otro escaneo con gobuster para ver si encontramos algo más. Antes que nada, voy a revisar el archivo notes.txt:

![](imagenes/Pasted%20image%2020240122185740.png)

Encontramos información crítica,(también, un posible nombre de usuario, *teo*) recordando el escaneo que hicimos hace poco con gobuster. Encontramos el directorio `/secrets`, entonces, corremos un escaneo en ese directorio, pero con una lista diferente, para asegurarnos al 100% que vamos a obtener algo útil:

`gobuster dir -u http://192.168.25.18/secrets/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -x php,html,txt`

*Nótese el cambio a una lista diferente y el cambio de directorio para el escaneo.*

Mientras corría el escaneo, revisé el directorio `/javascript`, y, como se esperaba, tenemos prohibido el acceso:

![](imagenes/Pasted%20image%2020240122190229.png)

Ahora, espero a que el escaneo se termine a ver si encontramos algo interesante...

![](imagenes/Pasted%20image%2020240122190402.png)

Nada... Ahora, recordemos el mensaje, especialmente, esta parte *Change x for your employee number*(*Cambia x por tu número de empleado*), bien, tal vez, si creamos una lista de números, digamos, de 1 al 100, ¿tal vez encontraremos algo? Entonces, creo un script simple en python que cuente hasta 100, y luego, transfiero eso a un archivo de texto. Hay varias maneras de hacer esto, es muy probable que encuentres una mejor manera de esto, pero bueno, yo lo hice de esta forma:

![](imagenes/Pasted%20image%2020240122191750.png)

1. Declarar una variable y empezar la cuenta desde 0, esto, debido a la forma que python cuenta, en vez de empezar desde 1, empieza desde 0, entonces debemos tomar eso en cuenta.
2. Hacemos uso de un ciclo `while`. En resumen, este ciclo realizará el proceso que está *indentado a él* hasta que satisfaga la condición que causará que pare. En este caso, este se romperá hasta que el número sea menor a 101. Luego tenemos el `n+=1,esta es una notación elegante que python tiene, es lo mismo que poner esto `n=n+1`, lo que hace exactamente lo que queremos, sumar el número anterior más 1:
Probamos el código para ver si funciona:

![](imagenes/Pasted%20image%2020240122192324.png)

¡Bien! Solo un número adicional, simplemente reemplazamos 101 por 100 en nuestro código y ahora debería trabajar como debe. Ahora, para guardar este output en un archivo de texto, usamos el operador de redirección de linux `>` para guardarlo en un archivo de texto:

`python3 counter.py > numberlist.txt`

![](imagenes/Pasted%20image%2020240122192522.png)

Ahora, intentamos enumerar con gobuster otra vez:

![](imagenes/Pasted%20image%2020240122192700.png)

Otra vez nada... ¿Y si probamos descargar el directorio `/secrets` con `wget`?
Todavía nada, solo me descargo un archivo html vacio. Decido correr otro escaneo con gobuster, pero esta vez con una lista más grande. También recuerdo que no revisé el directorio `/directorio` , tal vez hay algo ahí de lo que no me percate, reviso el código fuente de la página:

![](imagenes/Pasted%20image%2020240122193721.png)

Hay 8 líneas, pero están en blanco. Bien, tenemos un nombre de usuario, ¿tal vez podemos probar hacer un ataque de fuerza bruta para obtener una contraseña? Recordemos que el puerto SSH estaba abierto, intentemos eso. En mi caso, yo uso hydra:

`hydra -l teo -P /usr/share/wordlists/rockyou.txt 192.168.25.18 ssh -t 4`

*Después de un rato, parecía que el crackeo de la contraseña no avanzaba. En el caso de ser una contraseña fácil de crackear, con esta lista debería tomar lo máximo 60 segundos, entonces por eso decido parar el crackeo y buscar otras formas de ganar acceso al sistema.*

Mientras el crackeo de contraseña estaba corriendo, recuerdo que no corrí un escaneo en el directorio `/javascript`, solo para salir de la duda de que no hay nada de interés en ese directorio, corro el escaneo:

![](imagenes/Pasted%20image%2020240122194643.png)

Y si hay otro directorio ahí, el cual tenemos prohibido el acceso, corremos otro escaneo en ese directorio:

![](imagenes/Pasted%20image%2020240122195300.png)

Otro directorio, `/jquery`, ¡pero ahora tenemos un código `200`! Podemos acceder a esta página y revisar sus contenidos.

![](imagenes/Pasted%20image%2020240122195356.png)

Un código de javascript, ¿tal vez, está usando componentes contenidos en este directorio? Corremos otro escaneo:

![](imagenes/Pasted%20image%2020240122200547.png)

Nada... retrocedamos un poco, el equipo de IT menciono una *database*(*base de datos*), que se encuentra dentro del directorio `/secrets`, tal vez, si añadimos extensiones comunes relacionadas con las bases de datos, encontraremos algo, y, antes que nada, revisemos que extensiones están relacionadas con el almacenamiento de contraseñas, y encuentro una extensión que podría ser la que buscamos:

![](imagenes/Pasted%20image%2020240122201844.png)

Añado esa extensión al escaneo de gobuster, e intento otra vez. Mientras el escaneo corre, encuentro más información:

![](imagenes/Pasted%20image%2020240122202131.png)

(*Keypass es un programa para guardar contraseñas*) Y tiene sentido, ya que guardar una contraseña en un archivo SQL sin ningún tipo de encriptación no es seguro, otra cosa importante es que, en linux y macOS, el nombre cambia a *KeePassX*, entonces, la extensión del archivo cambia también, esperemos que termine el escaneo a ver si arroja algún resultado.

![](imagenes/Pasted%20image%2020240122202847.png)

Nada, intentemos *kdbx*, y miremos si encontramos algo:

![](imagenes/Pasted%20image%2020240122203004.png)

¡Bien! Paramo el scan, descargamos el archivo y ahora vemos que podemos hacer con el:

![](imagenes/Pasted%20image%2020240122203155.png)

Y, obviamente, si usamos `cat` el archivo, no nos arroja nada util:

![](imagenes/Pasted%20image%2020240122203232.png)

Bueno, investiguemos un poco, encontremos de qué forma podemos extraer información de este, y rápidamente encontré este [blogpost por "the dutch hacker"](https://www.thedutchhacker.com/how-to-crack-a-keepass-database-file/) que describe cómo crackear este tipo de archivo, y, es una herramienta que hemos usado antes, *john*, pero esta vez la sintaxis es diferente (*la última vez usamos john para crackear la contraseña de un archivo zip*):

![](imagenes/Pasted%20image%2020240122203534.png)

Entonces, obtengamos ese hash, y crackeemos la contraseña para acceder a esta base de datos:

![](imagenes/Pasted%20image%2020240122204033.png)

Encontramos la contraseña maestra a la *base de datos*, descarguemos el programa *KeePassX*, lo instalamos e intentamos abrir el archivo que descargamos antes:

Parece ser, que la versión de este programa de cuando la base de datos de contraseña fue creada ya no es desarrollada. Busquemos si podemos abrir este archivo con una versión más nueva del programa.

![](imagenes/Pasted%20image%2020240122204806.png)

Si podemos, descargamos e instalamos el programa, e intentamos abrir el archivo:

![](imagenes/Pasted%20image%2020240122204742.png)

![](imagenes/Pasted%20image%2020240122210253.png)


Tenemos el nombre de usuario: *admin* y la contraseña, y, observamos otra cosa: el nombre `webdav`. Recordemos el escaneo de gobuster que hicimos inicialmente, encontramos un directorio con ese nombre, pero nos arrojaba un código 401. Revisemos:

![](imagenes/Pasted%20image%2020240122210435.png)

Bien, usamos las credenciales que acabamos de encontrar para ver los contenidos de las páginas, y aparentemente, *no podemos acceder*. Ahora, la contraseña termina con *XXX*, y si recordamos el mensaje que estaba en el archivo de texto que revisamos al principio, tenemos que reemplazar esas X con el número de empleado de *teo*, y parece ser que son 3 números, nada más, podemos hacer un ataque de fuerza bruta, luego de investigar un poco, encontré esto en un post de [reddit](https://www.reddit.com/r/hacking/comments/ek2b5m/how_to_brute_force_a_popup_box/), usando hydra, podemos hacer un ataque de fuerza bruta.

![](imagenes/Pasted%20image%2020240122212314.png)

Tenemos los nombres de usuario *teo* y *admin*, pero ahora, ¿cómo encontramos la contraseña? Sabemos que es una combinación de *tres números*, entonces, el script que codeamos antes nos será de ayuda; lo adaptamos, así para no tener que escribir cada número manualmente:

![](imagenes/Pasted%20image%2020240122212750.png)

Lo probamos para ver si funciona como deberia:

![](imagenes/Pasted%20image%2020240122212835.png)

Bien, exportamos eso a un archivo de texto. Ahora adaptemos el archivo de hydra que vimos más arriba.

![](imagenes/Pasted%20image%2020240122213538.png)

Obtenemos algo extraño en la terminal; en vez de empezar al ataque, hizo eso. Viendo más a fondo el post, encuentro lo siguiente:

![](imagenes/Pasted%20image%2020240122213701.png)

Adaptó el comando y ahora se ve así: `hydra -l admin -P bruteforce_pass.txt -s 80 -f 192.168.25.18 http-get /webdav`, iniciamos el ataque y...

![](imagenes/Pasted%20image%2020240123122950.png)

¡Obtenemos la contraseña! Intentemos iniciar sesión otra vez en la página:

![](imagenes/Pasted%20image%2020240122214047.png)

Ok, entramos, pero no hay nada... ¿Tal vez nos podemos iniciar sesión por SSH?

![](imagenes/Pasted%20image%2020240122214748.png)

Ok, tenemos un directorio vacío, ¿tal vez, hay alguna forma de subir un archivo? A través de nuestra terminal, debido a que esta página carece de una opción de subir archivo.Encuentro [esto](https://reqbin.com/req/c-dot4w5a2/curl-post-file):

![](imagenes/Pasted%20image%2020240122215005.png)

Intento subir un archivo de texto de prueba:

![](imagenes/Pasted%20image%2020240122215212.png)

Bueno, lo bueno es que ya tenemos las credenciales. Revisemos qué switch tenemos que incluir en el comando:

![](imagenes/Pasted%20image%2020240122215318.png)

Bien, añadimos eso, e intentamos otra vez:

![](imagenes/Pasted%20image%2020240122215741.png)

En la misma página, investigamos con usar el switch `-u`:

![](imagenes/Pasted%20image%2020240122215716.png)

Intentamos otra vez:

![](imagenes/Pasted%20image%2020240122220004.png)

Mismo resultado, revisamos la página de ayuda del comando, para ver si nos falta algo:

![](imagenes/Pasted%20image%2020240122220125.png)

Entonces, ¿cuál es la diferencia con el switch `-d` ?

![](imagenes/Pasted%20image%2020240122220232.png)

Parece ser, que hemos estado enviando un request `POST` a la página. En vez de subir el archivo, cambiamos el switch e intentamos otra vez:

![](imagenes/Pasted%20image%2020240122220426.png)

Entonces, en la imagen de arriba, vi algo que me llamó la atención: el switch `--digest`. Investigué un poco sobre lo que hace. Y encontré en este [blog](https://www.hackingarticles.in/understanding-http-authentication-basic-digest/) lo que es:

![](imagenes/Pasted%20image%2020240122221244.png)

Aparentemente, es esa “cajita”, así es como la llamé cuando la vi. Ahora sé su nombre apropiado, entonces, añado el switch `--digest` e intento otra vez:

![](imagenes/Pasted%20image%2020240122221441.png)

¡Éxito! Revisemos la página:

![](imagenes/Pasted%20image%2020240122221522.png)

Bien, pero la pregunta es: ¿podemos abrir este archivo?

![](imagenes/Pasted%20image%2020240122221549.png)

¡Si podemos! Esto es bueno, significa que, si subimos, por ejemplo, un archivo php, debería correr. Esto nos permitiría obtener una reverse shell. Ahora subamos nuestro archivo php con el que obtendremos la reverse shell y preparamos nuestro escucha en netcat:

En mi caso voy a usar esta [payload php](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) , editamos el archivo con la información de nuestra máquina y el puerto cualquiera que no esté en uso, y subimos con el comando que usamos antes:

![](imagenes/Pasted%20image%2020240122221853.png)

![](imagenes/Pasted%20image%2020240122222523.png)

![](imagenes/Pasted%20image%2020240122222449.png)

Todo está listo, hacemos click en el archivo `.php` y revisamos nuestro escucha:

## Acceso inicial

![](imagenes/Pasted%20image%2020240122222607.png)

¡Bien! Entramos como el usuario `www-data`, chequemos que podemos hacer:

![](imagenes/Pasted%20image%2020240122222828.png)

Observamos que el usuario `teo` puede correr ese comando como administrador, sin necesitar ingresar contraseña. Intentemos obtener una sesión con ese usuario, revisemos por dónde podemos entrar:
*Antes de eso, corro el siguiente comando* `python3 -c 'import pty; pty.spawn("/bin/bash")'` *el cual hará que nuestra sesión se vea un poco mejor* .
Corro el comando `find / -name "*.txt" 2>/dev/null`, para ver si encuentro algún archivo de texto interesante *visitar este [link](https://explainshell.com/explain?cmd=find+%2F+-name+"*.txt"+2>%2Fdev%2Fnull) si quieres saber más a detalle qué es lo que hace este comando.*:

Encuentro bastantes archivos de texto, pero ninguno interesante. Investigo en la página [hacktricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/payloads-to-execute#www-data-to-sudoers) alguna forma de cambiar de usuario o escalar privilegios:

![](imagenes/Pasted%20image%2020240122224740.png)

Intentemos cada uno de esos métodos:

![](imagenes/Pasted%20image%2020240122225024.png)

Ninguno funcionó, intentemos algo más.

Ok, podemos correr el comando `wget` como *teo*, si usamos el switch `-u`, cuando usamos `sudo`, entonces tal vez, aunque no podemos acceder información en esta máquina, ¿tal vez podemos enviar la información a nuestra máquina? Revisemos el manual del comando `wget`:

![](imagenes/Pasted%20image%2020240122232354.png)

Encuentro esto en el manual del comando wget,  primero intento enviar un string. Preparo mi escucha:

![](imagenes/Pasted%20image%2020240122232637.png)

![](imagenes/Pasted%20image%2020240122232706.png)

¡Bien! Esto significa, que probablemente, podemos enviar los contenidos de un archivo con el switch `--post-file`. El usuario *teo*, si tenemos suerte, tiene el directorio `/.ssh`, con la llave privada `id_rsa` dentro de el, tal vez podemos mostrar esa información en nuestra máquina, intentemos eso ahora:

![](imagenes/Pasted%20image%2020240122232950.png)

¡Ahí está! Copiamos eso, lo nombramos `id_rsa` y le damos privilegios limitados con `chmod 400 id_rsa` y luego, intentamos iniciar sesión con SSH:

![](imagenes/Pasted%20image%2020240122233232.png)

Se nos pregunta una contraseña, usamos `ssh2john` y luego crackeamos este hash:

![](imagenes/Pasted%20image%2020240122233407.png)

Obtenemos la contraseña, intentemos iniciar sesión otra vez:


## Escalada de privilegios

![](imagenes/Pasted%20image%2020240122233441.png)

Entramos, y buscamos la bandera de usuario:

![](imagenes/Pasted%20image%2020240122233714.png)

Ahora, busquemos la forma de escalar privilegios:

![](imagenes/Pasted%20image%2020240122233911.png)

Revisamos los contenidos del archivo:

![](imagenes/Pasted%20image%2020240122233942.png)

Es un archivo ruby. Busquemos cómo escalar privilegios con esta información. Primero, veamos qué hace el script:

![](imagenes/Pasted%20image%2020240123000105.png)

Corremos el script y hacemos lo que sugiere:

![](imagenes/Pasted%20image%2020240123000218.png)

Nos lleva a una shell interactiva, parecida a **vim**, entonces, ¿tal vez nos podemos hacer un escape? Intentemos escribir `!/bin/bash`:

![](imagenes/Pasted%20image%2020240123000317.png)

Obtenemos privilegios de administrador, buscamos la bandera de root, y comprometemos esta máquina:

![](imagenes/Pasted%20image%2020240123000407.png)

Y con esto, hemos terminado esta máquina, algo larga, pero aprendí bastante de ella.  

## Cosas que aprendi de esta maquina
- Checar archivos con extensiones menos comunes, como se vio en esta máquina, donde encontramos un archivo`.kdbx`, correspondiente a la aplicación *KeePass*, lo cual nos ayudó con el acceso inicial a la máquina, mantener en mente este hecho para futuras CTF.
- Leer los manuales de los comandos "básicos", como `curl` o `wget`, pueden ser útiles de maneras en las que un principiante como yo no se imagina, en este caso, obtuvimos información crítica que nos permitió acceder como un usuario más privilegiado, una mejora significativa en comparación con el usuario `www-data`.
- Aprendí cómo hacer ataques de fuerza bruta a un `http digest feed` con hydra, con una **partial password** obtenida de la base datos de *KeePass* que descargamos de la página web.
- Si encontramos, por ejemplo, un script personalizado, intentar correrlo primero, y ver qué es lo que eso y luego checar los contenidos del código, para ver cómo podemos escalar privilegios. Si tenemos una shell interactiva dentro del script, podemos intentar una secuencia de escape con `!/bin/bash`.