Habilitar conexiones remotas
>`cmd /c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountFilterPolicy /t REG_DWORD /d 1 /f`

Habilitar RDP a nivel de registro
>`reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f`