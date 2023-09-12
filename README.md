# lab-cacti
Laboratorio para el Seminario de Aplicación de Redes 2023

## 1. Actualizar el servidor Ubuntu 22.04
```
sudo apt update && sudo apt upgrade -y
```

## 2. Instalamos las dependencias necesarias 
```
sudo apt-get install snmp php-snmp rrdtool librrds-perl unzip curl git gnupg2 php-intl -y
```
Iniciar y activar el Apache Web server:
```
sudo systemctl enable --now apache2
```

## 3. Instalción del Servidor LAMP (Linux, Apache, MariaDB, PHP y otras exensiones)
Para almacenar datos utilizamos MySQL/MariaDB, mientras que la interfaz de usuario web de Cacti está basada en PHP, por lo que necesitamos instalar este lenguaje de programación a nuestro sistema junto con algunas extensiones requeridas por Cacti para funcionar correctamente, asi como Apache para lograr acceder a nuestro servicio.

```
sudo apt-get install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-ldap php-mbstring php-gd php-gmp -y
```

## 4. Modificación de las configuraciones por defecto
```
sudo nano /etc/php/8.1/apache2/php.ini
```
Ajustamos el valor de ```date.timezone```, eliminar el ";" del inicio de la línea.
```
date.timezone = America/Guatemala
```
> [Zonaa horariaa](https://www.php.net/manual/en/timezones.php)

### Configuración de la memoria y tiempo de ejecución:
Cambiamos el tiempo de ejecución de ```max_execution_time``` de  30 a 60.
```
max_execution_time = 60
```
> Para hacer más fácil la busqueda se puede utilizar el comando Ctrl+W + ```max_execution_time```.

Cambiamos el límite de ```memory_limit``` de -1 a 512.
```
memory_limit = 512M
```
> Para hacer más fácil la busqueda se puede utilizar el comando Ctrl+W + ```memory_limit```.

Guardamos y salimos del archivo.

### 4.1 Realizaremos el mismo ajuste en el archivo de PHP CLI ```php.ini```.

```
memory_limit = 512M
max_execution_time = 60
date.timezone = America/Guatemala
```

## 5. Creación de la base de datos para Cacti
Editaremos el archivo de configuración predeterminado de MariaDB por lo cual se debe de modificar algunas configuraciones predeterminadas:
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Líneas a modificar o agregar, esto debe de ser en el apartado [mysqld]
```
collation-server = utf8mb4_unicode_ci
max_heap_table_size = 512M
tmp_table_size = 64M
join_buffer_size = 4M
sort_buffer_size = 4M
innodb_file_format = Barracuda
innodb_large_prefix = 1
innodb_buffer_pool_size = 1000M
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
innodb_io_capacity = 5000
innodb_io_capacity_max = 10000
innodb_doublewrite = OFF
```
Adicionalmente debemos comentar la siguiente línea, esto debido al cambio de confidicación que solicita CACTI:
```
#collation-server      = utf8mb4_general_ci
```
Guardamos y salimos del archivo.
Reiniciamos los MariaDB
```
sudo systemctl restart mariadb
```

### 5.1 Creación de la base de datos
Iniciamos el shell de MariaDB
```
mysql
```
> Notar que ahora estaremos en: MariaDB [(none)]>

Creamos la base de datos y el usuario para CACTI con el siguiente comando:
```
create database cactidb;
```
```
GRANT ALL ON cactidb.* TO cactiuser@localhost IDENTIFIED BY 'password';
```
> En este paso podemos cambiar el usuario y la contraseña del usuario que usara Cacti para conectarse a la base de datos.
```
flush privileges;
```
```
exit;
```
### 5.2 Importación de la información de la zona horaria a la base de datos
```
mysql mysql < /usr/share/mysql/mysql_test_data_timezone.sql
```
### 5.3 Asignamos los privilegios a la zona horaria
```
GRANT SELECT ON mysql.time_zone_name TO cactiuser@localhost;
```
```
flush privileges;
```
```
exit;
```

## 6. Configuración de CACTI
### 6.1 Obtención de los recursos
Descargamos la última versión de CACTI 
```
wget https://www.cacti.net/downloads/cacti-latest.tar.gz
```
Extraemos el archivo descargado
```
tar -zxvf cacti-latest.tar.gz
```
Movemos el directorio descomprimido al directorio raiz de Apache:
```
sudo mv cacti-1* /var/www/html/cacti
```

Utilizaremos la configuración de CACTI SQL para completar la base de datos creada previamente.
```
mysql cactidb < /var/www/html/cacti/cacti.sql
```

### 6.2 Ajustes de Cacti
Editamos el archivo de Cacti config.php y para definir la configuración la base de datos
```
nano /var/www/html/cacti/include/config.php
```
```
$database_type     = 'mysql';
$database_default  = 'cactidb';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = 'password';
$database_port     = '3306';
```
> En este punto si se realizó algun cambio en el usuario o contraseña se debe de realizar la modificación.

Guardamos y salimos del archivo.

Creamos el siguente archivo para poder almacenar logs
```
sudo touch /var/www/html/cacti/log/cacti.log
```

### 6.3 Modificamos el dueño y los permisos del directorio Cacti
```
chown -R www-data:www-data /var/www/html/cacti/
```
```
chmod -R 775 /var/www/html/cacti/
```

### 6.4 Creación del Cron para la ejecución de CACTI
```
sudo nano /etc/cron.d/cacti
```
Agregamos la configuración para que se ejecute cada 5 minutos
```
*/5 * * * * www-data php /var/www/html/cacti/poller.php > /dev/null 2>&1
```
Guardamos y salimos del archivo.

## 7. Creaciónde del virtual host para CACTI
Creamos el archivo:
```
nano /etc/apache2/sites-available/cacti.conf
```

Agregar las siguientes líneas al archivo:
```
Alias /cacti /var/www/html/cacti

  <Directory /var/www/html/cacti>
      Options +FollowSymLinks
      AllowOverride None
      <IfVersion >= 2.3>
      Require all granted
      </IfVersion>
      <IfVersion < 2.3>
      Order Allow,Deny
      Allow from all
      </IfVersion>

   AddType application/x-httpd-php .php

<IfModule mod_php.c>
      php_flag magic_quotes_gpc Off
      php_flag short_open_tag On
      php_flag register_globals Off
      php_flag register_argc_argv On
      php_flag track_vars On
      # this setting is necessary for some locales
      php_value mbstring.func_overload 0
      php_value include_path .
 </IfModule>

  DirectoryIndex index.php
</Directory>
```
Guardamos y cerramos el archivo.

Habilitamos el sitio
```
a2ensite cacti
```

Reiniciamos los servicios  modificados
```
sudo systemctl restart apache2 mariadb
```

## 8. Acceso a CACTI
```
http://your-server-IP-address/cacti/
```
Se solicitara un usuario/contraseña:
1. admin
2. admin

Se solicitará el cambio de contraseña, aqui se solicitara que poseean algunas caracteristicas. (ej. Sar2023!)

## 9. Asistente de instalación web de CACTI

9.1 Se solicitara aceptar la licencia, asi como podemos seleccionar el tema y el idioma.

9.2 Se mostrará un resumen de todos los prerequisitos que solicita CACTI.
> Errorres comunes
> 1. 
