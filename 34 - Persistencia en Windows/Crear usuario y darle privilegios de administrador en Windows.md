** Creamos un usuario y lo adjuntamos al grupo Administrators**
Se debe ser usuario Administrator para poder realizar estas acciones.
>Crear usuario
>`net user s4vitar s4vitar123! /add`
>Crear un usuario en el dominio
>`net user s4vitar s4vitar123! /add /domain`
>Cargar el usuario en grupo administrador
>`net localgroup Administrators s4vitar /add`
>Ver el grupo Administrators
>`net localgroup Administrators`
>Cargar el usuario en el grupo "Domain Admins" del dominio
>`net group "Domain Admins" s4vitar /add`
>



