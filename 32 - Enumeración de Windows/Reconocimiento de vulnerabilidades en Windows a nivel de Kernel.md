** Windows Exploit Suggester,  wesng github. **
> Descargar en carpeta /opt
> `git clone https://github.com/bitsadmin/wesng.git`
> Obtener características de la máquina Windows objetivo
> `systeminfo`
> Copiar todo el resultado y guardarlo en la máquina local
> `nano systeminfo.txt`
> Ejecutar Windows Exploit Suggester en máquina local
> `python wes.py systeminfo.txt -i "Elevation of Privileges"`

** Windows-exploit-suggester **
> Descargar en carpeta /opt
> `git clone https://github.com/AonCyberLabs/Windows-Exploit-Suggester.git`
> Obtener características de la máquina Windows objetivo
> `systeminfo`
> Copiar todo el resultado y guardarlo en la máquina local
> `nano systeminfo.txt`
> Ejecutar Windows Exploit Suggester en máquina local
> `python windows-exploit-suggester --update`
> `python windows-exploit-suggester --database 2019-11-19-mssb.xls --systeminfo systeminfo.txt`