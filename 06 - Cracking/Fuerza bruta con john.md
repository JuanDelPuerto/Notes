** Crackear claves id_rsa con contraseña **
> El contenido de id_rsa lo transformamos en un hash que john pueda manejar
> `ssh2john.py id_rsa > hash`
> Crackear hash
> `john --wordlist=rockyou.txt hash`
> Ver en qué línea del rockyou.txt se encuentra la contraseña
> `grep -n '<contraseña>' /usr/share/wordlists/rocktyou.txt`



