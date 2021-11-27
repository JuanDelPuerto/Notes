ipScan.sh
``` bash

#!/bin/bash` 
for ip in $(seq 1 255); do
	timeout 1 bash -c "ping -c 10.10.10.$ip > /dev/null" && echo "10.10.10.$ip - ACTIVE"
done; wait

```