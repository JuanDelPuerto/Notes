```
impacket-smbserver smbFolder $(pwd)
```

Dar soporte para SMB2 y creando un usuario y contraseña.
`impacket-smbserver smbFolder $(pwd) -smb2support -username s4vitar -password s4vitar`

Autenticarse desde Windows (máquina objetivo) a servidor SMB en en máquina local.
`net use \\10.10.14.6\smbFolder /u:s4vitar s4vitar`


