** Enumerar directorios de un servidor web **
`nmap --script http-enum -p80 <ip> -oN webScan`
	
** Ver las peticiones que realiza el script http-enum **
> Capturamos los paquetes relacionados con el escaneo.
> `tcpdump -i tun0 -v -w Capture`
> Vemos qué pesa la captura
> `du -hc Capture`
> Vemos la captura en bruto
> `tshark -r Capture 2>/dev/null`
> Filtrado de la captura para ver sólo paquetes http a una dirección determinada
> `tshark -r Capture -Y "http && ip.dst==10.10.10.78" -Tfields -e "tcp.payload" 2>/dev/null`
> Parseo de la salida en hexadecimal del tcp.payload
> `tshark -r Capture -Y "http && ip.dst==10.10.10.78" -Tfields -e "tcp.payload" 2>/dev/null | xxd -ps -r`
> Grepeo sobre los componentes GET y HEAD
> `tshark -r Capture -Y "http && ip.dst==10.10.10.78" -Tfields -e "tcp.payload" 2>/dev/null | xxd -ps -r | grep -E "GET|HEAD"`
> Ordenar y contar número de líneas
>  `tshark -r Capture -Y "http && ip.dst==10.10.10.78" -Tfields -e "tcp.payload" 2>/dev/null | xxd -ps -r | grep -E "GET|HEAD" | sort -u | wc -l`
	
** Buscar scripts de nmap y sus categorías **
`locate .nse | xargs grep -r "categories" | grep -oP  '".*?*"' | sort -u`

** Buscar vulnerabilidades sobre el puerto 445 de SMB **
`nmap --script "vuln and safe" -p445 10.10.10.45 -oN vulnScan`

** Enumeración de usuarios del AD vía krb5-enum-users **
`nmap -p88 --script krb5-enum-users --script-args krb4-enum-users.realm='htb.local', userdb=/opt/SecLists/Usernames/Names/names.txt 10.10.10.52 -oN kerberos_user_enum`

** Enumeración de vulnerabilidades en el protocolo SSL **
`nmap --script "vuln and safe" -p443 10.10.10.79 -oN sslCheck`

** Enumeración de LDAP **
`nmap --script ldap-search -p389 10.10.10.192 -oN ldapScan`


