```bash

#!/bin/bash
declare -r file = "message.decrypted"
while read password; do
	openssl aes-256-cbc -d -in message.crypted -out message.decrypted -k $password 2>/dev/null
	
	if ["$(echo$?)" == "0"]; then
		echo -e "\n[*]La password es $password" && cat $file
		rm $file && break
	fi
	
done < /usr/share/wordlists/rockyou.txt 

```
=================
Funcionamiento:
Se declara un archivo de sólo lectura message.decrypted
El fichero message.crypted se desencripta con openssl 
 if ["$(echo$?)" == "0"] --> Si el símbolo de estado es 0
 El iterador while lee cada línea del fichero rockyou.txt y la convierte en la variable password
 