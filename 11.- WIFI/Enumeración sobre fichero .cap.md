Observar los datos del fichero --> `pyrit -r captured.cap analyze`

Ver los paquetes de la captura que se transmiten a nivel de red --> `tshark -r captured.cap 2>/dev/null`

Parseos de captura para obtener resultados más precisos: 
> `tshark -r captured.cap -Y "eapol" 2>/dev/null`
> Extraemos información de "EAPOL" y 0x08 - Beacon Frame.
> `tshark -r captured.cap -Y "eapol || wlan.fc.type_subtype==0x08" 2>/dev/null -w filteredCapture`
> Extraemos información de "Probe Response", 0x05.
> `tshark -r captured.cap -Y "eapol || wlan.fc.type_subtype==0x05" 2>/dev/null -w filteredCapture`
> Información sobre una BISSD concreta.
> `tshark -r captured.cap -Y "(eapol || wlan.fc.type_subtype==0x05 || wlan.fc.type_subtype==0x08) && wlan.addr==f4:ec:38:ab:a8:a9" 2>/dev/null -w filteredCapture`
> Visualización de captura filtrada:
> `tshark -r filteredCapture`
> Cambiar el modo de exportación en el filtrado:
> `tshark -R captured.cap -Y "(eapol || wlan.fc.type_subtype==0x05 || wlan.fc.type_subtype==0x08) && wlan.addr==f4:ec:38:ab:a8:a9" 2>/dev/null -2 -w filteredCapture -F pcap`

Uso de aircrack-ng para poder capturar vectores de inicialización sobre fichero cap.
`aircrack-ng captured.cap`


