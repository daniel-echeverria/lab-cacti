# lab-cacti
Laboratorio para SAR2023 Lab Cacti

##1. Update your Ubuntu 22.04 or 20.04 Server
```
sudo apt update && sudo apt upgrade
```

##2. Instalar Apache para Cacti
```
sudo apt install apache2
```
Iniciar y activar el Apache Web server:
```
sudo systemctl enable --now apache2
```

##3. Instalción de PHP
Para almacenar datos utilizamos MySQL/MariaDB, mientras que la interfaz de usuario web de Cacti está basada en PHP, por lo que necesitamos instalar este lenguaje de programación instalado en nuestro sistema junto con algunas extensiones requeridas por Cacti para funcionar correctamente

###3.1 Instalando PHP:
```
sudo apt install php php-{mysql,curl,net-socket,gd,intl,pear,imap,memcache,pspell,tidy,xmlrpc,snmp,mbstring,gmp,json,xml,common,ldap}
```
```
sudo apt install libapache2-mod-php
```

###3.2 Configuración de la memoria y tiempo de ejecución:
Editar el archivo php.ini:
```
sudo nano /etc/php/*/apache2/php.ini
```
Cambiamos el límite de ```memory_limit``` de 128 a 512.
```
memory_limit = 512M
```
Para hacer más fácil la busqueda se puede utilizar el comando Ctrl+W + ```memory_limit```.

Cambiamos el tiempo de ejecución de ```max_execution_time``` de  30 a 300.
```
max_execution_time = 300
```
Para hacer más fácil la busqueda se puede utilizar el comando Ctrl+W + ```max_execution_time```.

###3.3 Ajustamos el valor de ```date.timezone```
```
date.timezone = America/Guatemala
```

Guardamos y salimos del archivo.

###3.3.1 Realizaremos el mismo ajuste en el archivo de PHP CLI ```php.ini```.

```
sudo nano /etc/php/*/cli/php.ini
```
Buscamos nuevamente ```date.timezone```
```
date.timezone = America/Guatemala
```
 
##4.Install MariaDB
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

###4.1 Creación de la base de datos MariaDB Database para Cacti
```
sudo mysql -u root -p
```
```
CREATE DATABASE cacti DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci ;
```
```
GRANT ALL PRIVILEGES ON cacti.* TO 'cacti_user'@'localhost' IDENTIFIED BY 'strongpassword';
```
```
GRANT SELECT ON mysql.time_zone_name TO cacti_user@localhost;
```
```
ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
```
FLUSH PRIVILEGES;
```
```
EXIT;
```
