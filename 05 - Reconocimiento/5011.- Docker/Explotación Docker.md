Buscar fichero "docker env" || "dockerfile" -- Identifican que estamos en un contenedor.
`ls -a /.dockerenv`

** Vulnerabilidad en el grupo "docker" **
> Existe la posibilidad de crear una montura de la raiz del sistema operativo anfitrión desde dentro de uno de los contendores bajo la ruta /mnt (mount). Tratando de montar una sesión interactiva de ese contenedor. El usuario que lo ejecuta debe estar en el grupo "docker".
> `docker run -v /:/mnt/ -i -t <imagen> bash`
> Este comando nos ejecutará el contenedor sobre la imagen especificada y nos dará una consola bash interactiva. Además sobre la ruta /mnt habrá montado todo el directorio raiz de la máquina que hospeda al contenedor.

