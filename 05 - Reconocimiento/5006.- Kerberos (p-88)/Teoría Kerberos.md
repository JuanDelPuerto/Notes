Kerberos es un protocolo de autenticación, pero no de autorización. El protocolo se encarga de identificar a cada usuario a través de una contraseña sólo conocida por éste, pero no determina a qué recursos o servicios puede acceder o no.

Elementos que forman parte de Kerberos:
1. Capa de transporte --> Protocolos UDP y TCP, ambos en el puerto 88.
2. Agentes: varios servicios encargados de realizar la autenticación del usuario.
		1. Cliente o usuario.
		2. AP (Application Server), donde se expone el servicio al que el usuario quiere acceder.
		3. KDC (Key Distribution Center), servicio de Kerberos instalado en el DC (Domain Controller), encargado de distribuir los tickets.
		4. AS (Authentication Service), servicio de KDC encargado de expedir los TGT's
3. Claves de cifrado
		1. Clave del KDC o Krbtgt --> clave derivada del hash NTLM de la cuenta krbtgt
		2. Clave de usuario --> clave derivada del hash NTLM del usuario.
		3. Clave de servicio --> clave derivada del hast NTLM del propietario del servicio (usuario o servidor).
		4. clave de sesión --> clave negociada por el cliente y el KDC.
		5. Clave de sesión de servicio --> clave negociada para utilizar entre el cliente y el AP.
4. Tickets
		1. TGS --> (Ticket Granting Service), ticket que se presenta ante un servicio para poder aaceder a sus recursos. Se cifra con la clave del servicio correspondiente.
		2. TGT --> (Ticket Granting Ticket), ticket que se presenta ante el KDC para obtener los TGS. Se cifra con la clave del KDC.
5. PAC (Privilege Atribute Certificate)
		Estructura contentida en la mayoría de los tickets, contiene los privilegios del usuario y está firmada con la clave dle KDC. La verificación del PAC sólo consiste en comprobar su firma, sin comprobar si los privilegios son correctos. 
		
		
** Ataques a Kerberos **
* PTK ( Pass the key / Overpass the key ):
		El ataque utiliza el hash del usuario par conseguir suplantarlo frente al KDC y obtener acceso a los servicos del dominio disponibles para dicho usuario.
		Los hashes de usuario se pueden extraer de los ficheros SAM de las estaciones de trabajo, el fichero NTDS.DIT de los DC, o de la memoria del proceso lsass.
* PTT ( Pass the ticket ):
		Se trata de obtener un ticket de usuario y utilizarlo para ganar acceso a los recursos por los que el usuario tenga permiso. Se ha de conseguir también la clave de sesión respectiva, para poder usar el ticket en las comunicaciones con el servicio.
		Estos tickets se podrían conseguir mediante un MiM (Man in the middle) o en la memoria del proceso lsass.
* Golden Ticket:
		El objetivo es construir un TGT. Se necesita la clave del krbtgt, se necesitará obtener el hash NTLm de la cuenta krbtgt. Este TGT podría contar con la caducidad y permisos que se deseen, incluso con privilegios de administrador del dominio.
* Silver Ticket:
		Se contruye un TGS, por loq eu se requiere la clave del servicio al que se quere acceder. Se debe obtener el hash NTLM de la cuenta propietaria del servicio.
* Kerberoasting:
		Se trata de usar los TGS para realizar cracking de las contraseñas de los usuarios de forma offline. Los TGS vienen cifrados con la clave del sercicio, que se deriva del hash NTLm de la cuenta propietaria del servicio. Si los servicios son propiedad de las cuentas del equipo en el que se ejecutan (lo más normal), estas contraseñas serán muy robustas para ser crackeadas, aplicando esto también a la cuenta krbtgt, por lo que tampoco será viable crackear el TGT.
		Pero si lo servicios son propiedad de algún usuario humano, éste seguro tendrá permisos de administrador y será más probable crackear su contraseña.
* ASREPROAST:
		Técnica similar al kerberoasting, que busca crackear de forma offline las credenciales. Este ataque se puede llevar a cabo si un usuario tiene configurado el atributo "DONT_REQ_PREAUTH", no necesita preautenticación, con lo que sería posible construir un mensaje "KRB_AS_REQ" sin conocer credenciales.
		Una vez construido y enviado, el KDC  responderá con un mensaje "KRB_AS_REP" que contiene datos cifrados con el hash de este usuario, pudiendo ser utilizados para crakeo offline.