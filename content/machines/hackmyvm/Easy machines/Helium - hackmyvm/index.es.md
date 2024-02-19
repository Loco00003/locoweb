+++
title = "Helium"
date = 2024-01-23T17:56:40-06:00
summary="Another machine by sml, but this time with a twist, this one also took me a while..."
tags=["pentesting","hackmyvm","steganography"]
series=["Hackmyvm Fácil"]
series_order=3
+++

## Información de la maquina
| Nombre       | helium       |
| ---------- | ------ |
| Sistema operativo        | Linux Debian |
| Autor     | sml    | 
| Sitio       | [hackmym](https://hackmyvm.eu)       |
| Dificultad | Fácil       |

## Reconocimiento y enumeración
Corro mi escaneo preferido de nmap `sudo nmap -oN helium_scan.txt -vv -sS -sC -sV -T4 -p- 192.168.25.19`

![](imagenes/Pasted%20image%2020240123181823.png)

Puerto 22 y 80 abiertos, revisemos la pagina web:

![](imagenes/Pasted%20image%2020240123181931.png)

Algo de contenido en la pagina web para variar, revisemos el código:

![](imagenes/Pasted%20image%2020240123182004.png)

Bien, conseguimos un nombre de directorio y de usuario *paul*, revisemos el directorio:

![](imagenes/Pasted%20image%2020240123182045.png)

Enumeremos este directorio con gobuster (incluimos las extension `.wav`, para ver si el archivo de verdad esta ahi).Mmm, nada, pero, la pista es, ¿podemos subir algo? Usando lo que aprendimos en la maquina *Serve de vulnyx*, podemos intentar subir un archivo con `curl`.

![](imagenes/Pasted%20image%2020240123182925.png)

Nada, retrocedamos un poco, el usuario *paul* ha recibido un llamado de atención por subir *archivos extraños* a la pagina web, entonces, intentemos descargar el archivo `.wav`  y correr algunas prueba en el. [Tome algunas referencias de este blog](https://medium.com/analytics-vidhya/get-secret-message-from-audio-file-8769421205c3):

Nada fuera de lugar, de momento... Usemos el programa [binwalk](https://github.com/ReFirmLabs/binwalk).

![](imagenes/Pasted%20image%2020240123185110.png)

![](imagenes/Pasted%20image%2020240123185741.png)

Interesante, usemos las pagina de ayuda, y buscamos como podemos extraer esta información:

![](imagenes/Pasted%20image%2020240123185851.png)

Usaremos este switch, corremos el comando otra vez, y miramos si extrajo algún archivo:

![](imagenes/Pasted%20image%2020240123190211.png)

Nada, aparentemente, los archivos *index* no nos son de utilidad.Entonces, dejamos de un lado el análisis del archivo. En el blog que referencia antes, el autor analiza el archivo usando un script de python que usa la librería `wave`, intentemos eso también:

![](imagenes/Pasted%20image%2020240123191633.png)

Después de intentar con eso, investigue mas, y encontré esta [herramienta](https://github.com/ragibson/Steganography#WavSteg):

![](imagenes/Pasted%20image%2020240123192200.png)

Lo descargo y lo pruebo:

![](imagenes/Pasted%20image%2020240123195902.png)

Nada... Intento esta [herramienta](https://github.com/danielcardeenas/AudioStego) también:

![](imagenes/Pasted%20image%2020240123195951.png)

Ok, nada. Sabemos que es un hecho que la clave para resolver esta maquina es un **archivo de audio**. ¿tal vez, hemos estado tratando con el ***equivocado***? Exploremos la pagina web una vez mas para ver si se nos paso algo por alto:

![](imagenes/Pasted%20image%2020240123200102.png)

Y asi fue... Intento descargar el archivo con `wget`:

![](imagenes/Pasted%20image%2020240123200204.png)

Lo tenemos. Lo escucho, parece ser algún tipo de código morse, busquemos si podemos encontrar algún herramienta que nos sirva. Finalmente, encuentro esta [pagina](https://morsecode.world/international/decoder/audio-decoder-expert.html) y en ella, subo el archivo, y selecciono la opción que mas se asemeja al sonido del archivo que tengo:

![](imagenes/Pasted%20image%2020240123201347.png)

Reproduzco el archivo, y reviso los resultados:

![](imagenes/Pasted%20image%2020240123201437.png)

Y en la sección de *espectrograma*, encuentro lo que mas seguro es la contraseña del usuario *paul*, intentemos iniciar sesión con SSH...

## Acceso inicial y escalado de privilegios

![](imagenes/Pasted%20image%2020240123201606.png)

¡Entramos! Obtengamos la bandera de usuario y encontremos con el comando `sudo -l` que comando podemos correr con privilegios de admin sin necesitar contraseña:

![](imagenes/Pasted%20image%2020240123202348.png)

Busquemos eso en [gtfobins](https://gtfobins.github.io/gtfobins/ln/):

![](imagenes/Pasted%20image%2020240123202539.png)

Intentemos eso:

![](imagenes/Pasted%20image%2020240123202630.png)

Y obtenemos privilegios de administrador, obtengamos la bandera y terminemos esta maquina:

![](imagenes/Pasted%20image%2020240123202738.png)

Y con eso, hemos comprometido la maquina, me tomo un rato, pero muy interesante después de todo.

## Cosas que aprendí de esta maquina
- Si un archivo tiene contenido escondido, o cualquier información, probar con el **método mas fácil posible** primero. Las herramientas de esteganografia, como se pudo ver, pueden ser bastantes expansivas. En esta caso, tal vez te distes cuentas antes que yo, que simplemente puede haber usado el espectrógrafo, y ese fue el caso. Me concentre mas en buscar la herramienta adecuada, que no considere usar el espectrógrafo. Algo a tener en cuenta cuando me encuentre otra vez con un archivo de audio en CTFs.
- **SIEMPRE**, revisar **TODOS** los contenidos de las paginas web **MANUALMENTE**, antes de seguir usando herramientas automáticas, perdí bastante tiempo enumerando con gobuster, en cambio, pude haber buscado mejor en la pagina web, lo cual me hubiera llevado a encontrar la pista que nos ayudo a resolver la maquina.