** Port knocking (Golpeo de puertos) **
Mecanismo para abrir puertos externamente en un firewall mediante una secuencia preestablecida de intentos de conexión a puertos que se encuentran cerrados. Una vez que el firewall recibe una secuencia de conexión correcta, sus reglas son modificadas para permitir al host que realizó los intentos conectarse a un puerto específico.

El "Port knocking" consiste en establecer ciertas reglas de firewall que permiten abrir un puerto temporalmente cuando una secuencia pre-establecida de puertos es golpeado/escaneado/intento de conectar.

** Port knocking con Nmap **
> Primera prueba sobre puerto 22, da como resultado filtrado:
> `nmap -p22 -T5 -v -n 10.10.10.83`
> Golpeando los puertos, se le incluye -r para que el orden establecido sea el determinado. Puerto 22 abierto.
> `nmap -p3456,8234,62431,22 -T5 -v -n 10.10.10.83 -r`
> Golpeo de puerto y conexión directa a ssh con credenciales válidas:
> `nmap -p3456,8234,62431 -T5 -v -n 10.10.10.83 -r; ssh prometheus@10.10.10.83`


