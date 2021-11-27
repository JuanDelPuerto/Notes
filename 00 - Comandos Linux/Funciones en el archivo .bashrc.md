### Archivo principal de la consola bash
** Actualizar archivo .bashrc **
source ˜/.bashrc

** Funciones **
Función mkt, para crear un conjunto de directorios
``` 
function mkt(){
	mkdir {nmap,content,exploits,scripts,report}
}

```


Función para mostrar por pantalla la información más relevante de un archivo grepeable de nmap con los puertos abiertos y copiar estos a la clipboard.
```
function extractPorts(){
	ports = "$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr '' ',')"
	ip_address = "$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1"
	echo -e "\n[*]Extracting information...\n" > extractPorts.tmp
	echo -e "\t[*]IP Address: $ip_address\n" >> extractPorts.tmp
	echo -e "\t[*]Open Ports: $ports\n" >> extractPorts.tmp
	echo $ports | tr -d '\n' | xclip -sel clip
	echo -e "[*]Ports copied to clipboard\n" >> extractPorts.tmp
	cat extractPorts.tmp; rm extractPorts.tmp
}

```

Borrar el histórico de la shell.
`echo '' > ~/.bash.history`


