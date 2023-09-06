# lab-cacti
Laboratorio para el Seminario de Aplicación de Redes 2023

## 1. Update your Ubuntu 22.04 or 20.04 Server
```
sudo apt update && sudo apt upgrade -y
```

## 2. Instalar Apache para Cacti
```
sudo apt install apache2 -y
```
Iniciar y activar el Apache Web server:
```
sudo systemctl enable --now apache2
```

## 3. Instalción de PHP
Para almacenar datos utilizamos MySQL/MariaDB, mientras que la interfaz de usuario web de Cacti está basada en PHP, por lo que necesitamos instalar este lenguaje de programación instalado en nuestro sistema junto con algunas extensiones requeridas por Cacti para funcionar correctamente

### 3.1 Instalando PHP:
```
sudo apt install php php-{mysql,curl,net-socket,gd,intl,pear,imap,memcache,pspell,tidy,xmlrpc,snmp,mbstring,gmp,json,xml,common,ldap} -y
```
```
sudo apt install libapache2-mod-php -y
```

### 3.2 Configuración de la memoria y tiempo de ejecución:
Editar el archivo php.ini:
```
sudo nano /etc/php/*/apache2/php.ini
```
Cambiamos el límite de ```memory_limit``` de 128 a 512.
```
memory_limit = 512M
```
> Para hacer más fácil la busqueda se puede utilizar el comando Ctrl+W + ```memory_limit```.

Cambiamos el tiempo de ejecución de ```max_execution_time``` de  30 a 300.
```
max_execution_time = 300
```
> Para hacer más fácil la busqueda se puede utilizar el comando Ctrl+W + ```max_execution_time```.

### 3.3 Ajustamos el valor de ```date.timezone```, eliminar el ";" del inicio de la línea.
```
date.timezone = America/Guatemala
```
> [Si no estan seguros pueden buscar su zona horaria](https://www.php.net/manual/en/timezones.php)
Guardamos y salimos del archivo.

### 3.3.1 Realizaremos el mismo ajuste en el archivo de PHP CLI ```php.ini```.

```
sudo nano /etc/php/*/cli/php.ini
```
Buscamos nuevamente ```date.timezone```, eliminar el ";" del inicio de la línea.
```
date.timezone = America/Guatemala
```
 
## 4.Install MariaDB
Instalamos MariaDB como base de datos para Cacti.
```
sudo apt install mariadb-server -y
```
Iniciamos y activamos el servidor de base de datos"
```
sudo systemctl enable --now mariadb
```
Validamos el estado actual del servcio:
```
sudo systemctl status mariadb
```

### 4.1 Creación de la base de datos MariaDB Database para Cacti
```
sudo mysql -u root -p
```
```
CREATE DATABASE cacti DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ;
```
```
CREATE USER 'cactiuser'@'localhost' IDENTIFIED BY 'cactiuser';
```
```
GRANT ALL ON cacti.* TO 'cactiuser'@'localhost';
```
```
GRANT SELECT ON mysql.time_zone_name TO 'cactiuser'@'localhost';
```
```
ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
```
FLUSH PRIVILEGES;
```

## 5. Configuración de MariaDb para Cacti
Editamos el siguiente archivo
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Copiamos y pegamos las siguientes configuraciones debajo de la línea **[mariadb]**
```
innodb_file_format=Barracuda
innodb_large_prefix=1
collation-server=utf8mb4_unicode_ci
character-set-server=utf8mb4
innodb_doublewrite=OFF
max_heap_table_size=128M
tmp_table_size=128M
join_buffer_size=128M
innodb_buffer_pool_size=1G
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
innodb_io_capacity=5000
innodb_io_capacity_max=10000
innodb_buffer_pool_instances=9
```
Volveremos comentario (#) algunas líneas que ya existen en el archivo actual:
```
#character-set-server = utf8mb4
#collation-server = utf8mb4_general_ci
```

## 6. Instalación de SNMP y otras herramientas para CACTI
```
sudo apt install snmp snmpd rrdtool -y
```
## 7. Configuración de CACTI

### 7.1 Instalación de git para clonar el repositorio de CACTI

```
sudo apt install git
```
Copiamos el repositorio
```
git clone -b 1.2.x https://github.com/Cacti/cacti.git
```

Debemos mover el repositorio al directorio WEB:
```
sudo mv cacti /var/www/html
```
Utilizaremos la configuración de CACTI SQL para completar la base de datos creada previamente.
```
sudo mysql -u root cacti < /var/www/html/cacti/cacti.sql

```
### 7.2 Creación de la configuración de PHP para CACTI:
```
cd /var/www/html/cacti/include
```
```
cp config.php.dist config.php
```
Editamos el archivo de configuración
```
sudo nano config.php
```
En este punto debemos de cambiar los siguientes datos
1. Database name
2. Username
3. Password

Guardamos y cerramos el archivo.

### 7.3 Agregamos permisos al usuario de Apache para acceder al directorio de CACTI.
```
sudo chown -R www-data:www-data /var/www/html/cacti
 ```

8. Creación del servicio de CACTI SYSTEMD
Para ejecutar el servicio  de Cacti en segundo plano, se creara un servicio del sistema de CACTI utilizando los siguientes comandos.
```
sudo nano /etc/systemd/system/cactid.service
```
Agregar las siguientes líneas al archivo:
```
[Unit]
Description=Cacti Daemon Main Poller Service
After=network.target

[Service]
Type=forking
User=www-data
Group=www-data
EnvironmentFile=/etc/default/cactid
ExecStart=/var/www/html/cacti/cactid.php
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Guardamos y cerramos el archivo.
Creamos el siguiente archivo
```
sudo touch /etc/default/cactid
```

Iniciar y activar el servicio de CACTI:
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable cactid
```
```
sudo systemctl restart cactid
```
Validamos el estado actual del servicio:
```
sudo systemctl status cactid
```
Cacti monitoring service Ubuntu 22.04 or 20.04

Reiniciamos el servicio MARIADB
```
sudo systemctl restart apache2 mariadb
```

## 9. Acceso a CACTI
```
http://your-server-IP-address/cacti/
```

Se solicitara un usuario el cual sera el creado previamente:
1. admin
2. admin

## 10. Inicialización de CACTI 

10.1 Se solicitará el cambio de contraseña, aqui se solicitara que poseean algunas caracteristicas. (Test1234!)

10.2 Se solicitara aceptar la licencia de CACTI, asi como podemos seleccionar el tema y el idioma.

10.3 
