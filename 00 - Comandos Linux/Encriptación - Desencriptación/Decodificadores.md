** xxd -- Herramienta para la manipulación de binarios y hexadecimales **
>Decodificar
>`echo "texto" | xxd -ps -r; echo`
>Codificar
>`echo "hola que tal" | xxd -ps`

** Decodificación de texto en base64 **
`echo "texto" | base64 -d; echo `

** Archivo cuyo contenido está en base64. Lo decodificamos.**
`cat drupal.txt.enc | tr -d '\n' | base64 -d > message.crypted` 