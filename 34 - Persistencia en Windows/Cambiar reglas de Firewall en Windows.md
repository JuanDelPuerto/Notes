** Configurar reglas de Firewall en Windows para abrir puerto con SMB (445) **
>Abrir puerto SMB con el firewall
>`netsh advfirewall firewall add rule name="Samba Port" protocol=TCP dir=in localport=445 action=allow`
>`netsh advfirewall firewall add rule name="Samba Port" protocol=TCP dir=out localport=445 action=allow`
>Retocar políticas de acceso remoto en Windows
>`cmd /c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountFilterPolicy /t REG_DWORD /d 1 /f`

** Habilitar RDP **
>RDP (Remote Desktop Protocol) con reglas de firewall
>`netsh advfirewall firewall add rule name="RDP Port" protocol=TCP dir=in localport=3389 action=allow`
>`netsh advfirewall firewall add rule name="RDP Port" protocol=TCP dir=out localport=3389 action=allow`
>Habilitar RDP a nivel de registro
>`reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f`
>Habilitar usuario para utilizar RDP
>`net localgroup "Remote Desktop Users" s4vitar /add`