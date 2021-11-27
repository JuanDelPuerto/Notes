** WFUZZ **
`wfuzz -c --hc=404 -w <wordlist> http://10.10.10.103/FUZZ`
-t 		threads
-L 		redirect

Fuzzeando con extensiones:
`wfuzz -c --hc=404 -w /usr/share/dirbuster/wordlist/directory-list-2.3-medium -w extensions.txt http://10.10.10.98/FUZZ.FUZ2Z`

** Gobuster **
Fuzzing:
`gobuster dir -u http://10.10.10.91:5000/ -w /usr/share/wordlists/directory-list-2.3-medium.txt`
Incluir hilos:
`gobuster dir -u http://10.10.10.93/ -w /usr/share/wordlists/directory-list-2.3-medium.txt -t 25`
Incluir búsqueda de ficheros con una extensión determinada:
`gobuster dir -u http://10.10.10.93/ -w /usr/share/wordlists/directory-list-2.3-medium.txt -x aspx`
