
- [**Amazon Linux**](#amazon-linux)
  - [1.1 **Acceder a AWS Educate**](#11-acceder-a-aws-educate)
  - [1.2 **Crear par de claves**](#12-crear-par-de-claves)
  - [1.3 **Grupos de seguridad**](#13-grupos-de-seguridad)
    - [1.3.1 **Reglas de entrada grupo de seguridad**](#131-reglas-de-entrada-grupo-de-seguridad)
  - [1.4 **Lanzar instancias**](#14-lanzar-instancias)
  - [1.5 **Conexión con instancia**](#15-conexión-con-instancia)
  - [1.6 **Instalación de un servidor web LAMP en Amazon Linux 2**](#16-instalación-de-un-servidor-web-lamp-en-amazon-linux-2)
    - [1.6.1 **Instalación Apache, MariaDB**](#161-instalación-apache-mariadb)
    - [1.6.2 **Establecer permisos de directorio**](#162-establecer-permisos-de-directorio)
    - [(Opcional) 1.6.3 **Probar el servidor LAMP**](#opcional-163-probar-el-servidor-lamp)
    - [1.6.4 **Proteger el servidor de base de datos**](#164-proteger-el-servidor-de-base-de-datos)
  - [1.7 **Cómo servir 2 webs con Apache**](#17-cómo-servir-2-webs-con-apache)
  - [1.8 **Instalar phpMyAdmin**](#18-instalar-phpmyadmin)
  - [1.9 **Autenticación BASIC**](#19-autenticación-basic)
  - [2.0 **Autenticación DIGEST**](#20-autenticación-digest)
    - [2.1 Crear archivo de usuarios y passwords DIGEST](#21-crear-archivo-de-usuarios-y-passwords-digest)
    - [2.2 Editar .conf](#22-editar-conf)

# **Amazon Linux**



## 1.1 **Acceder a AWS Educate**
Entrar [aquí](https://www.awseducate.com/signin/SiteLogin)

* * * 

## 1.2 **Crear par de claves**

Desde la instancia, deberemos crear un par de claves.

pem -> Mac o Linux
ppk -> Linux

Guarda la clave en un lugar seguro.

* * *

## 1.3 **Grupos de seguridad**

En los grupos de seguridad, estableceremos reglas para regular que tipo de tráfico va a permitir el servidor.

Los grupos de seguridad vienen dados por zonas, si creamos uno en Virginia, sólo funcionará en Virgina. Por ello al crearlo le pondremos también el nombre de la zona para identificarlo.

![imagen](img/captura-1.png)


### 1.3.1 **Reglas de entrada grupo de seguridad**

![imagen](img/captura-7.png)

1. `Tipo`: HTTP (puerto 80) ==== `Origen`: Mi ip o cualquier lugar

2. `Tipo`: TCP (puerto a elegir) ==== `Origen`: Mi ip o cualquier lugar

3. `Tipo`: SSH === `Origen`: Mi ip

En caso de no pedirnos puerto 80, no crearemos HTTP y sólo crearemos por TCP los puertos correspondientes.

La regla de entrada SSH debe estar siempre o no podremos conectarnos de forma remota.

Si cambia nuestra IP que tenemos en ese momento, no podremos acceder.
Puede ocurrir al reiniciar el router.

Darle a `Finalizar y Crear grupo de seguridad`

* * *

## 1.4 **Lanzar instancias**
Instancias > Lanzar instancia. Las que ponen `Free tier eligible` son gratuitas.
Seleecionamos la que queremos pulsando `Select`.

En nuestro caso `AMAZON LINUX EC2`.

`Review and Launch` a la máquina que queremos.
Nos llevará a la página de la instancia.
Seleccionamos nuestro grupo de seguridad y en la página de **Review** ya podremos ver el grupo añadido.
![image](img/captura-2.png)

Le damos a `Launch` y nos avisa de la clave que hay que elegir, y que sabemos que si perdemos el archivo de la key no podremos acceder a la máquina.

Si todo ha ido correcto, `nos dirá que la Instancia está siendo ejecutada`.


* * *

## 1.5 **Conexión con instancia**

Nos conectamos mediante el siguiente comando `ssh -i **clave.pem** **usuario**@**dns o ip**`:
~~~
ssh -i pc-agil-centros.pem ec2-user@52.90.41.221
~~~


* * * 

## 1.6 **Instalación de un servidor web LAMP en Amazon Linux 2**
https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html



### 1.6.1 **Instalación Apache, MariaDB**

1. Conectarse a la instancia.
   
2. Para asegurarse de que todos los paquetes de software están actualizados, realice una actualización rápida del software en la instancia. Este proceso puede durar unos minutos, pero es importante realizarlo para asegurarse de que tiene las actualizaciones de seguridad y las correcciones de errores más recientes. La opción -y instala las actualizaciones sin necesidad de confirmación. Si le gustaría examinar las actualizaciones antes de la instalación, puede omitir esta opción. 

~~~
$ sudo yum update -y
~~~

3. Instale los repositorios lamp-mariadb10.2-php7.2 y php7.2 Amazon Linux Extras para obtener las versiones más recientes de los paquetes LAMP MariaDB y PHP de Amazon Linux 2. `Instalación dependencias`.

~~~
$ sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
~~~

Si recibimos el error **amazon-linux-extras: command not found**, es que no estamos utilizando una instancia de Amazon Linux 2. Con el siguiente comando podemos consultar nuestra versión de Amazon Linux:
~~~
$ cat /etc/system-release
~~~


1. Ahora que la instancia está actualizada, puede instalar los paquetes de software PHP, MariaDB y el servidor web Apache. Utilice el comando `yum install` para instalar varios paquetes de software y todas las `dependencias` relacionadas al mismo tiempo. 

~~~
$ sudo yum install -y httpd mariadb-server
~~~

5. Inicie el servidor web Apache.

~~~
$ sudo systemctl start httpd
~~~

6. Utilice el comando `systemctl enable httpd` para configurar el servidor web Apache de forma que se inicie cada vez que arranque el sistema.

~~~
$ sudo systemctl enable httpd

Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.

~~~

7. Con el siguiente comando podemos comprobar si Apache se encuentra habilitado o no, debería de respondernos `enabled`.
~~~
$ sudo systemctl is-enabled httpd
enabled
~~~



### 1.6.2 **Establecer permisos de directorio**

1. Añada el usuario (en este caso, nuestro usuario es ec2-user) al grupo apache.
~~~
$ sudo usermod -a -G apache ec2-user
~~~

2. Cerramos sesión, conectamos de nuevo a la instancia y comprobamos mediante `groups` si pertenecemos al grupo `apache`.
~~~
$ exit

$ ssh -i pc-agil-centros.pem ec2-user@54.164.130**.214**

$ groups
ec2-user adm wheel apache systemd-journal
~~~

3. Cambiar la propiedad de grupo de /var/www y su contenido al grupo apache dónde **ec2-user** es nuestro usuario.
~~~
$ sudo chown -R ec2-user:apache /var/www
~~~

4. Para agregar permisos de escritura de grupo y establecer el ID de grupo en futuros subdirectorios, cambie los permisos del directorio /var/www y sus subdirectorios. 

~~~
$ sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
~~~

5. Para agregar permisos de escritura de grupo, cambie recursivamente los permisos de archivo de /var/www y sus subdirectorios: 
~~~
$ find /var/www -type f -exec sudo chmod 0664 {} \;
~~~

6. Ahora podemos o bien crear un archivo **.php** (punto 1.6.3) para comprobar si funciona o entrar directamente desde el navegador a la ip o dns de nuestra instancia. En caso de aparecernos la pantalla **Test Page** de Apache, todo estaría correcto.

### (Opcional) 1.6.3 **Probar el servidor LAMP**

Si todo está correcto podemos crear un archivo php y acceder a él:
~~~
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
~~~
Poner en el navegador http://nuestra-ip-o-dns/phpinfo.php

En caso de no estar correcto, con el siguiente comando se comprueban si están instalados los paquetes necesarios: 

~~~
$ sudo yum list installed httpd mariadb-server php-mysqlnd
~~~

Eliminar el archivo creado anteriormente `phpinfo.php`:
~~~
$ rm /var/www/html/phpinfo.php
~~~

### 1.6.4 **Proteger el servidor de base de datos**

1. Iniciar servidor MAriaDB
~~~
$ sudo systemctl start mariadb
~~~

2. Ejecute mysql_secure_installation. 

~~~
$ sudo mysql_secure_installation
~~~

3. Nos pedirá contraseña de root, (por defecto está vacía), pulsar `Y` para confirmar, introducimos la contraseña nueva elegida 2 veces. Nos pregunta si queremos borrar usuarios anónimos: `Y`. Deshabilitar login remoto de root: `Y`. Borrar BBDD de test: `Y`. Recargar tablas de privilegios: `Y`. 

En el caso de no querer usar MariaDb, detener:
~~~
$ sudo systemctl stop mariadb
~~~

En caso de querer usarla, con el siguiente comando MariaDB estableceremos por defecto que se inicie automáticamente al arrancar el sistema:
~~~
$ sudo systemctl enable mariadb
~~~

* * * 

## 1.7 **Cómo servir 2 webs con Apache**

Nos dirigimos al directorio `html` donde ubicamos las páginas:

~~~
$ mkdir academia
$ mkdir tienda

$ nano tienda/index.html
$ nano academia/index.html
~~~

Procedemos a configurar apache para que reconozca las webs
~~~
$ cd /etc/httpd/conf.d/
~~~
Aquí dentro todo es de usuario root, para crear archivos necesitamos hacerlo con sudo

~~~
$ sudo nano tienda.conf
~~~ 

Con esto le decimos que creamos un virtualhost que se puede acceder desde cualquier IP con el puerto 80, con DocumentRoot le decimos el directorio de la carpeta donde está ubicada la web, y él ya sen encargará de buscar el index.html, .php, etc..
~~~
<VirtualHost *:80>
    DocumentRoot /var/www/html/tienda
</VirtualHost>
~~~

Reiniciamos apache para que reconozco los cambios, cada vez que cambiemos la configuracion de apache, es RECOMENDABLE introducir el siguiente comando:

~~~
$ sudo apachectl configtest
~~~
En caso de estar correcto nos responde con el mensaje `Syntax OK`, en caso contrario nos mostrará la ubicación del error y no dejará arrancar Apache, reiniciamos:
~~~
$ sudo systemctl restart httpd
~~~

Agregar otra página web en otro puerto, en este caso el 8080
![imagen](img/captura-6.png)

Creamos los pasos anteriores a diferencia del `conf` que le tendremos que indicar que escuche el puerto 8080.
~~~
$ sudo nano academia.conf
~~~ 

~~~
Listen 8080
<VirtualHost *:8080>
  DocumentRoot /var/www/html/academia
</VirtualHost>
~~~

* * * 

## 1.8 **Instalar phpMyAdmin**

1. Procedemos a instalar dependencias:
~~~
$ sudo yum install php-mbstring -y
~~~

2. Reiniciamos Apache y php-fpm:
~~~
$ sudo systemctl restart httpd
$ sudo systemctl restart php-fpm
~~~

3. Entramos al directorio de la página en donde vamos a implementar phpMyAdmin:
~~~
cd /var/www/html/academia
~~~

4. Seleccionar el paquete de phpMyAdmin mas reciente, crear la carpeta phpmyadmin y eliminar el .tar
~~~
$ wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
$ mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
$ rm phpMyAdmin-latest-all-languages.tar.gz
~~~

5. Si MariaDB está funcionando, obviar este paso, en caso contrario:
~~~
$ sudo systemctl start mariadb
~~~

6. Entrar en nuestra direccion ip y si todo ha ido bien, nos aparecerá el login de phpMyAdmin. Cuidado con el nombre de los directorios, son `case sensitive` y deberemos de introducirlos igual en la barra de navegación.

Ir a `http://direccionip/phpMyAdmin`, Usuario: `root` y nuestra `password`.


* * * 

## 1.9 **Autenticación BASIC**

1. Creamos una página dentro de tienda
~~~
$ mkdir /var/www/tienda/admin
$ nano /var/www/tienda/admin/index.html
~~~

2. Creamos una carpeta para guardar las credenciales de usuarios
~~~
$ sudo mkdir /etc/httpd/password
~~~

~~~
$ sudo htpasswd -c /etc/httpd/password/passwords-admin admin

$ sudo htpasswd -c /etc/httpd/password/passwords-admin `nombreusuario`
~~~
-c si no existe lo creará
password-admin es el nombre del archivo que vamos a crear.


3. Ahora nos dirigimos al archivo `.conf` que queremos proteger  en nuestro caso el de tienda y lo editamos y agregamos lo siguiente

~~~
<VirtualHost *:80>
    DocumentRoot /var/www/html/tienda
</VirtualHost>

<Directory "/var/www/html/tienda/admin">
  AuthType Basic
  AuthName "administrador"
  AuthUserFile /etc/httpd/password/passwords-admin
  Require valid-user
</Directory>
~~~

Comprobamos Sintaxis y si recibimos un `Sintaxis OK` reiniciamos apache y comprobamos si el directorio está protegido:
~~~
$ sudo apachectl configtest

$ sudo systemctl restart httpd
~~~

* * *

## 2.0 **Autenticación DIGEST**


### 2.1 Crear archivo de usuarios y passwords DIGEST

1. Crear directorio a proteger.
~~~
$ mkdir /var/www/html/academia/profesorado
$ nano /var/www/html/academia/profesorado/index.html
~~~

2. En caso de no tener creado un directorio donde almacenar las passwords, lo creamos:

~~~
$ sudo mkdir /etc/httpd/password
~~~

3. Activar password digest, para ello debemos pasar como parámetro un `realm` y el `usuario`. El realm deberemos colocarlo exactamente igual en el .`conf` con `AuthName` o no funcionará el login. 

~~~
$ sudo htdigest -c /etc/httpd/password/passwords-digest "ENCHUFES" president
~~~

### 2.2 Editar .conf
Editar el .conf para agregar la identificación DIGEST:
~~~

<VirtualHost *:80>
   DocumentRoot /var/www/diputacion
</VirtualHost>

<Directory "/var/www/diputacion/enchufes">
  AuthType Digest
  AuthName "ENCHUFES"
  AuthUserfile /etc/httpd/password/passwords-digest
  Require valid-user
</Directory>

~~~

Comprobamos sintaxis, y si recibimos un `Syntax OK`, recargamos apache y listo!
~~~
$ sudo apachectl configtest

$ sudo systemctl restart httpd
~~~


