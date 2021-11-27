** Captura el tráfico de red con tcpdump **
`tcpdump -i tun0 -w Captura -v`

** Análisis de una captura de tráfico con tshark **
>Muestra de todos los paquetes http con destino, mostrado en formato json
>`tshark -r Captura -Y "ip.dst=<ip-destino> && http" -Tjson 2>/dev/null`	
>Filtro por el payload de los paquetes tcp
>`tshark -r Captura -Y "ip.dst=<ip-destino> && http" -Tfields -e tcp.payload 2>/dev/null`
>Capturar la data en hexadecimal de las peticiones y respuestas http
>`tshark -r Captura.cap -Y "http" -Tfields -e data.data 2>/dev/null | xxd -ps -r`
>`tshark -r Captura.cap -Y "http" -Tfields -e tcp.payload 2>/dev/null | xxd -ps -r`
>Enumerar el número de peticiones "GET" que se hacen contra un servidor con el script de "Nmap" http-enum
>`tshark -r Captura.cap -Y "http" -Tfields -e tcp.payload 2>/dev/null | xxd -ps -r | grep "GET" | awk '{print $2}' | sort -u | wc -l`

