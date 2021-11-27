** H2 Database 1.4.196 - Remote Code Execution **
https://www.exploit-db.com/exploits/45506
Este script toma ventaja de la la vulnerabilidad 'H2 Database - 'Alias' Arbitrary Code Execution' (https://www.exploit-db.com/exploits/44422/).
Cuando se base de datos test tiene una contraseña desconocida, es posible provocar una ejecución de código creando una nueva base de datos. La consola web permite introducir el nombre de la nueva base de datos y un cadena de conexión. 
Como hemos podido crear una nueva base de datos y sabemos que las credenciales por defecto al crear una nueva base de datos son username="sa" y password="", el atacante puede loguearse automáticamente.

Ejecución 
>`python3 exploit.py -H 10.10.10.102:8082 -d jdbc:h2:˜/loquesea`
>Se obtiene una reverse shell con permisos de administrador


