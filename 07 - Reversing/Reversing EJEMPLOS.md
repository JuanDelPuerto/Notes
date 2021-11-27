** Binario try-harder, máquina BookStore_THM **

`file try-harder`	// Ver qué tipo de fichero es: ELF 64-bit. Binario compilado para Linux
`ltrace try-harder`	// Ver funciones que realiza a alto nivel el binario
`strace try-harder`	// Ver acciones a bajo nivel que está realizando el binario
`strings try-harder` 	// Ver cadenas de caracteres imprimibles que tiene el binario

Transferencia a máquina local
> Creamos un hash en base64 del binario:
> `bae64 -w 0 try-harder; echo`
> Copiamos el hash y lo decodificamos en la máquina atacante:
> `echo "<hash-base64>" | base64 -d > content`
> Comprobamos la integridad de la data del binario:
> `md5sum try-harder`

Inspección del binario con RADARE2:
> Abrimos el binario con radare2:
> `radare2 ./content`
> Funciones de radare2:
> Entrar en modo análisis --> `aaa`
> Ver funciones del binario --> `afl`
> Cargar una función o instrucción en memoria --> `s main` || `s 0x5dcd21f4`
> Mostrar función o instrucción cargada --> `pdf`

Inspección del binario con GHIDRA (ghidra-sre.org)
> Descargar archivo comprimido y descomprimir con "unzip".
> Arrancar Ghidra --> `./ghidraRun`
> Crear un nuevo proyecto, arrastrar el binario a inspeccionar dentro del proyecto y arrastrar el proyecto hasta el botón del dragón para ejecutarlo.
> 