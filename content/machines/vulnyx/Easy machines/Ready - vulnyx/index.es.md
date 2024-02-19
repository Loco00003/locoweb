+++
title = 'Ready'
date = 2024-01-20T16:44:27-06:00
draft = false
description = "Una de las pocas maquinas que he logrado resolver sin consultar writeups de otros hackers eticos"
etiquetas = ["hacking etico","vulnyx","redis","password bruteforcing","john"]
series=["Vulnyx Fácil"]
series_order=1
+++

## Información acerca de la máquina
| Nombre       | Ready   |
| ---------- | ------- |
| Sistema Operativo         | Linux   |
| Autor     | d4t4s3c |
| Dificultad | Easy    |
| Sitio          | [vulnyx](https://vulnyx.com)|


## Enumeración

![](images/ready%20(1).png)

Revisamos las paginas web:

![](images/ready%20(2).png)

![](images/ready%20(3).png)

Ambas son páginas predeterminadas de Apache, enumeramos con gobuster solo para estar seguros. Mientras el escaneo corre, revisamos que es redis:

![](images/ready%20(4).png)

Aparentemente, estamos tratando con una base de datos. Revisemos si podemos lograr una sesión con el siguiente comando *redis-cli*:

![](images/ready%20(5).png)

Probemos ingresar sin un nombre de usuario:

![](images/ready%20(6).png)

Hemos ingresado, buscamos los comandos comunes para redis :

![](images/ready%20(7).png)

Antes de eso, corremos un escan con nmap para ver qué otra información podemos encontrar(*Parece ser que no hay ninguna base de datos en este servidor...*):

![](images/ready%20(8).png)

*No encontramos nada de interés en las páginas web.*

![](images/ready%20(9).png)

Encuentro esta información en la página [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#ssh):

![](images/ready%20(10).png)

Y en redis, encuentro los siguientes directorios:

![](images/ready%20(11).png)

Seguimos los pasos utilizando el directorio `/root/.ssh`

![](images/ready%20(12).png)

Exportamos la llave al servidor de redis y seguimos los pasos de abajo:

![](images/ready%20(13).png)

## Acceso inicial y escalada de privilegios
Intentamos ingresar con una sesión SSH:

![](images/ready%20(14).png)

¡Y tenemos privilegios root! Instalamos `unzip` en estas máquinas para extraer los contenidos:

![](images/ready%20(15).png)

Se nos pide contraseña… Vamos a crackearla.

Arrancamos un servidor http con este comando: `python3 -m http.server -b <TARGET-MACHINE-IP> <PORT>`

Descargamos el archivo en nuestra maquina con `wget <TARGET-MACHINE-IP>:<PORT>/root.zip`

![](images/ready%20(16).png)

Ahora usamos `zip2john` para escribir el hash del archivo zip a un archivo de texto y luego crackeamos:

![](images/ready%20(17).png)

Extraemos los contenidos del archivo:

![](images/ready%20(18).png)

Encontramos la bandera de root, ahora buscamos la bandera de usuario:

![](images/ready%20(19).png)

Y con esto, hemos comprometido la maquina.

## Que aprendí de está maquina
- Escalada de privilegios con redis.