** CertEnroll - Active Directory Certificate Service Share **
Servicio que permite emitir y administrar certificados.
--> Certificate (password-less) based authentication in winrm (Windows Remote Management). El uso de certificados en vez de credenciales a la hora de autenticarse vía WinRM.

** Abusar de WinRM a través del panel de administración de certificados vía web **
Existen webs de administración para la creación de certificados que puedan usarse para la autenticación vía WinRM. 
>Crear CSR
>`openssl req -newkey rsa:2048 -nodes -keyout amanda.key -out amanda.csr`
>Copiamos el contenido de amanda.csr para pegarlo en la consola de administración web de Microsoft Active Directory Certificate Services
>Una vez subido, descargar en base64.

** Obtener consola interactiva abusando de WinRM gracias a certificado CSR **
Descargar winrm shell alamot de github --> https://github.com/Alamot/code-snippets/blob/master/winrm/winrm_shell.rb
Instalar --> gem install winrm (está escrito en ruby)

Modificar script:
> Sustituir los valores de user and password por los de client_cert y client_key que hemos obtenido al crear el CSR.