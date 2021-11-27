** Linux Smart Enumeration **
>https://github.com/diego-treitos/linux-smart-enumeration
>`wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh`
>`./lse.sh -l 1`

** Comandos de enumeración **
> Ver los segmentos de red de la máquina:
> `cat /proc/net/fib_trie | grep -B1 "32 host LOCAL`
> Ver el ARP (tabla ARP) sin comandos:
> `cat /proc/net/arp`
> Ver puertos abiertos:
> `cat /proc/net/tcp`
> Ver procesos de las tareas en ejecución del Task Scheduler
> `cat /proc/sched_debug`
> Ver interfaces
> `ip a`
> `hostname -I`
> 

** Enumeración **
Información de procesos actuales corriendo:
> f=full format, a=all processes with tty, u=user oriented format, x=register format
> `ps -faux`

Regular background program processing daemon
`service cron status`
Ver posibles tareas potenciales que se estén ejecutando a intervalos regulares
`cd /etc/cron.d`
Mostrar las tareas cron para el usuario que tenemos
`crontab -l`
Ver información del kernel:
`uname -a`
`lsb_release -a`
Ver capabilities
`getcap -r / 2>/dev/null`

=====================================================================

