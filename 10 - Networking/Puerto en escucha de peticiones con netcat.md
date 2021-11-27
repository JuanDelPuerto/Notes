** Linux **
`nc -nlvp 443`

** Windows **
`rlwrap nc -nlvp 443`

** Transferencia de ficheros  con netcat **
> En máquina local, puerto en espera de recepción de un fichero:
> `nc -nlvp 4646 > captured.cap`
> En máquina remota, transferencia de archivo mediante netcat:
> `nc 10.10.14.55 4646 < captured.cap`
> Comprobar que los ficheros pesan lo mismo:
> `du -hc captured.cap`
> Validar la integridad de la data a nivel de hashing:
> `md5sum captured.cap`


