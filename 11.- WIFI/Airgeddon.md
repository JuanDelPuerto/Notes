Programa para realizar auditorías WIFI en Linux. https://github.com/v1s1t0r1sh3r3/airgeddon

Script en bash que permite poner la interfaz inalámbrica en modo monitor, sin necesidad de ejecutar otros programas como Airmon-NG para realizar esta acción, el propio Airgeddon se encarga de ello. Otra característica es que incorpora asistentes para capturar el handshake de las redes WPA y WPA2.

Posteriormente pñodremos realizar un crackeo offline de este handshake e intentar sacar la contraseña PSK de la red inalámbrica a través de diccionario de claves o fuerza bruta.

Es necesario tener instalada una serie de programs de los que el script hace uso: 
- iwconfig
- iw
- awk
- airmon-ng
- airodump-ng
- aircrack-ng
- xterm
- wpaclean
- crunch
- aireplay-ng
- mdk3
- hashcat




