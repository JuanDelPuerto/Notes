** Herramienta aircrack-ng ** 
https://github.com/aircrack-ng/aircrack-ng

Suite de software de seguridad inalámbrica. Consiste en un analizador de paquetes de red, recuperador de contraseñas WEP y WPA/WPA2-PSK, y otro conjunto de herramientas de auditoría inalámbrica.

Entre las herramientas incluidas en la suite encontramos:
| airbase-ng 		| airdriver-ng 		| airolib-ng 			| packetforge-ng |
| airocrack-ng 		| aireplay-ng 		| airserv-ng 			| tkiptun-ng |
| airdecap-ng 		| airmon-ng 		| airtun-ng 			| wesside-ng |
| airdecloak-ng		| airodump-ng 		| easside-ng |

Las herramientas más utilizadas en la auditoría inalámbrica son:
- Airckrack-ng --> Descifra la clave de los vectores de inicio.
- Airodump-ng --> Escanea las redes y captura vectores de inicio.
- Aireplay-ng --> Inyecta tráfico para elevar la captura de vectores de inicio.
- Airmon-ng --> Establece la tarjeta de red inalámbrica en modo monitor, para poder capturar e inyectar vectores.
 
 
 ** Crackear contraseña encriptada **
1. Generar un "hashcat file" (HCCAP):
		`aircrack-ng -J miCaptura filteredCapture`	//genera miCaptura.hccap
		`hccap2john miCaptura.hccap > hash`		//genera un hash de la contraseña encriptada
		`john -wordilist=rockyou.txt hash` //fuerza bruta sobre la contraseña	
		
2. Crackear con aircrack-ng:
		`aircrack-ng -w rockyou.txt filteredCapture`
		

** ¿Qué se puede hacer con la contraseña de una red inalámbrica? **
Si tenemos credenciales de una red wifi, podemos leer en una captura ciertos paquetes que se hayan transmitido vía POST de HTTP, donde se pueda encontrar alguna contraseña encriptada.
`airdecap-ng -e <nombre_de_la_red> -p <contraseña> captured.cap > captured-decrypted.cap`
Visualizamos la nueva captura ya con todo desencriptado:
`tshark -r captured-decrypted.cap 2>/dev/null`


		
