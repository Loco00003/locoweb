+++
title = "Webmaster"
date = 2024-01-28T22:57:32-06:00
summary="Another machine by sml, and pretty interesting too, introduced me to the use of the dig command."
tags=["hackmyvm","pentesting","nginx","dns","dig"]
series=["Hackmyvm Fácil"]
series_order=4
draft= false
+++
## Información de la maquina
| Nombre       | webmaster |
| ---------- | --------- |
| Sistema operativo         | Linux     | 
| Autor     | sml          |
| Sitio       | [hackymyvm](https://hackmyvm.eu)          |
| Dificultad | Fácil         |

## Reconocimiento y enumeración
Empezamos con un escaneo de reconocimiento de servicios con nmap:

![](imagenes/Pasted%20image%2020240128185639.png)

Tres puertos abiertos.Los usuales , puerto 80 y 22, y mi primero encuentro con el puerto *53*, que parece ser un servidor *DNS*, revisemos la pagina web ahora:

![](imagenes/Pasted%20image%2020240128185754.png)

Primera pista, una contraseña *puede* estar en un archivo *.txt*, echemos un vistazo al código fuente de la pagina:

![](imagenes/Pasted%20image%2020240128185847.png)

Un comentario *webmaster.hmv*, puede que tenga algún significado para resolver esta maquina, o no... Bueno, como no podemos hacer nada mas manualmente en esta pagina, corramos nuestra enumeración de directorios con gobuster como lo hacemos usualmente:

![](imagenes/Pasted%20image%2020240128201038.png)

*Hice algunos escaneos mas, pero no encontré nada...* 

Retrocedamos un poco, encontramos un comentario, *webmaster.hmv*, ahora, ¿tal vez, ese es un **nombre de dominio**? Añadimos ese dominio a nuestro archivo `/etc/hosts`, asi:

![](imagenes/Pasted%20image%2020240128201308.png)

Intentamos ver la pagina web otra vez:

![](imagenes/Pasted%20image%2020240128201329.png)

Funciona, bien. Ahora, tenemos un *nombre de dominio*, y tenemos el *puerto 53* abierto, consultemos la pagina [hacktricks](https://book.hacktricks.xyz) y veamos que encontramos. Eventualmente,encuentro algo en esta sección [sección](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns#zone-transfer).

![](imagenes/Pasted%20image%2020240128201505.png)

La verdad no sabia a que se refería con `AXFT`, entonces lo [googlee](https://www.briskinfosec.com/blogs/blogsdetail/DNS-Zone-Transfer#:~:text=AXFR%20(Asynchronous%20Full%20Transfer%20Zone,DNS%20server%20is%20well%20synced.)):

![](imagenes/Pasted%20image%2020240128202047.png)

Como tenemos tanto la `IP` y el `domain name (nombre de dominio)`, intentemos con el segundo comando que esta en la sección `zone transfer section`.

![](imagenes/Pasted%20image%2020240128202241.png)

## Acceso inicial
Y, seguramente hemos encontrado lo que son las credenciales para iniciar sesión por `SSH`, intentemos iniciar sesión:

![](imagenes/Pasted%20image%2020240128202414.png)

Obtenemos la bandera de usuario, bueno ahora, tenemos un script en el directorio home, revisemos que es lo que hace:

![](imagenes/Pasted%20image%2020240128203911.png)

Mm, pruebo el script, y me muestra esto:

![](imagenes/Pasted%20image%2020240128203946.png)

Obviamente, no me va a mostrar la bandera root, porque no esta en el directorio, intento añadir `/root/` antes de `root.txt`:

![](imagenes/Pasted%20image%2020240128204437.png)

No hay cambio... Miremos que mas podemos hacer:

![](imagenes/Pasted%20image%2020240128204536.png)

Mmm, tenemos privilegios de administrador con el *servidor web nginx*, investiguemos como podemos explotar eso. Después de un rato, *consigo la bandera de root*, pero no escale privilegios. De todas maneras, aquí esta lo que hice:

Encuentro este script en este [blog](https://darrenmartynie.wordpress.com/2021/10/25/zimbra-nginx-local-root-exploit/) escrito por darrenmarty:

![](imagenes/Pasted%20image%2020240128211456.png)

En resumen,lo que hace es, crea un archivo de configuracion *nginx.conf* en el directorio `/tmp`, asigna el `usuario` como `root`, establece algunas características del servidor, luego, **inicia un servidor http en el directorio root**. Después de eso, corre *nginx* con privilegios `sudo` privileges con el archivo de configuración que fue creado y muestras los contenidos del directorio `/etc/passwd`. 

Ahora, adapte este script según mis necesidades para esta maquina, entonces, hice esto:

![](imagenes/Pasted%20image%2020240128211843.png)

Se preguntaran, *porque alistar un escucha de ncat en la maquina objetivo*? Debido a esto:

![](imagenes/Pasted%20image%2020240128211941.png)

El script inicia exitosamente el servidor http con el archivo `config` que el script creo, pero, en esta maquina, `curl` no esta instalado o el usuario no puede usarlo. Pero, el servidor http esta corriendo, entonces, podemos intentar descargar el archivo con el comando `wget`. 

![](imagenes/Pasted%20image%2020240128212212.png)

## Escalada de privilegios
Y como dije, obtuvimos la bandera de root, pero, **no escalamos privilegios**. Revisando el blog mas a fondo, el autor si pudo escalar privilegios, aqui esta como lo hizo el:

![](imagenes/Pasted%20image%2020240128214019.png)

No se lo que es un *dynamic linker*, entonces investigo que es:

![](imagenes/Pasted%20image%2020240128214352.png)

![](imagenes/Pasted%20image%2020240128214439.png)

Entonces, uso el [script modificado](https://github.com/darrenmartyn/zimbra-hinginx/blob/main/hinginx.sh) y lo adapto a lo que ocupo, y veamos si obtenemos privilegios elevados:

![](imagenes/Pasted%20image%2020240128220024.png)

Eso no es bueno.Tal vez hay alguna forma mas fácil. Al final, decidí consultar otros writeups, para ver si alguien escalo privilegios con otros métodos. Consultando el writeup del usuario de hackmyvm [ARZ](https://arz101.medium.com/hackmyvm-webmaster-6f5862000c0a). Parece ser que, en el *directorio html* tenemos **privilegios no restringidos**, lo cual nos permite escribir un script en un archivo php, lo cual nos permitirá recibir una reverse shell en nuestra maquina. Buen hecho a tener en mente en el futuro.

![](imagenes/Pasted%20image%2020240128222557.png)

![](imagenes/Pasted%20image%2020240128222531.png)

![](imagenes/Pasted%20image%2020240128222758.png)

*Ejecutar este comando después de haber iniciado un escucha en tu maquina.*

![](imagenes/Pasted%20image%2020240128222620.png)

## Cosas que aprendí de esta maquina
- Si hay algún nombre de dominio, añadirlo a mi archivo `/etc/hosts`, y correr los comandos relacionados con el descubrimiento de información de dominios(`dig` or `nslookup`) para obtener información acerca de la maquina.
- **Siempre** revisar los contenidos del directorio donde están los archivos de la paginas web cuando tenemos privilegios de usuario, esta la posibilidad que, tengamos privilegios no restringidos en ese directorio, y codear un código simple en php para **habilitar remote code execution, y obtener una reverse shell nuestra maquina**.