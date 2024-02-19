+++
title = 'Friendly'
date = 2024-01-09T20:12:37-06:00
summary="My first machine, despite the easy difficulty, it took me quite some time"
draft = false
tags=["pentesting","hackmyvm","ftp"]
series=['Hackmyvm Fácil']
series_order=2
+++

## Información de la maquina  
***
| Nombre  | Friendly  |
|---|---|
| Sistema operativo | Linux  |
|Autor   |RiJaba1   |
|Dificultad  |Easy   |
|Sitio   |[hackmyvm](https://hackmyvm.eu)   |

***

## Preparación de la red NAT (virtual box)

Antes que nada, preparamos en virtual box nuestra red NAT, para que nuestras maquinas se puedan comunicar, asignamos la ip con la que queremos identificar a la red, y luego en cada maquina revisamos que estén en la misma red :
![](imagenes/Pasted%20image%2020240109202337.png)
![](imagenes/Pasted%20image%2020240109213033.png)
![](imagenes/Pasted%20image%2020240109213047.png)


## Reconocimiento y enumeración

Corremos un escaneo de nmap en la maquina:
```
Nmap 7.94SVN scan initiated Tue Jan  9 20:20:06 2024 as: nmap -oN friendly_scan.txt -vv -sS -sV -sC -T4 -p 0-9999 192.168.25.4
Warning: Hit PCRE_ERROR_MATCHLIMIT when probing for service http with the regex '^HTTP/1\.1 \d\d\d (?:[^\r\n]*\r\n(?!\r\n))*?.*\r\nServer: Virata-EmWeb/R([\d_]+)\r\nContent-Type: text/html; ?charset=UTF-8\r\nExpires: .*<title>HP (Color |)LaserJet ([\w._ -]+)&nbsp;&nbsp;&nbsp;'
Nmap scan report for 192.168.25.4
Host is up, received arp-response (0.00015s latency).
Scanned at 2024-01-09 20:20:06 CST for 14s
Not shown: 9998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--   1 root     root        10725 Feb 23  2023 index.html
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.54 ((Debian))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.54 (Debian)
MAC Address: 08:00:27:A2:9F:C0 (Oracle VirtualBox virtual NIC)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan  9 20:20:20 2024 -- 1 IP address (1 host up) scanned in 14.18 seconds
```

Puerto FTP y HTTP abiertos, revisamos la pagina web para ver que información podemos obtener:

![](imagenes/Pasted%20image%2020240109202627.png)

Parece ser una pagina predeterminada de apache2, enumeramos los directorios para ver si encontramos algo de interés:

![](imagenes/Pasted%20image%2020240109202929.png)

Como podemos ver, tanto con la lista pequeña y mediana de dirbuster, no encontramos nada de interés en la pagina, ahora vamos a revisar el servicio FTP. Intentamos hacer un *anonymous login* para ver si podemos entrar sin credenciales:

![](imagenes/Pasted%20image%2020240109203206.png)

Entramos al servidor FTP, revisemos si hay algún archivo interesante.

![](imagenes/Pasted%20image%2020240109204803.png)

Encontramos un archivo index.html, que es el archivo que esta en la pagina default de apache2, y parece ser que tenemos privilegios de escritura y lectura, ¿Tal vez podemos subir algo? Intentare con el archivo de texto donde se guardo mi escaneo de nmap:

![](imagenes/Pasted%20image%2020240109205653.png)

Y se subió correctamente, revisamos la pagina, deberíamos ver nuestro escaneo:

![](imagenes/Pasted%20image%2020240109205725.png)
Ahora, vamos a intentar subir un script php, el cual nos arrojara una reverse shell a nuestra maquina, pero antes que nada, tenemos que modificar ciertos contenidos del script, para el escucha de netcat que vamos a correr en nuestra maquina:

![](imagenes/Pasted%20image%2020240109210055.png)

Subimos el script al servidor FTP:

![](imagenes/Pasted%20image%2020240109210147.png)

Preparamos nuestro escucha de netcat, y luego en la pagina, vamos a la barra de url y escribimos esto `http://192.168.25.4/php-reverse-shell.php` y damos enter, deberíamos obtener una shell session en nuestra escucha:

![](imagenes/Pasted%20image%2020240109210347.png)

## Acceso inicial y escalada de privilegios
Tenemos la shell de usuario, listamos los contenidos del directorio:

![](imagenes/Pasted%20image%2020240109210430.png)

Vamos al directorio home, para ver si encontramos algo de interés ahi:
Encontramos un posible *nombre de usuario: RiJaba1*
Y dentro de ese mismo directorio encontramos la bandera de usuario:

![](imagenes/Pasted%20image%2020240109210733.png)

Ahora, necesitamos escalar privilegios, corremos `sudo -l`:

![](imagenes/Pasted%20image%2020240109210836.png)

Interesante, buscamos en `gtfo bins` alguna manera de explotar esto:

https://gtfobins.github.io/gtfobins/vim/

Si el binario tiene permitido correr como administrador el comando `sudo`, no nos dará privilegios elevados, pero podrá ser usado para navegar el sistema de archivos, escalar o mantener acceso privilegiado.

A. `sudo vim -c ':!/bin/sh'`
B. Requiere que `vim` se compilado con soporte de python. Agregamos `:py3` para Python 3.
	`sudo vim -c ':py import os; os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'`

C. Este requiere que `vim` sea compilado con soporte Lua.
`sudo vim -c ':lua os.execute("reset; exec sh")'`

Vamos a intentar el primer método.

![](imagenes/Pasted%20image%2020240109211405.png)

Corremos el comando, y tenemos privilegios de administrador, revisemos que podemos encontrar:

![](imagenes/Pasted%20image%2020240109211436.png)

Vamos al directorio `/root`, y corremos el comando `ls -al` , y podemos ver que no tenemos los privilegios suficientes para ver el archivo, cambiamos los privilegios con `chmod 777 root.txt` e intentamos ver sus contenidos con cat:

![](imagenes/Pasted%20image%2020240109211740.png)

Parece ser, que la bandera que encontramos no es la correcta, corremos el siguiente comando `find / -name root.txt` , para ver si nos encuentra el archivo correcto:

![](imagenes/Pasted%20image%2020240109212033.png)

Y encontramos definitivamente el que estamos buscando, el cual es el que esta en la carpeta de apache2.Corremos el comando `cat` en el archivo, y deberíamos obtener la bandera de root:

![](imagenes/Pasted%20image%2020240109212322.png)

Y con esto, hemos comprometido la maquina.

## Cosas que aprendí de esta maquina
- Siempre revisar si puedo subir archivos si el servicio tiene la opción.
