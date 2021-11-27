** Inspeccionar certificado SSL **
Buscar información en el certificado SSL
`openssl s_client -connect 10.10.10.228:443`

** Test ssl **
https://github.com/drwetter/testssl.sh
`./testssl https://10.10.10.79`

** Vulnerabilidad Heartbleed **
Heartbleed es un agujero de seguridad de software en la biblioteca de código abierto OpenSSL, solo vulnerable en su versión 1.0.1f, que permite a un atacante leer la memoria de un servidor o un cliente, permitiéndole por ejemplo, conseguir las claves privadas SSL de un servidor.

La RFC 6520 Heartbeat Extension prueba los enlaces de comunicación segura TLS/DTLS al permitir que un ordenador en un extremo de una conexión envíe un mensaje de "solicitud de latido de corazón" ("Heartbeat Request"), que consiste en una carga útil, típicamente una cadena de texto, junto con la longitud de dicha carga útil como un entero de 16-bits. El equipo receptor debe entonces enviar la misma carga exacta de vuelta al remitente.

Las versiones afectadas de OpenSSL asignan un búfer de memoria para el mensaje a devolver basado en el campo de longitud en el mensaje de solicitud, sin tener en cuenta el tamaño real de la carga útil de ese mensaje. Debido a esta falla de revisión de los límites apropiados, el mensaje devuelto consta de la carga útil, posiblemente seguido de cualquier otra cosa que sea que esté asignada en el _buffer_ de memoria.

** Explotación Heartbleed **
https://gist.github.com/eelsivart/10174134
> Múltiple ejecución del script hasta obtener data de procesos importantes.
> `python heartbleed.py -p 443 10.10.10.79`


