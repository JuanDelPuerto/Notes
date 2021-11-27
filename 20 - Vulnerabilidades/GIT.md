Git es un software de control de versiones diseñado por Linus Torvalds, pensando en la eficiencia, la confiabilidad y compatibilidad del mantenimiento de versiones de aplicaciones cuando estas tienen un gran número de archivos de código fuente). Su propósito es llevar registro de los cambios en archivos de computadora incluyendo coordinar el trabajo que varias personas realizan sobre archivos compartidos en un repositorio de código.

==================================================================

Órdenes básicas
-   Buscar repositorios GIT en Linux: `find /-name \.git 2>/dev/null`
-   Ver actualizaciones del proyecto GIT: `git log`
-   Ver cambios en alguna actualización: `git log -p <id>`

-   `git init`: Esto crea un subdirectorio nuevo llamado .git, el cual contiene todos los archivos necesarios del repositorio – un esqueleto de un repositorio de Git. Todavía no hay nada en tu proyecto que esté bajo seguimiento.
-   `git fetch`: Descarga los cambios realizados en el repositorio remoto.
-   `git merge _<nombre_rama>_`: Impacta en la rama en la que te encuentras parado, los cambios realizados en la rama “nombre_rama”.
-   `git pull`: Unifica los comandos _fetch_ y _merge_ en un único comando.
-   `git commit -m "<mensaje>"`: Confirma los cambios realizados. El “mensaje” generalmente se usa para asociar al _commit_ una breve descripción de los cambios realizados.
-   `git push origin _<nombre_rama>_`: Sube la rama “nombre_rama” al servidor remoto.
-   `git status`: Muestra el estado actual de la rama, como los cambios que hay sin commitear.
-   `git add _<nombre_archivo>_`: Comienza a trackear el archivo “nombre_archivo”.
-   `git checkout -b _<nombre_rama_nueva>_`: Crea una rama a partir de la que te encuentres parado con el nombre “nombre_rama_nueva”, y luego salta sobre la rama nueva, por lo que quedas parado en esta última.
-   `git checkout -t origin/_<nombre_rama>_`: Si existe una rama remota de nombre “nombre_rama”, al ejecutar este comando se crea una rama local con el nombre “nombre_rama” para hacer un seguimiento de la rama remota con el mismo nombre.
-   `git branch`: Lista todas las ramas locales.
-   `git branch -a`: Lista todas las ramas locales y remotas.
-   `git branch -d _<nombre_rama>_`: Elimina la rama local con el nombre “nombre_rama”.
-   `git push origin _<nombre_rama>_`: Commitea los cambios desde el branch local origin al branch “nombre_rama”.
-   `git remote prune origin`: Actualiza tu repositorio remoto en caso que algún otro desarrollador haya eliminado alguna rama remota.
-   `git reset --hard HEAD`: Elimina los cambios realizados que aún no se hayan hecho _commit_.
-   `git revert _<hash_commit>_`: Revierte el _commit_ realizado, identificado por el “hash_commit”.

==================================================================

### Reconstruir ficheros de proyecto con repositorio Git 
* Si se encuentra un recurso .git en un servicio web, siempre podemos tratar de descargarlo: `git clone http://10.10.187.200/.git`. 
* Comprobar que se puede descargar o acceder al fichero HEAD, típico de los repositorios de Git, con los logs de los commit. `curl -s -X GET http://10.10.187.200/.git/logs/HEAD`
* Reconstruir el repositorio gracias a los commit del HEAD:
	1. Construir un repositorio nuevo: `git init`
	2. Acceder a los HEAD: `curl -s -X GET http://10.10.187.200/.git/logs/HEAD`
	3. Se deben reconstruir los objetos en la carpeta objects del repositorio local: `cd /.git/objects`
	4. Descargar el objeto mediante su identificador:
		1. Ejemplo de identificador: 79c9539b6566........
		2. Se debe crear una carpeta con los dos primeros dígitos del hash del objeto: `mkdir /.git/objects/79`
		3. Descarga del objeto en su carpeta identificativa:  `curl -s -X GET http://10.10.187.200/.git/objects/79/<hash_sin_los_dos_primeros_digitos> -o <hash_sin_los_dos_primeros_digitos>`
		4. Podemos comprobar qué tipo de data es el fichero descargado: `file <hash_sin_los_dos_primeros_digitos>`. En este caso se debería tratar de ficheros zlib compressed data.
		5. Para ver el fichero u objeto descargado debe verse desde la raiz del proyecto: `git cat-file -p <hash_con_los_dos_primeros_digitos>`
		6. El leer un commit nos dará un objeto "tree" con un identificador, este identificador será la información que ha cambiado en cada fichero en este commit.
		7. Leer u obtener tanto el tree como los ficheros que han sido commiteados siguen el mismo proceso:
			1.  `curl -s -X GET http://10.10.187.200/.git/objects/<dos_primeros _digitos>/<hash_sin_los_dos_primeros_digitos> -o <hash_sin_los_dos_primeros_digitos>`
			2.  `git cat-file -p <hash_con_los_dos_primeros_digitos>`