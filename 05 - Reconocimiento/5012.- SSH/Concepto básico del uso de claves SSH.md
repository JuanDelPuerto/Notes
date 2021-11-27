Las claves SSH se pueden crear con: `ssh-keygen`. Pudiendo poner contraseña o no.
Con esta herramienta se crearán dos ficheros: id_rsa (clave privada) y id_rsa.pub (clave pública).
Para arrancar el servicio ssh: `service ssh start`.
Con estas claves se podrá acceder a nuestra máquina de forma remota.

Formas de utilizar estas claves:
1. La clave pública se puede depositar en otra máquina en la que tengamos acceso y renombrarla como 'authorized_keys' > `cp id_rsa.pub authorized_keys`
	Esto la convierte en una cave autorizada cuando es depositada en el directorio '.ssh'.
	Esto permitirá autenticarse en esa máquina de forma remota sin proporcionar credenciales válidas para tal máquina.
	`ssh s4vitar@<maquina_objetivo>`
	
2. Con la herramienta --> `ssh-copy-id -i id_rsa s4vitar@localhost`
	Esta tool ejecutada en la máquina objetivo crea una clave autorizada que es compartida con la máquina especificada, en este caso debería ser la nuestra de atacante.
	Esta herramienta debe usarse con un usuario válido porque se deberá proporcionar su contraseña.
	Para conectarse a la máquina remotamente --> `ssh s4vitar@localhost -i id_rsa`
	Se deberá proporcionar la clave privada que se ha creado anteriormente.
	
	
Permisos fichero "id_rsa" --> `chmod 600 id_rsa`