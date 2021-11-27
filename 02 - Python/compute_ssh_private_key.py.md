``` python

#!/usr/bin/python3

from Crypto.PublicKey import RSA
from pwn import *

f = open("decoder.pub", "r")
key = RSA.importKey(f.read())

log.info("e: %s" % key.e)
log.info("n: %s" % key.n)

e = key.e
n = key.n
p = 	# Incrustar aquí valor p obtenido de la formla n = p * q en factordb.com
q = 	# Incrustar aquí valor q obtenido de la formla n = p * q en factordb.com

log.info("p: %s" % p)
log.info("q: %s" % q)

# Función modular multiplicativa inversa (modinv)
def egcd(a, b):
	if a == 0:
		return (b, 0, 1)
	else:
		g, y, x = egcd(b % a, a)
		return (g, x - (b // a) * y, y)
		
def modinv(a, m):
	g, x, y = egcd(a, m)
	if g != 1:
		raise Exception('modular inverse does not exist!')
	else:
		return x % m

m = n-(p+q-1)
d = modinv(e, m)

log.info("m: %s" % m)
log.info("d: %s" % d)

key = RSA.construct((n, e, d, p, q))
print(key.exportkey())

```