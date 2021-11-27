** Docker **
Proyecto de código abierto que automatiza el despliegue de aplicacones dentro de contenedores software, proporcionando una capa adicional de abstracción y automatización de virtualización de aplicaciones en múltiples sistemas operativos.

Docker utiliza características de aislamiento de recursos del kernel Linux, tales como "cgroups" y "namespaces" (espacio de nombres) para permitir que contenedores independientes se ejecuten dentro de una sola instancia de Linux, evitando la sobrecarga de iniciar y mantener máquinas virtuales.

El soporte del kernel Linux para los "namespaces" aísla la vista que tiene una aplicación de su entorno operativo, incluyendo árboles de proceso, red, ID de usuario y sistemas de archivos montados, mientras que los "cgroups" del kernel proporcionan aislamiento de recursos, incluyendo la CPU, la memoria , el bloque de E/S y de la red.

Docker incluye la bibilioteca "libcontainer" como su propia manera de utilizar directamente las capacidades de virtualiación que ofrece el kernel Linux. Además utiliza las interfaces abstraidas de virtualización mediante "libvirt", "LXC" (Linux Containers) y "systemd-nspawn".

Mediante el uso de contenedores, los recursos pueden ser aislados, los servicios restringidos y se otorga a los procesos la cpacidad de tener una visión completamente privada del sistema operativo con su propio identificador de espacio  de proceso, la estructura del sistema de archivos y las interfaces de red.

Docker es una herramienta que permite empaquetar una aplicación y sus dependencias en un contendor virtual que se puede ejecutar en cuarlquier servidor Linux. Esto ayuda a permitir la flexibilidad y la portabilidad. Ya sea en instalaciones físiscas, la nube pública o privada, etc.


** Comandos principales de Docker **
> Instalación --> `apt-get install docker.io -y`
> Matar los procesos propios de la instalación que pueden ocasionar errores en la creación de contenedores posteriores --> `pkill dockerd && pkill docker`
> Arrancar demonio en segundo plano y no visualizar errores --> `docker > /dev/null 2>&! &`
> Hacer este proceso independiente y que no esté relacionado con la ventana en la que lo abrimos --> `disown %`
> Visualizar imágenes --> `docker images`

Archivo Dockerfile: 
![[Pasted image 20210715132908.png]]

Ejemplo de estructura de Dockerfile:
![[Pasted image 20210715133158.png]]

Archivo ejemplo:
```
FROM ubuntu:18.04
RUN apt-get -y update; \
	apt-get -y upgrade; \
	apt-get -y install apt-utils; \
	vim \ 
	htop;
RUN apt-get -y install dstat
CMD ["bash"]
```

Otros conceptos de Dockerfile:
> `MAINTAINER s4vitar <email>`
> `ENV DEBIAN_FRONTEND noninteractive`
> `ADD <file> <path>`	//añadir ficheros
> `EXPOSE <puerto>` 	//puertos en escucha
> `WORKDIR /opt`			// cambio de directorio
> `FROM php:7.0-apache`		//imagen preinstalada
> `ENTRYPOINT service apache2 start & /bin/bash`
> `COPY cmd.php /var/www/html/` 		//copia de máquina a contenedor

Comandos Docker:
> Creamos imagen --> `docker build -t "pruebas:dockerfile"`
> Creamos contenedor --> `docker run -dit --network bridge --name app <imagen>`		//dti (detached interactive mode), network (tipo de red)
> Ver contenedores --> `docker ps`
> Interactuar con el contenedor --> `docker exec -it app [bash]`		//it (interactive mode)
> Apagar el contenedor --> `docker stop app`
> Ver si quedan instancias abiertas de un contenedor --> `docker ps -a`
> Obtener parámetro ID de procesos abiertos --> `docker ps -a -q`
> Borrar de forma recursiva el contenedor --> `docker rm &(docker ps -a -q)`
> Listar imagenes obsoletas --> `docker images --filter="changling=true"`
> Crear reglas de puerto para comunicar puertos entre máquina y contenedor --> `docker run -dit -p80:80 --network bridge --name app <imagen>`
> Ver reglas de puertos --> `docker port <name_container>`