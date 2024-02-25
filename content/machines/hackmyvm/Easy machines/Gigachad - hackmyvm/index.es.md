+++
title = "Gigachad"
date = 2024-02-24T21:56:29-06:00
summary="Easy machine, a bit of OSINT, and exploit research."
draft=false
tags=["",""]
series=["Hackmyvm Fácil"]
series_order=7
+++
## Prefacio
Como dije en mis post previo, he estado un poco ocupado con la universidad, pero, todavía sigo aprendiendo de hacking ético en mi tiempo libre. De todas formas, sigamos con la resolucion de esta maquina fácil de tasiyanci.


## Información de la maquina
| Name       | Gigachad                                                         |
| ---------- | ---------------------------------------------------------------- |
| Sistema operativo        | Linux Debian                                                     |
| Autor     | tasiyanci                                                        |
| Sitio       | [hackmyvm](https://hackmyvm.eu/machines/machine.php?vm=Gigachad) |
| Dificultad | Fácil                                                             |

## Reconocimiento

Verificamos la conexión con la maquina:


![](imagenes/Pasted%20image%2020240224193655.png)

![](imagenes/Pasted%20image%2020240224193743.png)

Corremos nuestro escaneo de nmap:

```shell-session
Nmap 7.94SVN scan initiated Sat Feb 24 19:35:25 2024 as: nmap -oN gigachad_scan.txt -Pn -vv -sS -sC -sV -T3 -p- 192.168.25.29
Nmap scan report for 192.168.25.29
Host is up, received arp-response (0.00038s latency).
Scanned at 2024-02-24 19:35:25 CST for 11s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r-xr-xr-x    1 1000     1000          297 Feb 07  2021 chadinfo
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.25.5
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6a:fe:d6:17:23:cb:90:79:2b:b1:2d:37:53:97:46:58 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4uqqKMblsYkzCZ7j1Mn8OX4iKqTf55w3nolFxM6IDIrQ7SV4JthEGqnYsiWFGY0OpwHLJ80/pnc/Ehlnub7RCGyL5gxGkGhZPKYag6RDv0cJNgIHf5oTkJOaFhRhZPDXztGlfafcVVw0Agxg3xweEVfU0GP24cb7jXq8Obu0j4bNsx7L0xbDCB1zxYwiqBRbkvRWpiQXNns/4HKlFzO19D8bCY/GXeX4IekE98kZgcG20x/zoBjMPXWXHUcYKoIVXQCDmBGAnlIdaC7IBJMNc1YbXVv7vhMRtaf/ffTtNDX0sYydBbqbubdZJsjWL0oHHK3Uwf+HlEhkO1jBZw3Aj
|   256 5b:c4:68:d1:89:59:d7:48:b0:96:f3:11:87:1c:08:ac (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDkds8dHvtrZmMxX2P71ej+q+QDe/MG8OGk7uYjWBT5K/TZR/QUkD9FboGbq1+SpCox5qqIVo8UQ+xvcEDDVKaU=
|   256 61:39:66:88:1d:8f:f1:d0:40:61:1e:99:c5:1a:1f:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIoK0bHJ3ceMQ1mfATBnU9sChixXFA613cXEXeAyl2Y2
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
| http-robots.txt: 1 disallowed entry 
|_/kingchad.html
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
MAC Address: 08:00:27:AC:C9:45 (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```

| Puertos abiertos | 21,22,80     |
| ---------- | ------------ |
| Servicios   | FTP,SSH,HTTP |
Bien, parece que nos podemos conectar al servicio ftp, y con un login anónimo, revisemos:

![](imagenes/Pasted%20image%2020240224194509.png)

Y si pudimos, revisemos que información podemos recopilar:

![](imagenes/Pasted%20image%2020240224194634.png)

Revisamos ese directorio en la pagina web:

![](imagenes/Pasted%20image%2020240224194704.png)

Una imagen, la descargamos. Pero, antes que nada, reviso el código fuente de la pagina y encuentro esto:

![](imagenes/Pasted%20image%2020240224194752.png)

Parece ser algún tipo de hash, un hash `md5` para ser mas preciso, revisemos si lo es en [hashes.com](https://hashes.com) :

![](imagenes/Pasted%20image%2020240224194856.png)

Jeje, parece que lo es, ahora, ¿tal vez la imagen tiene hashes escondidas? Corramos `exiftool` para revisar:

![](imagenes/Pasted%20image%2020240224200120.png)

>Pense que los parámetros `Document ancestors` y `Derived from Original Document ID` eran hashes, pero no lo eran.Ahora, antes de sobre complicar las cosas, tal vez la contraseña es el lugar en la foto. En lo personal , pienso que es `Estambul`, entonces, rápidamente con google maps, encuentro el nombre del lugar en la foto:

![](imagenes/Pasted%20image%2020240224201240.png)

![](imagenes/Pasted%20image%2020240224201504.png)

![](imagenes/Pasted%20image%2020240224201649.png)

Ahora, algunas posibles contraseñas son:
- `maidenstower`
- `kizkulesi`
- `istanbul`
- `leanderstower`

## Acceso inicial y escalada de privilegios
Podemos crear una lista para hacer ataque de fuerza bruta con `hydra` o solo intentamos manualmente. Después de intentar manualmente, encuentro que la contraseña correcta es `maidenstower`:

![](imagenes/Pasted%20image%2020240224202200.png)

Encuentro la bandera y empiezo a buscar como escalar privilegios:

![](imagenes/Pasted%20image%2020240224202236.png)

Primero, revisemos el directorio en la pagina web, para ver si encontramos algo interesante:

![](imagenes/Pasted%20image%2020240224203540.png)

Un escaneo con `gobuster` no hubiera arrojado nada util en esta maquina... Bueno, revisemos el archivo `robots.txt`:

![](imagenes/Pasted%20image%2020240224203655.png)

Revisemos la pagina web:

![](imagenes/Pasted%20image%2020240224203728.png)

Nada util. Corramos el comando `find / -perm -4000 2>/dev/null`, para ver si hay algo fuera de lo ordinario que podamos explotar:

![](imagenes/Pasted%20image%2020240224212720.png)

Este archivo no es común. Pego una ojeada en [gtfobins](https://gtfobins.github.io) y no encuentro nada.Pero, en [exploit-db](https://www.exploit-db.com/exploits/47172), encuentro algo interesante, a bash script(que corre un código en `c` dentro de el) para escalar privilegios localmente:

Aquí esta como trabaja:

![](imagenes/Pasted%20image%2020240224213022.png)

Y asi se mira cuando el exploit se completa exitosamente:

![](imagenes/Pasted%20image%2020240224213109.png)

Como ya estamos en el directorio `/tmp`, creamos un archivo `.sh`, le asignamos binarios de ejecución, y veamos si obtenemos root.

![](imagenes/Pasted%20image%2020240224214939.png)

¡Obtenemos root! Encontremos la bandera de root y terminemos la maquina:

![](imagenes/Pasted%20image%2020240224215524.png)

Y con eso, hemos comprometido esta maquina. ¡Gracias por leer!
