# Almacenamiento en Docker

[TOC]

**Los contenedores son efímeros**, es decir, los ficheros, datos y configuraciones que creamos en los contenedores sobreviven a las paradas de los mismos pero, sin embargo, son destruidos si el contenedor es destruido.

Ante la situación anteriormente descrita Docker nos proporciona varias soluciones para persistir los datos de los contenedores:

- Los **volúmenes docker**

- Los **bind mount**

## Teoría

### Volúmenes docker y bind mount

#### Los contenedores son efímeros

**Los contenedores son efímeros**, es decir, los ficheros, datos y configuraciones que creamos en
los contenedores sobreviven a las paradas de los mismos pero, sin embargo, son destruidos si el
contenedor es destruido.

Veamos un ejemplo:

```bash
$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4

$ docker exec my-apache-app bash -c 'echo "<h1>Hola</h1>" >
/usr/local/apache2/htdocs/index.html'
$ curl http://localhost:8080

$ docker rm -f my-apache-app

$ docker run -d --name my-apache-app -p 8080:80 httpd:2.4

$ curl http://localhost:8080
```

<img src="./practica_almacenamiento_docker.assets/image-20240126101005274.png" alt="image-20240126101005274"  />

![image-20240126101417869](./practica_almacenamiento_docker.assets/image-20240126101417869.png)

![image-20240126101743720](./practica_almacenamiento_docker.assets/image-20240126101743720.png)

![image-20240126101815655](./practica_almacenamiento_docker.assets/image-20240126101815655.png)

![image-20240126101910406](./practica_almacenamiento_docker.assets/image-20240126101910406.png)

![image-20240126101950453](./practica_almacenamiento_docker.assets/image-20240126101950453.png)

Vemos como al eliminar el contenedor, la información que habíamos guardado en el fichero `index.html` se pierde, y al crear un nuevo contenedor ese fichero tendrá el contenido original.



#### Los datos en los contenedores

![image-20240131104538846](./practica_almacenamiento_docker.assets/image-20240131104538846.png)

Ante la situación anteriormente descrita Docker nos proporciona varias soluciones para persistir los datos de los contenedores. En este curso nos vamos a centrar en las dos que considero que son más importantes:

- Los volúmenes docker
- Los bind mount



### Volúmenes docker

Si elegimos conseguir la persistencia usando volúmenes estamos haciendo que los datos de los contenedores que nosotros decidamos se almacenen en una parte del sistema de ficheros que es gestionada por docker y a la que, debido a sus permisos, sólo docker tendrá acceso. En linux se guardan en `/var/lib/docker/volumes`. Este tipo de volúmenes se suele usar en los siguiente casos:

- Para compartir datos entre contenedores. Simplemente tendrán que usar el mismo volumen.
- Para copias de seguridad ya sea para que sean usadas posteriormente por otros contenedores o para mover esos volúmenes a otros hosts.
- Cuando quiero almacenar los datos de mi contenedor no localmente sino en un proveedor cloud.



#### Gestionando volúmenes

Algunos comando útiles para trabajar con volúmenes docker:

- docker volumen create: Crea un volumen con el nombre indicado.
- docker volume rm: Elimina el volumen indicado.
- docker volumen prune: Para eliminar los volúmenes que no están siendo usados por ningún contenedor.
- docker volume ls: Nos proporciona una lista de los volúmenes creados y algo de información adicional.
- docker volume inspect: Nos dará una información mucho más detallada de el volumen que hayamos elegido.




#### Asociando almacenamiento a los contenedores: volúmenes Docker

Veamos como puedo usar los volúmenes y los bind mounts en los contenedores. Aunque hay dos formas de asociar el almacenamiento al contenedor nosotros vamos a usar el flag -`-volume o -v` .
Si usamos imágenes de DockerHub, debemos leer la información que cada imagen nos proporciona en su página, ya que esa información suele indicar cómo persistir los datos de esa imagen, ya sea con volúmenes o bind mounts, y cuáles son las carpetas importantes en caso de ser imágenes que contengan ciertos servicios (web, base de datos etc…)



#### Ejemplo usando volúmenes docker

Lo primero que vamos a hacer es crear un volumen docker:

```bash
$ docker volume create miweb
```

![image-20240131105218979](./practica_almacenamiento_docker.assets/image-20240131105218979.png)

A continuación creamos un contenedor con el volumen asociado, usando --mount , y creamos un fichero index.html:

```bash
$ docker run -d --name my-apache-app -v miweb:/usr/local/apache2/htdocs -p 8080:80 httpd:2.4

$ docker exec my-apache-app bash -c 'echo "<h1>Hola</h1>" >
/usr/local/apache2/htdocs/index.html'
$ curl http://localhost:8080

$ docker rm -f my-apache-app
```

### Bind mounts

Si elegimos conseguir la persistencia de los datos de los contenedores usando bind mount, lo que estamos haciendo es “mapear” (montar) una parte de mi sistema de ficheros, de la que yo normalmente tengo el control, con una parte del sistema de ficheros del contenedor. Por lo tanto podemos montar tanto directorios como ficheros. De esta manera conseguimos:

- Compartir ficheros entre el host y los containers.
- Que otras aplicaciones que no sean docker tengan acceso a esos ficheros, ya sean código, ficheros etc…

## Ejercicios

### Volúmenes

1. Crea un volumen docker que se llame miweb.

   ```bash
   $ docker volume create miweb
   ```

   ![image-20240202085747627](./practica_almacenamiento_docker.assets/image-20240202085747627.png)

2. Crea un contenedor desde la imagen php:7.4-apache donde montes en el directorio /`var/www/html` (que sabemos que es el DocumentRoot del servidor que nos ofrece esa imagen) el volumen docker que has creado.

   ```bash
   $ docker run -d -p 8080:80 --name miweb-container -v miweb:/var/www/html php:7.4-apache
   ```

  ![image-20240202090404841](./practica_almacenamiento_docker.assets/image-20240202090404841.png)

3. Utiliza el comando docker cp para copiar un fichero index.html en el directorio `/var/www/html`.

```bash
$ docker cp index.html miweb-container:/var/www/html/
```

![image-20240202093155284](./practica_almacenamiento_docker.assets/image-20240202093155284.png)

![image-20240202093114002](./practica_almacenamiento_docker.assets/image-20240202093114002.png)

4. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero `index.html`.

  ![image-20240202093409042](./practica_almacenamiento_docker.assets/image-20240202093409042.png)

5. Borra el contenedor.

   ```bash
   $ docker rm -f miweb-container
   ```

   ![image-20240202093457640](./practica_almacenamiento_docker.assets/image-20240202093457640.png)

6. Crea un nuevo contenedor y monta el mismo volumen como en el ejercicio anterior.

   ```bash
   $ docker docker run -d -p 8080:80 --name miweb-container2 -v miweb:/var/www/html php:7.4-apache
   ```

   

![image-20240202093643429](./practica_almacenamiento_docker.assets/image-20240202093643429.png)

7. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero index.html . ¿Seguía existiendo ese fichero?

![image-20240202093750682](./practica_almacenamiento_docker.assets/image-20240202093750682.png)


### Bind mount

1. Crea un directorio en tu host y dentro crea un fichero `index.html`. 

   ```shell
   mkdir bindMountFolder 
   cp index.html ./bindMountFolder/index.html 
   cd bindMountFolder
   ```
   ![](./practica_almacenamiento_docker.assets/Ejercicio01_bind.png)

2. Crea un contenedor desde la imagen `php:7.4-apache` donde montes en el directorio `/var/www/html` el directorio que has creado por medio de `bind mount`. 
   
   ```shell
   docker run -d --name apache-container -p 8080:80 -v /home/server_admin/bindMountFolder:/var/www/html php:7.4-apache
   ```

   ![](./practica_almacenamiento_docker.assets/Ejercicio02_bind.png)

3. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero `index.html`. 

   ![](./practica_almacenamiento_docker.assets/Ejercicio03_bind.png)

4. Modifica el contenido del fichero `index.html` en tu host y comprueba que al refrescar la página ofrecida por el contenedor, el contenido ha cambiado. 
   
   ```shell
   nano bindMountFolder/index.html
   ```

   ![](./practica_almacenamiento_docker.assets/Ejercicio04a_bind.png)   
   ![](./practica_almacenamiento_docker.assets/Ejercicio04b_bind.png)

5. Borra el contenedor 

   ```shell
   docker stop apache-container
   docker rm apache-container
   ```

   ![](./practica_almacenamiento_docker.assets/Ejercicio05_bind.png)

6. Crea un nuevo contenedor y monta el mismo directorio como en el ejercicio anterior. 

   ```shell
   docker run -d --name apache-container -p 8080:80 -v /home/server_admin/bindMountFolder:/var/www/html php:7.4-apache
   ```

   ![](./practica_almacenamiento_docker.assets/Ejercicio06_bind.png)


7. Accede al contenedor desde el navegador para ver la información ofrecida por el fichero `index.html`. ¿Se sigue viendo el mismo contenido?

   Se sigue viendo la misma página.

   ![](./practica_almacenamiento_docker.assets/Ejercicio04b_bind.png)

## Redes

Aunque hasta ahora no lo hemos tenido en cuenta, cada vez que creamos un contenedor, esté se conecta a una red virtual y docker hace una configuración del sistema (usando interfaces puente e iptables) para que la máquina tenga una ip interna, tenga acceso al exterior, podamos mapear (DNAT) puertos,…

Cuando instalamos docker tenemos las siguientes redes predefinidas:

- Red de tipo bridge
- Red de tipo host
- Red de tipo none