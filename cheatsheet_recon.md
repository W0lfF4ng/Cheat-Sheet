# Reconnaissance Cheat Sheet

## Mapeo de red

### Ping

Herramienta que permite enviar mensajes icmp echo request a un destino.

`ping [option] [target]`

Opciones Linux:

```
-c [count]: Cantidad de paquetes a enviar
-I [interface]: Interfaz a usar para enviar los mensajes ping
-s [bytes]: Especifica la cantidad de bytes a enviar
-v: Verbose
```

Opciones Windows:

```
-t: Ping continuo
-a: Resolver IP en FQDN
-n [count]: Cantidad de paquetes a enviar
```

### FPing

Herramienta de ping sweeping.

`fping -a -g [ip_range]`

```
-a: Muestra solo los hosts vivos
-g: Realiza un ping sweep
```

#### Ejemplo especificando CIDR y rango de hosts

`fping -a -g 192.168.1.0/24`

`fping -a -g 192.168.1.8 192.168.1.45`

#### Ejemplo eliminando los mensajes ICMP Host Unreachable

> El 2 equivale a los stderr (standard error).

`fping -a -g 192.168.1.0/24 2>/dev/null`

### Nmap

Herramienta de network mapping, permite enumerar dispositivos, servicios, OS, versiones, etc.

`nmap [scan type(s)] [options] [target]`

#### Opciones de escaneo básico

```
nmap [target]: Escaneo de los 1000 puertos más comunes
-iL [list_name]: Uso de un archivo con lista de objetivos
-iR [number]: Escaneo random de la cantidad de direcciones especificadas (no se indica un target)
--exclide [addresses]: Se realiza una exclusión de las direcciones a no escanear
--excludefile [file]: Lista con las direcciones a excluir
-A: Escaneo agresivo
-6: Escaneo a direcciones IPv6
```

#### Opciones de descubrimiento

```
-PS[port_list]: Descubrimiento mediante TCP SYN
-PA[port_list]: Descubrimiento mediante TCP ACK
-PU[port_list]: Descubrimiento mediante UDP
-PE: Descubrimiento mediante ICMP echo
-PP: Descubrimiento mediante ICMP timestamp
-PM: Descubrimiento mediante ICMP address mask
-PO[protocol_list]: Descubrimiento mediante IP protocol
-PY: Descubrimiento mediante SCTP init
-PR: Descubrimiento mediante ARP (Se usa de forma local, y necesita privilegios de root)
-sn: Escaneo de ping
-Pn/-PN: No realiza Ping
--traceroute: Realiza un traceroute al destino
-n: Deshabilitar resolución reversa de DNS
-R: Fuerza resolución reversa de DNS
--dns-server [dns_server]: Especifica los servidores DNS para realizar las resoluciones
--system-dns: Usa el sistema para resoluciones DNS (es un método lento)
-sL: DNS reverso a lista de escaneo
```

#### Opciones de escaneo avanzado

```
-sS: Escaneo mediante TCP SYN
-sT: Escaneo realizando la conexión TCP
-sU: Escaneo mediante UDP
-sN: Escaneo mediante TCP NULL (sin flags TCP habilitados)
-sF: Escaneo mediante TCP FIN
-sX: Escaneo Xmas, se envían paquetes TCP con flag URG, FIN y PSH habilitados
--scanflags [flags]: Escaneo con flags habilitados customizados (sin espacio entre los flags, y en MYCLS). Estos pueden ser: SYN, ACK, PSH, URG, RST, FIN
-sA: Escaneo mediante TCP ACK, puede indicar si hay un FW (RST = noFW = unfiltered)
-sO: Escaneo de los IP protocol soportados por el objetivo
```

> [IP protocol](http://iana.org/assignments/protocol-numbers)

#### Opciones de escaneo de puertos

```
-F: Escanea el top 100 de puertos más comunes
-p [port/name]: Se especifican los rangos de puertos o nombres, en los nombres se puede incluir un “*” como wildcard del nombre
-p U:[ports],T:[ports]: Se definen puertos UDP (U:) y/o TCP (T:). Es necesario especificar el tipo de escaneo a usar (-sU y/o -sT)
-p-: Escanea todos los puertos (se debe indicar el tipo de escaneo, por defecto es TCP)
--top-ports [number]: Se especifica el top “X” a escanear
-r: Realiza un escaneo secuencial de puertos
--open: Solo muestra los puertos abiertos, removiendo los cerrados y filtrados
```

#### Detección de servicios y sistema operativo

```
-O: Detección del OS/Fingerprint
-O --osscan-guess: Adivinar el OS de forma más agresiva
-O --osscan-limit: Limitar el escaneo de OS a los objetivos prometedores
-sV: Detección de las versiones de los servicios
-sV --version-trace: Muestra el verbose del escaneo de la versión
```

> [Fingerprint](http://nmap.org/submit/)

#### Opciones de tiempo

```
-T[0-5]: Velocidad de escaneo, 0 es el más lento, 5 el más rápido
```

#### Evasión de Firewalls

```
-f: Envío de paquetes fragmentados (paquetes de 8-B)
--mtu [bytes]: Se especifica la MTU de los paquetes a enviar (múltiples de 8)
-D [decoys]: Enmascara el escaneo usando uno o más decoys (RND:10 envía 10 random decoys)
-sI [zombie_ip_host]: Explota un sistema idle y lo usa para realizar el escaneo
--source-port [port]: Especifica el puerto de origen del escaneo
```

#### Opciones de output

```
-oN [file.txt]: Guarda el output en un archivo de texto
-oX [file.xml]: Guarda el output en un archivo XML
-oG [file.txt]: Guarda el output en un archivo que permite el GREP
-oA [filename]: Guarda el output en todos los tipos de archivos soportados (.gnmap, .nmap, .xml)
-oS [file.txt]: Guarda el output en un archivo de texto con lenguaje “juvenil” (para script kiddies)
```

#### Ejemplo de escaneo usando nmap

`nmap -Pn -sS -sV -sC -p- 192.168.1-50.1-254 -oA nmapTarget`

#### Ejemplo de escaneo de vulnerabilidades usando nmap

`nmap -Pn -n --script vuln 192.168.1.32 -oN vulns.txt`

### [Masscan](https://github.com/robertdavidgraham/masscan)

Herrmaienta para realizar escaneos masivos.

`masscan -p[ports] [target] [options]`

```
-Pn: No realiza Ping
-sS: Escaneo usando TCP SYN
-n: No realiza una resolución DNS
-p[ports]: Permite indicar que puertos escanear
--rate=[rate]: Cuantos paquetes por segundo se enviarán
--banners: Obtener el banner del target (fingerprint)
-e [interface]: Permite indicar desde que interfaz hacer el escaneo
--router-ip [ip_address]: Permite indicar desde que IP realizar el escaneo
```

#### Ejemplo de escaneo usando masscan

`masscan -p0-65535 -sS 192.168.1.1 –rate=10000 –e tun0 > masscanTarget.txt`
