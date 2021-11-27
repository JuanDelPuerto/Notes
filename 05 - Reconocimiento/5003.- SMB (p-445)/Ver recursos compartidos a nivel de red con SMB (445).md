** Ver información del equipo a través de SMB **
`crackmapexec smb 10.10.10.10` || `crackmapexec smb 10.10.10.10/24`

** Mapeo de directorios compartidos con SMB y sus permisos para un Null Session **
`smbmap -H 10.10.10.40`

** Ver recursos compartidos de forma redundante **
`smbmap -H 10.10.10.10 -R <directorio>`
`smbmap -H 10.10.10.10 -r Replication/active.htb/Policies/{xxxxx}`

** Ver recursos compartidos con SMB a través de un NULL session **
`smbmap -H 10.10.10.40 -u "null"`
`smbclient -L 10.10.10.40 -N 2>/dev/null`

** Ver recursos compartidos con CME **
`cme smb 10.10.10.52 -u 'admin' -p 'm$$ql_S@_P@ssWOrd!' --shares`

** Autenticarse en la máquina objetivo contra los recursos compartidos que estén disponibles a través de un NULL session **
`smbclient //10.10.10.40/users -N`

** Ver recursos compartidos aportando credenciales **
`smbclient -L 10.10.10.52 -U <usuario>%<contraseña> 2>/dev/null`

** Ver permisos de los recursos compartidos **
>smbcacls --> herramienta que manipula las NT ACLs (Access Control Lists) en SMB sobre los recursos compartidos, que define los permisos y restricciones sobre un usuario o grupo
>`smbcacls //10.10.10.40/users Default/Desktop -N`
>Podemos filtrar los resultados con una expresión regular
>`smbcacls //10.10.10.40/users Default/Desktop -N | grep -i everyone`

** Montar directorio compartido en Linux para ver recursos **
>Ir al directorio /mnt
>Crear directorio a compartir
>`mkdir smbMounted`
>Montar el directorio compartido con permisos de escritura y lectura
>`mount -t cifs //10.10.10.40/user /mnt/smbMounted -o username=null,password=null,domain=WORKGROUP,rw`
>mount.cifs o mount -t cifs {servicio} {punto de montaje} {-o opciones}
>Comando nativo del kernel de linux, cifs es el sucesor del protocolo SMB. Esta utilidad adjunta o monta el recurso de red compartido en un servidor remoto, especificado como //servidor/recurso
>Sobre el punto de montaje /mnt/smbMounted/Users podemos movernos entre directorios.
>`ls -la /mnt/smbMounted/Users`
>Desmontar montura
>`umount /mnt/smbMounted`

** Ver permisos reales de los directorios compartidos con función en bash **
> `for file in $(ls); do echo $file; done | grep -v -i "^ntuser" | while read line; do echo -e "\n[*]$line:\n"; smbcacls //10.10.10.40/Users Default/$line -N | grep -i "everyone"; done`
> Lee cada línea que devuelve el comando "ls" la muestra "echo $file"
> Filtra con "grep" para elimintar "-v", case-insensitive "-i", elimina la línea que comienza "^" con "ntuser"
> Nuevo bucle "while-do" lee cada línea
> Ejecuta "smbcacls" sobre cada recurso que va obteniendo por línea
> Filtra por "everyone", case-insensitive "-i"

** Búsqueda recursiva para listar el contenido de cada directorio en el servidor  sin credenciales **
>`smbclient -L 10.10.10.103 -N 2>/dev/null | grep "disk" | sed 's/\s*\(.*\)\s*Disk.*/\1/' | while read sharedFolder; do echo "===${sharedFolder}==="; smbclient -N "//10.10.10.103/${sharedFolder}" -c dir; echo; done`

** Búsqueda recursiva para listar el contenido de cada directorio en el servidor con credenciales **
>`echo; smbclient -L 10.10.10.52 -U <usuario>%<contraseña> 2>/dev/null | grep "disk" | awk '{print $1}' | while read sharedFolder; do echo "===${sharedFolder}==="; smbclient //10.10.10.103/${sharedFolder} -U <usuario>%<contraseña> -c dir; echo; done`

** Listar de forma recursiva los directorios a los que tenemos permiso **
Buscamos todo desde la carpeta actual (.). Bucle que mostrará por pantalla sólo los directorios en lso que el signo de estado en la ejecucion del comando sea igual a 0, es decir, ejecución correcta.
>`find . -type d | while read directory; do`
>`touch ${directory}/s4vitar 2>/dev/null && echo "${directory} - Archivo creado" && rm ${directory}/s4vitar`
>`mkdir ${directory}/s4vitar 2>/dev/null && echo "${directory} - Directori creado" && rm ${directory}/s4vitar`

** Comprobar si hay extensiones "blacklist" creadas **
Intentamos crear un conjunto de archivos con diferentes extensiones en los directorios que se tiene acceso.
>`touch ${mount/smbMounted/<directorio>/,./}s4vitar.{exe,dll,scf}`
>Comprobamos si estos recursos se han creado

** Descargar de forma recursiva todos los recursos con 'smbclient' **
```
smbclient -Udomainname/fordodone //10.234.92.21/sharename
Password:
Domain=[DOMAINNAME] OS=[Windows 5.0] Server=[Windows 2000 LAN Manager]
smb: \> cd testdir
smb: \testdir\> get C
NT_STATUS_FILE_IS_A_DIRECTORY opening remote file \testdir\C
smb: \testdir\> prompt
smb: \testdir\> recurse
smb: \testdir\> mget C
getting file ...
```

** Comandos básicos de SMB **
> Autenticación sin aportar credenciales a un recurso determinado:
> `smbclient //10.10.10.125/Reports -N`
> Traernos un fichero a nuestra máquina local:
> `get "Currency Volume Reports.xlsm"`
> Salir de SMB:
> `exit`

** Montar servidor SMB compartido **
`impacket smbserver smbFolder $(pwd) --smb2support`





