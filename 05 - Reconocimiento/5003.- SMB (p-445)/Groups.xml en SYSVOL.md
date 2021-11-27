En muchos DC (Domain Controlers), si tenemos acceso mediante SMB al recurso compartido SYSVOL podemos encontrar una contraseña hasheada en el archivo Groups.xml.

Esta contraseña puede ser desencriptada gracias a la herramienta "gpp-decrypt" porque Microsoft en su momento compartió la clave AES.

En Windows, en SYSVOL se crea un archivo xml cada vez que se crea una GPO. Este archivo puede guardar las credenciales del usuario que crea la GPO, normalmente el usuario Administrator. Esta contraseña está encriptada con AES256, pero Microsoft liberó la clave.

Explotación GPP SYSVOL:
https://vk9-sec.com/exploiting-gpp-sysvol-groups-xml/

Este fichero suele encontrarse en:
Policies\{xxxx}\Preferences\Groups\Groups.xml

Si este fichero podemos verlo a través de SMB, podemos traerlo mediante: get Groups.xml

![[Pasted image 20210627161950.png]]

Encontramos un campo 'cpassword' con la contraseña encriptada. Para desencriptar:
`gpp-decrypt <contraseña>`

Herramienta gpp-decrypt.rb:
`ruby gpp-decrypt.rb 2>/dev/null <contraseña>`

Con la contraseña del usuario Administrator podemos conectarnos gracias a "psexec.py".

