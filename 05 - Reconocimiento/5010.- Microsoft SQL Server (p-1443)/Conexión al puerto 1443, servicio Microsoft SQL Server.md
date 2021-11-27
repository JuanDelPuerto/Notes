** SQSH **
Herramienta "sqsh" (SQShell), sustituta de "isql", cliente SQL para bases de datos relacionales. En esencia, estas herramientas realizan transacciones SQL en la CLI contra la base de datos de un servidor.

Uso:
> Prueba para verificar si usuario por defecto 'sa' está habilitado.
> `sqsh -S 10.10.10.125 -U 'sa'`


** mssqlclient.py **
Herramienta dentro de la suite de Impacket (https://github.com/SecureAuthCorp/impacket).
Impacket es uan colección de clases en python para trabajar con protocolos de red.

Uso:
> Comprobar credenciales para conectarnos a la base de datos del servidor:
> `mssqlclient.py WORKGROUP/<usuario>:<contraseña>@10.10.10.125 -db volume`
> Probando con autenticación de Windows sobre la base de datos en vez de la autenticación SQL Server:
> `mssqlclient.py WORKGROUP/<usuario>:<contraseña>@10.10.10.125 -db volume -windows-auth`

** Consultas en Microsoft SQL **
> Enumerar las tablas existentes
> `SELECT TABLE_NAME FROM orcharddb.INFORMATION_SCHEMA_TABLES WHERE TABLE_TYPE = 'BASE_TABLE';`
> `GO -m CSV > output.csv`  		//Método de sqsh para exportar los resultados en formato csv
> Usar tablas
> `USE orcharddb;`
> `GO`
> `SELECT * FROM blog_Orchard_Users_UserPartRecorded;`
> `GO -m HTML > index.html`