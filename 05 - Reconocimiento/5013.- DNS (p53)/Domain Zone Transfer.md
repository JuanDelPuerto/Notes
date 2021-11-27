** Introducir en /etc/hosts los dominios para una dirección de red **
`10.10.10.83	ctfolympus.htb ns1.ctfolympus.htb ns2.ctfolympus.htb`

** Dig **
Herramienta a nivel de comandos para administrar redes haciendo consultas DNS.
`dig @10.10.10.83 ctfolympus.htb [ns|mx]`
`dig @10.10.10.83 ctfolympus.htb axfr`

** Ejecución de Domain Zone Transfer ** 
`host -t axfr ctfolympus.htb [ns1.ctfolympus.htb|ns2.ctfolympus.htb]`

