Descargar en /opt, SecLists de Github.
Buscamos la lista IIS --> `find \-name IIS.fuzz.txt`
Da como resultado: /opt/SecLists/Discovery/Web-Content
Fuzzeamos --> `wfuzz -c --hc=404 -w IIS.fuzz.txt http://10.10.10.103/FUZZ`
