** Obtener puertos de una captura grepeable de nmap **
> `cat top5000Ports | grep -oP '\d{1,5}/open'`

** Buscar por la data entre comillas **
> `grep -oP '".*?"'`

** Buscar scripts de nmap y sus categorías **
> `locate .nse | xargs grep -r "categories" | grep -oP  '".*?*"' | sort -u` 

** Buscar usuarios con una shell en sistemas Linux, líneas que terminen en 'sh' **
> `grep "sh$" /etc/passwd`

** Flags de grep **
> `-oP` 				//-o only matching, -P perl-regex
> `-v`					// -v borra

** Eliminar con cut **
> `cut -d '<' -f 1` 	// Elimina la primera coincidencia de '<' en cada línea

** AWK, manipulador de textos **
> `| awk '{print $!}' FS='/'`
> `| awk '{print $3}' FS='>'` 	// Obtener el tercer elemento delimitado por >

** XARGS **
> `| xargs` 	// Concentrar toda la salida en una sola línea
> `| xargs cat` 	//Por cada línea resultante, realizar un comando, por ejemplo una lectura de cada fichero

** TR, elimina/cambia el primer argumento por el segundo **
> Cambia los espacios en blanco por una coma
> `| tr '' ','`
> Elimina las comas en el texto
> `| tr -d ','`
> Quitar saltos de línea
> `| tr -d '\n'`
> Cambiar las comas por saltos de línea
> `| tr ',' '\n'`

** sed **
> Quitar espacios en blanco al principio y final de cada línea
> `sed '/^\s*$/d'`
> `sed s/^*//`

** Formatear salida de lectura del fichero allPorts **
> Este fichero contiene los puertos abiertos con un escaneo con nmap
> `cat allPorts | grep -oP '\d{1,5}/open' | awk '{print $!}' FS='/' | xargs |  tr '' ','`

** Quitar espacios y convertirlos en una nueva línea **
> `sed 's/\s\+/\n/g' id_rsa`

** Uso de "sed" para cambiar texto en un archivo **
> `cat file.xml`
``` xml
<note>
	<to>Tove</to>
	<from>Jani</from>
	<heading>Reminder</heading>
	<body>Don't forget me this weekend!</body>
</note>
```
> `cat file.xml | sed 's/note/elements' | sed 's/to/Author/g' | sed 's/from/Subject/g' | sed 's/heading/Content/g'`
``` xml
<elements>
	<Author>Tove</Author>
	<Subject>Jani</Subject>
	<Content>Reminder</Content>
	<body>Don't forget me this weekend!</body>
</elements>
```

** Quitar una línea con grep **
> `cat file.xml | grep -v "body"`

** Convertir de hexadecimal a decimal con consola de python **
0x0016 --> 22
0x1388 --> 5000

** Depurar interfaces de red del archivo /proc/net/fib_trie a través de un LFI **
`curl -s "http://localhost/example.php?file=/proc/net/fib_trie" | grep -B 1 -i "host local" | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}'`

** Enumeración de puertos internos del fichero /proc/net/tcp **
`curl -s "http://localhost/example.php?file=/proc/net/tcp" | awk "{print $2}" | grep -v "local_address" | awk "{print $2}" FS=":" | sort -u`

** Contar número de líneas de un fichero **
> `cat diccionario.txt | wc -l`

