+++
title = "Simple"
date = 2024-02-17T18:17:56-06:00
summary="First windows machine"
draft=false
tags=["",""]
series=["Hackmyvm Fácil"]
series_order=6
+++

## Prefacio
Reciente complete el contenido de [sobre escalada de privilegios de windows](https://tryhackme.com/room/windowsprivesc20) en tryhackme.Entonces, decidí intentar resolver una maquina windows para variar, la primera que intento por cierto.

## Información sobre la maquina
| Nombre       | Simple                                                         |
| ---------- | ---------------------------------------------------------------- |
|Sistema Operativo|Windows Server 2019|
| Autor     | GatoGamer                                                          |
| Sitio       | [hackmyvm](https://hackmyvm.eu/machines/machine.php?vm=Simple) | 
| Dificultad| Fácil                                                                |


## Reconocimiento y enumeración

Nos aseguramos de tener comunicación con la maquina:

![](imagenes/Pasted%20image%2020240217182231.png)

Y ahora corremos nuestro escaneo de `nmap`:

```shell-session
nmap -oN simple_scan.txt -vv -sS -sC -sV -T4 -p- 192.168.25.27
Nmap scan report for 192.168.25.27
Host is up, received arp-response (0.00030s latency).
Scanned at 2024-02-17 18:23:05 CST for 101s
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE       REASON          VERSION
80/tcp    open  http          syn-ack ttl 128 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Simple
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 128 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 128
5985/tcp  open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          syn-ack ttl 128 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 128 Microsoft Windows RPC
MAC Address: 08:00:27:57:28:8B (Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| nbstat: NetBIOS name: SIMPLE, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:57:28:8b (Oracle VirtualBox virtual NIC)
| Names:
|   SIMPLE<00>           Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   SIMPLE<20>           Flags: <unique><active>
| Statistics:
|   08:00:27:57:28:8b:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 4952/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 17879/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 64050/udp): CLEAN (Timeout)
|   Check 4 (port 12789/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: 0s
| smb2-time: 
|   date: 2024-02-18T00:24:41
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

```

Resultados interesantes, obtenemos un posible nombre de area de trabajo de windows: `SIMPLE`, eso puede ser util después, revisemos la pagina web:

![](imagenes/Pasted%20image%2020240217183044.png)

Tenemos un par de potenciales nombres de usuario, los tendremos en mente.También revise el código fuente de la pagina, pero no encontré nada interesante, corramos nuestro escaneo con `gobuster` en esta pagina web:

![](imagenes/Pasted%20image%2020240217183940.png)

En el proceso de revisar estos directorios manualmente, a todos ellos nos fue denegado el acceso. En vez de enumerar cada uno de ellos, sigo intentando otros medios.
Ahora, tenemos esta información:
- Los siguientes nombres de usuario:
	1. ruy
	2. marcos
	3. lander
	4. bogo
	5. vaiper
- Y el nombre de maquina, o workstation : `SIMPLE`

Investigo como enumerar ciertos servicios de windows, y encuentro las siguientes herramientas: `crackmapexec`,`smbmap`,`smbclient`. Intentemos usar cada una de ellas y revisemos si podemos obtener algún tipo de información util:

Primero, intentemos un login anónimo con `smbclient`:

![](imagenes/Pasted%20image%2020240217210521.png)

Nada, intentemos usar `smbmap`:

![](imagenes/Pasted%20image%2020240217210653.png)

Intentamos otra vez con un login anónimo, y no obtenemos nada. Ahora, una de las razones por la cual podríamos preferir `crackmapexec` para esta maquina, es el hecho que nos deja usar *archivos* para usuarios y contraseñas, entonces, hagamos eso:

![](imagenes/Pasted%20image%2020240217210931.png)

Interesante, veamos que podemos hacer con eso. Después de un rato, parece ser, que cuando esta maquina fue creada, podíamos acceder con la contraseña que tenia, pero debido a alguna configuración de windows, la contraseña ha expirado.Entonces, para poder continuar, tenemos que cambiar la contraseña, y como ya lo hice, pero no tome fotos, aquí esta paso a paso como hacerlo:
1. Ve a la maquina windows.
2. En la ventada del command prompt, vas a ver `presione ctrl+alt+supr` para continuar, dependiendo que programa de visualización uses, tendrás que habilitar que las combinaciones especiales de teclado puedan ser detectada en la maquina, en virtual box, use esta opción:
		![](imagenes/Pasted%20image%2020240217211721.png)
		
3. Luego, se preguntara por la contraseña de admin, no tenemos eso. Por lo tanto, presionamos `esc` y nos mostrara una lista de usuario, ahi, seleccionamos `bogo`, introducimos la contraseña `bogo`, y cambiamos la contraseña a cualquiera que nos convenga.
## Acceso Inicial
Ahora, con nuestra contraseña, intentamos usar otra vez `crackmapexec`:

![](imagenes/Pasted%20image%2020240217211942.png)

Bien, tenemos privilegios de lectura tanto en la share `IPC$` como en `LOGS`. Intentemos ingresar con `smbclient` a la share `LOGS`:

![](imagenes/Pasted%20image%2020240217212247.png)

Ingresamos, y encontramos un `archivo log`, lo descargamos, y revisamos sus contenidos:

![](imagenes/Pasted%20image%2020240217212549.png)

Ahi, encontramos unas credenciales que nos podrían permitir acceso remoto, intentemos eso:

![](imagenes/Pasted%20image%2020240217212937.png)

Encontramos que la contraseña de `marcos` ha expirado también, tenemos que cambiarla, seguimos los pasos que enumere mas arriba:

![](imagenes/Pasted%20image%2020240217213601.png)

![](imagenes/Pasted%20image%2020240217213624.png)

![](imagenes/Pasted%20image%2020240217213646.png)

Cambiamos la contraseña, y hemos ingresado como el usuario marcos, revisamos que podemos encontrar:

![](imagenes/Pasted%20image%2020240217214719.png)

Encontramos la bandera de usuario, ahora, o encontramos la forma de obtener una reverse shell, o continuamos usando la maquina windows. Personalmente pienso que usar la maquina directamente no es la forma en la que el creador pretendía que la resolveríamos la maquina.Entonces, investigo como obtener una reverse shell en windows. Ahora, recordemos, acabamos de restablecer la contraseña del usuario `marcos`, y parece ser que, el tiene acceso a los contenidos de la pagina.Podemos intentar ingresar como el usuario `marcos` con `smbclient`, pero antes que nada, echemos un vistazo a que shares tiene acceso el usuario `marcos`:

![](imagenes/Pasted%20image%2020240217220528.png)

Este va a ser nuestro pase para obtener privilegios elevados como `admin`, ingresemos como `marcos` usando `smbclient`:

![](imagenes/Pasted%20image%2020240217220736.png)

Intentemos subir un archivo con `put`:

![](imagenes/Pasted%20image%2020240217220824.png)

![](imagenes/Pasted%20image%2020240217220854.png)

Bien, esto significa, que definitivamente podemos subir un script que nos va a permitir obtener una revese shell en nuestra maquina, investiguemos como hacer eso. Me fijo especialmente en el servicio `asp_client`, después de investigar un poco, encuentro [esto](https://www.lifewire.com/what-is-an-aspx-file-2619705):

![](imagenes/Pasted%20image%2020240217221818.png)

Y en la [pagina hacktricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/msfvenom#asp-x) para nuestra suerte, hay una cheatsheet donde podemos encontrar el comando correcto para `msfvenom`, la cual creara el payload que vamos a subir a la pagina web:

![](imagenes/Pasted%20image%2020240217222025.png)

Generamos la payload, la subimos, y alistamos nuestro escucha en netcat, activamos la payload en la pagina, y vemos si obtenemos una reverse shell:

![](imagenes/Pasted%20image%2020240217222257.png)

![](imagenes/Pasted%20image%2020240217222306.png)

![](imagenes/Pasted%20image%2020240217222320.png)

*En este punto , me di cuenta que, con este payload, tengo que usar meterpreter, y como hacer las cosas manualmente, busco una reverse shell .aspx diferente*.

Encuentro [este](https://raw.githubusercontent.com/borjmz/aspx-reverse-shell/master/shell.aspx) script en github, lo descargo y cambio parámetros, lo subo, e intento otra vez:

![](imagenes/Pasted%20image%2020240217223723.png)

![](imagenes/Pasted%20image%2020240217223741.png)

![](imagenes/Pasted%20image%2020240217223752.png)

## Escalada de privilegios

![](imagenes/Pasted%20image%2020240217223817.png)

![](imagenes/Pasted%20image%2020240217223826.png)

Ahi esta, ahora, miremos como podemos escalar privilegios, corro el comando `whoami /priv`, para ver que privilegios posee con este usuario:

![](imagenes/Pasted%20image%2020240217224109.png)

Revisando mis notas, encuentro una forma de escalar privilegios usando `RogueWinRM`, en el [repositorio de github](https://github.com/antonioCoco/RogueWinRM) explica muy bien los pasos para usar este script. Vamos a usarlo por obtener una cmd shell con privilegios de `admin`, primero, subimos `RogueWinRM.exe` al folder escondido `C:\ProgramData` (también puede ser la carpeta `C:\windows\temp`).*Me tomo un rato encontrar una carpeta donde me permitiera descargar*. Ahora, hacemos lo siguiente:

![](imagenes/Pasted%20image%2020240217232535.png)

Parece que no funciona. En la pagina hacktricks, habían otros exploits que podíamos usar, revisemos. 

Después de un par de horas intentando escalar privilegios, finalmente, obtengo privilegios de administrador y obtengo la bandera:

![](imagenes/Pasted%20image%2020240218022802.png)

Para esto, usamos el [exploit](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/roguepotato-and-printspoofer#godpotato) `GodPotato` junto con `netcat` para poder una sesión con privilegios elevados:

![](imagenes/Pasted%20image%2020240218022314.png)

Entonces, se preguntaran: ¿que me tomo tanto tiempo? Bueno, la cosa es, el exe `nc64.exe` que descargue antes no funcionaba y bueno, no pense realmente que hubieran otros repositorios de donde pudiera descargar netcat.Descargue el primero de [aquí](https://github.com/int0x33/nc.exe/) , el cual, fue actualizado **hace 7 años**.Entonces por eso, estaba obteniendo un error acerca de que la version de netcat que estaba utilizando no era compatible con la version de windows de la maquina. Al final, busque otra version, y efectivamente, encontré [otra](https://github.com/vinsworldcom/NetCat64/releases) la cual era mas reciente, bueno, la fecha de lanzamiento por lo menos. Re-intente el proceso, y finalmente logre escalar privilegios.

¡Gracias por leer y buena suerte!

## Cosas que aprendí de esta maquina
- El uso de `smbclient`,`crackmapexec` y `smbmap` para enumeración de shares en los puertos `139` y `445`.En esa shares, es posible encontrar información de interés.
- Enumerar con diferentes herramientas, el mismo servicio, pueden haber diferencias en la información que arroje cada herramienta.
- Escalada de privilegios con el privilegio `SeImpersonatePrivilege` con el exploit `GodPotato`.
- Me trabe porque por `netcat` no funcionaba, pero el exploit estaba funcionando como debía. Si lo hubiera notado antes, hubiera resuelto la maquina mucho mas rápido.Tener en mente que si un programa no funciona en la maquina objetivo, buscar otra version del programa.