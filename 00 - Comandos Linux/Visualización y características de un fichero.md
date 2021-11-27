`cat message.crypted`		//imposible de leer un fichero encriptado
`string message.crypted`		//lee las cadenas imprimibles del fichero, da igual su formato
`xxd message.crypted`		//parsea el archivo con referencia hexadecimal
`file message.crypted`		//devuelve el tipo de fichero y sus características
`exiftool document.xlsm`	//Leer, escribir y modificar metadatos de un fichero.

** Archivos XLSM (Microsoft Excel 2007) **
> Descargar WPS Office para Linux.
> Instalar: `dkpg <archivo.deb>`
> Ver metadatos con exiftool
> `exiftool document.xlsm | grep -i -E "Creator|author|mime type"`
> Ver macros del archivo con ole-tools: https://github.com/decalage2/oletools
> `olevba3 -a document.xlsm 2>/dev/null | grep -i "connectionstring" | grep -oP '".*?*"' | tr -d '\"' | tr ';' '\n' > database`
> Análisis de todo el documento con Vipermonkey: https://github.com/decalage2/ViperMonkey
> `vmonkey --ioc document.xlsm`	//--ioc --> indicadores de compromiso.
> Herramientas online para analizar y comprobar documento XLSM:
> hybrid-analysis.com
