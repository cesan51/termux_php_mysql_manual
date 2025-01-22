# Manual de Instalación Rápida: Termux, PHP y MySQL

## 1. Instalación de Termux desde repositorio oficial o desde F-Droid
Descarga e instala la aplicación Termux desde el repositorio oficial ó desde F-Droid con el siguiente enlace

[Termux](https://f-droid.org/es/packages/com.termux/)

Recomendación: Instalar versión sugerida Versión 0.118.1

Acepta todos los permisos de seguridad.

## 2. Configuración inicial de Termux
Ejecuta los siguientes comandos para actualizar los repositorios e instalar las herramientas necesarias:
```bash
pkg install root-repo
pkg install x11-repo
apt update
apt upgrade
pkg install openssh
pkg install net-tools
pkg install coreutils
```

## 3. Configuración de SSH
Para habilitar el acceso remoto a Termux mediante PuTTY:
```bash
sshd
whoami # Para ver el usuario
ifconfig # Para ver la IP
passwd # Para asignar una contraseña al usuario SSH
pkill sshd # Para finalizar
```

Instalación de PuTTy y conexión de forma remota para manejo sencillo desde equipo

```bash
whoami@ifconfig # Ejemplo u0_564@192.168.1.2
port 8022
```

## 4. Instalación del módulo PHP
Instala el módulo de PHP, incluido apache y gd y verifica su instalación:
```bash
apt install php-apache php-gd
ls -la /data/data/com.termux/files/usr/libexec/apache2 | grep php
```
Debería devolver algo similar a:
```bash
-rwx------ 1 u0_a252 u0_a252 15001832 Jul  5 01:32 libphp.so
```

## 5. Configuración de Apache
Edita el archivo de configuración de Apache:
```bash
cd /data/data/com.termux/files/usr/etc/apache2/
nano httpd.conf
```
Reemplaza las siguientes líneas:
```apache
#LoadModule mpm_prefork_module libexec/apache2/mod_mpm_prefork.so
LoadModule mpm_worker_module libexec/apache2/mod_mpm_worker.so
```
Por estas:
```apache
LoadModule mpm_prefork_module libexec/apache2/mod_mpm_prefork.so
LoadModule php_module /data/data/com.termux/files/usr/libexec/apache2/libphp.so
<FilesMatch \.php$>
  SetHandler application/x-httpd-php
</FilesMatch>
#LoadModule mpm_worker_module libexec/apache2/mod_mpm_worker.so
```

Ajusta la raíz del documento si lo deseas y añade soporte para `index.php`:
```apache
DocumentRoot "/data/data/com.termux/files/usr/share/apache2/default-site/htdocs"
<Directory "/data/data/com.termux/files/usr/share/apache2/default-site/htdocs">

<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>
```

Configura el puerto a `8081`

```apache
Listen 8081
```

## 6. Prueba del servidor Apache
Crea un archivo `info.php` para verificar el funcionamiento:
```bash
cd /data/data/com.termux/files/usr/share/apache2/default-site/htdocs
nano info.php
```
Contenido del archivo:
```php
<?php phpinfo(); ?>
```
Debes verificar que la opción GD este habilitada para el manejo de imagenes.

## 7. Configuración de PHP
Crea y edita el archivo `php.ini`:
```bash
nano /data/data/com.termux/files/usr/etc/php/php.ini
```

Ten en cuenta que si no existe la carpeta php debes crearla y luego crear php.ini
```bash
mkdir php
nano php.ini
```

Contenido sugerido:
```ini
; Ajustes de error
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
display_errors = On
log_errors = On
error_log = "/data/data/com.termux/files/usr/tmp/php_errors.log"

; Configuración de sesiones
session.save_path = "/data/data/com.termux/files/usr/tmp"

; Límite de memoria
memory_limit = 256M

; Ajustes de carga de archivos
file_uploads = On
upload_tmp_dir = "/data/data/com.termux/files/usr/tmp"
upload_max_filesize = 50M
post_max_size = 50M

; Ajustes generales
max_execution_time = 300
max_input_time = 300

; Configuración de extensiones
extension_dir = "/data/data/com.termux/files/usr/lib/php"
extension=gd.so
```
Guardas con CONTROL + O y luego enter.
Cierras con CONTROL + X.

## 8. Instalación y configuración de MySQL (MariaDB)
Instala MariaDB:
```bash
pkg install mariadb
mysqld_safe --datadir=$PREFIX/var/lib/mysql &
```
Accede como root:
```bash
mysql -u root
```
Crea un usuario con privilegios locales y remotos:
```sql
CREATE USER 'usuario'@'localhost' IDENTIFIED BY 'contraseña';
GRANT ALL PRIVILEGES ON *.* TO 'usuario'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;

CREATE USER 'usuario'@'%' IDENTIFIED BY 'contraseña';
GRANT ALL PRIVILEGES ON *.* TO 'usuario'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGIOS;
```
Verifica los privilegios:
```sql
SHOW GRANTS FOR 'usuario'@'%';
```

## 9. Configuración adicional de GD para PHP en PHP 8.4
Instala y configura GD:
```bash
pkg install patchelf
patchelf --add-needed libphp.so $PREFIX/lib/php/gd.so
php -i | grep -i "gd"
rm /data/data/com.termux/files/usr/etc/php/conf.d/gd.ini
```
Debes verificar que la opción GD este habilitada para el manejo de imagenes desde `info.php`.

## 10. Automatización del inicio y detención de servicios
Crea scripts para manejar los servicios:

**start_services.sh**:
```bash
#!/bin/bash
mysqld_safe --datadir=$PREFIX/var/lib/mysql &
httpd &
php-fpm &
```

**stop_services.sh**:
```bash
#!/bin/bash
pkill mysqld
pkill httpd
pkill php-fpm
```

**web.sh**:
```bash
#!/bin/bash
if pgrep mysqld > /dev/null
then
    ./stop_services.sh
else
    ./start_services.sh
fi
```

Asigna permisos de ejecución:
```bash
chmod +x start_services.sh stop_services.sh web.sh
```
Ejecuta el script para iniciar los servicios:
```bash
./web.sh
```

## 11. Subida de software mediante SSH
Para subir archivos desde un equipo externo:
```bash
scp -P 8022 software.zip u0_a249@192.168.1.21:/data/data/com.termux/files/usr/share/apache2/default-site/htdocs
```
Crea la carpeta del software y descomprime el archivo el archivo en ella:
```bash
cd /data/data/com.termux/files/usr/share/apache2/default-site/htdocs
mkdir software
mv software.zip software
cd software
unzip software.zip
```

## 12. Configuración de acceso al aplicativo
Accede al aplicativo desde la URL:
```text
http://192.168.1.2:8081/software
```
Crea carpetas adicionales según sea necesario:
```bash
mkdir /data/data/com.termux/files/home/software/_lib/file/img/tmpSincroniza
```

## 13. Notas finales
- Verifica que los permisos de los archivos y carpetas sean correctos.
- Asegúrate de probar la conexión a la base de datos e insertar la misma desde herramientas externas como MySQL Workbench.

Recuerda que la conxión que usaras es `127.0.0.1`.

Este manual detalla una configuración completa para desarrollar y gestionar proyectos PHP con MySQL en Termux, facilitando su uso tanto en dispositivos móviles como en entornos remotos.

