Búsqueda de binarios con permisos SUID:
`find /-perm -4000 2>/dev/null`

Búsqueda de ficheros con palabras clave:
`grep -r -i -E "user|pass|key"`
		-r recursive, -i case insensitive, -E multiple matching
		
Búsqueda de binarios ejecutables como root:
`sudo -l`

Búsqueda de archivos interesantes en un servidor corriendo Drupal:
`find /-type f 2>/dev/null | grep -v -E "themes|modules"`
		-type f --> tipo ficheros, no directorios.
		
Búsqueda a nivel de fichero donde aparezca la palabra "hugo", propietario del user.txt, para poder realizar 'user-pivoting':
`grep -r -i "hugo" 2>/dev/null`

User pivoting a un usuario como el que podemos ejecutar cualquier comando, aún sin tener su contraseña:
`sudo -u shaun bash`


