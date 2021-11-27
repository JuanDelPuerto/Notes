** Nikto **
`nikto -h http://10.10.10.103`


** Python en modo consola **
>>> import requests
>>> r = requests.get ('http://10.10.10.83')
>>> dir(r)																			//mostrará todos los atributos de la petición
>>> print r.text																//imprimirá la respuesta del servidor
>>> print r.headers															//veremos las cabeceras


** Búsqueda de vulnerabilidades sobre SSL (p-443) **
https://github.com/drwetter/testssl.sh



