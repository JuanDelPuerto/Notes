** LdapDomainDump **
https://github.com/byt3bl33d3r/ldapdomaindump
En Active Directory, mucha información interesante puede ser extraida via LDAP por cualquier equipo o usuario autenticado en el dominio. Esto hace de LDAP un protocolo interesante para obtener información en la fase de reconocimiento de un pentesting a una red interna. El problema de esta data es que a menudo no está disponible en un formato fácil de leer.

ldapdomaindump es una herramienta que ayuda a solventar este problema coleccionando y parseando la información disponible via LDAP y dando como salida a esta información en un formato HTML, json, csv/tsv/greppable, etc.

Funcionamiento:
>Clonamos el repositorio
>`git clone https://github.com/byt3bl33d3r/ldapdomaindump`
>Instalamos la herramienta
>`python3 setup.py install`
>Ejecutamos la herramienta
>`python3 ldapdomaindump.py -u 'htb.local\amanda' -p '<contraseña>' 10.10.10.103 -o /var/www/html`
>Abrimos el servidor Apache para poder visualizar los datos desde el navegador
>`service apache2 start`
>Accedemos a localhost para ver los datos