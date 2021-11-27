tput civis		//ocultar cursor
tpuc cnorm		//hacer visible el cursor

locate 			//localiza un fichero en el sistema
which			//busca y muestra la información de un script

chmod +x lse.sh 	//Dar privilegios de ejecución a un fichero en bash

//Ejecuar un fichero en bash
>`./lse.sh`
>`bash lse.sh`

** Bucle en bash **
Ejecuta 100 el fichero cookie.php, lo muestra por salida y ordena sus resultados
	`for in in $(seq 1 100); do php cookie.php; echo; done | sort -u`

** Función para transformar los puertos en formato hexadecimal del fichero /proc/net/tcp **
`for port in $(curl -s "http://localhost/example.php?file=/proc/net/tcp" | awk "{print $2}" | grep -v "local_address" | awk "{print $2}" FS=":" | sort -u); do echo "[$port] --> Puerto $(echo "ibase=16;$port" | bc)"; done`


