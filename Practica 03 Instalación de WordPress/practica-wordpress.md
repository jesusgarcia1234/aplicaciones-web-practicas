
# Práctica: Instalación de WordPress en Ubuntu 24.04

**Alumno:** Jesús García Ibáñez  
**Curso:** 2.º SMX  
**Asignatura:** Aplicaciones Web  
**Fecha:** 28-10-2026

## Índice

1. [Conexión SSH](#apartado-1-conexión-ssh)
2. [Configuración de Apache](#apartado-2-configuración-de-apache)
3. [Base de datos](#apartado-3-base-de-datos)
4. [Instalación de WordPress](#apartado-4-instalación-de-wordpress)
5. [Instalación desde el navegador](#apartado-5-instalación-desde-el-navegador)
6. [Comprobación de WordPress](#apartado-6-comprobación-de-wordpress)
7. [Configuración de la IP](#apartado-7-configuración-de-la-ip)
8. [Ajustes de Apache y PHP](#apartado-8-ajustes-de-apache-y-php)
9. [Error de MariaDB](#apartado-9-error-de-mariadb)
10. [Comprobación final](#apartado-10-comprobación-final)
11. [Conclusiones](#conclusiones)

---

## Apartado 1. Conexión SSH

Primero comprobamos la IP del servidor:

```bash id="wdd9dy"
hostname -I
```

Después nos conectamos desde nuestro ordenador mediante SSH:

```bash id="5w23lr"
ssh ls_jgarcia@192.168.56.23
```

Comprobamos que Apache y MariaDB funcionan y consultamos la versión de PHP:

```bash id="xgq08a"
systemctl is-active apache2 mariadb
php -v
```

Los dos servicios deben mostrar `active` y PHP debe ser la versión 8.3.

![Comprobación del servidor](./Imagenes/Captura%20001%20apartado%201.png)

**❓ ¿Qué dos puertas utilizamos?**

SSH (puerto 22) permite administrar el servidor desde la terminal. HTTP (puerto 80) permite acceder a WordPress desde el navegador.

[Volver al índice](#índice)

## Apartado 2. Configuración de Apache

Desactivamos los sitios anteriores y activamos el predeterminado:

```bash id="lu05jb"
sudo a2dissite sitio1.conf sitio2.conf
sudo a2ensite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

`configtest` debe mostrar `Syntax OK`.

Comprobamos que solo tenemos un sitio activo y que utiliza `/var/www/html`:

```bash id="ljdcbc"
ls /etc/apache2/sites-enabled/
grep DocumentRoot /etc/apache2/sites-enabled/000-default.conf
```

![Configuración de Apache](./Imagenes/Captura%20002%20apartado%202.png)

Abrimos `http://192.168.56.23` desde el navegador para comprobar que Apache responde.

![Página de Apache](./Imagenes/Captura%20003%20apartado%202.png)

**❓ ¿Qué diferencia hay entre desactivar y borrar un sitio? ¿Por qué ahora vemos la página predeterminada?**

`a2dissite` desactiva el sitio sin borrar sus archivos. Ahora vemos la página predeterminada porque hemos activado `000-default.conf` en lugar de `sitio1`.

[Volver al índice](#índice)

## Apartado 3. Base de datos

Entramos en MariaDB:

```bash id="tfrf15"
sudo mariadb
```

Creamos la base de datos, el usuario y sus permisos:

```sql id="y09u4n"
CREATE DATABASE wordpress;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY '********';
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
exit;
```

Después comprobamos que podemos acceder con el nuevo usuario:

```bash id="k292xp"
mariadb -u wp_user -p wordpress
```

Introducimos la contraseña y salimos con `exit;`.

![Base de datos creada](./Imagenes/Captura%20004%20apartado%203.png)

### Tabla de comandos

| Comando | Función | Captura |
|---|---|---|
| `CREATE DATABASE wordpress;` | Crea la base de datos. | ![Crear base](./Imagenes/Imagenes%20tablas/Captura%20001%20tabla%20apartado%203.png) |
| `CREATE USER ...` | Crea el usuario de WordPress. | ![Crear usuario](./Imagenes/Imagenes%20tablas/Captura%20002%20tabla%20apartado%203.png) |
| `GRANT ALL ...` | Asigna permisos sobre `wordpress`. | ![Permisos](./Imagenes/Imagenes%20tablas/Captura%20003%20tabla%20apartado%203.png) |

**❓ ¿Por qué utilizamos `wp_user` y no root?**

Porque es más seguro utilizar un usuario que solo tenga permisos sobre la base de datos de WordPress.

[Volver al índice](#índice)

## Apartado 4. Instalación de WordPress

### 4.1. Descargar WordPress

Instalamos `wget`, entramos en `/tmp` y descargamos WordPress:

```bash id="8g4ox6"
sudo apt install wget -y
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
ls wordpress
```

Tuve un problema con la descarga porque el servidor no conseguía resolver `wordpress.org`. Comprobé la conexión DNS y HTTPS hasta conseguir descargarlo correctamente.

### 4.2. Copiar los archivos

Eliminamos la página anterior de Apache y copiamos WordPress:

```bash id="6vo2qi"
sudo rm /var/www/html/index.html
sudo cp -a /tmp/wordpress/. /var/www/html/
rm -r /tmp/wordpress /tmp/latest.tar.gz
ls /var/www/html
```

`rm` elimina los archivos anteriores o temporales, `cp -a` copia WordPress y `ls` permite comprobar el resultado.

### 4.3. Configurar permisos

```bash id="3vqgis"
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
ls -ld /var/www/html
```

`chown` cambia el propietario a `www-data`, `chmod` establece los permisos y `ls -ld` permite comprobarlos.

![Archivos y permisos](./Imagenes/Captura%20005%20apartado%204.png)

**❓ ¿Por qué utilizamos `www-data`? ¿Qué pasaría si mantuviéramos nuestro usuario?**

Porque Apache utiliza `www-data`. Si no tuviera los permisos necesarios, WordPress podría fallar al subir imágenes o instalar plugins.

[Volver al índice](#índice)

## Apartado 5. Instalación desde el navegador

Abrimos `http://192.168.56.23`, seleccionamos Español y pulsamos **¡Vamos a ello!**

### 5.1. Conexión con MariaDB

Introducimos los datos:

| Campo | Valor |
|---|---|
| Base de datos | `wordpress` |
| Usuario | `wp_user` |
| Contraseña | `********` |
| Servidor | `localhost` |
| Prefijo | `wp_` |

![Datos de la base de datos](./Imagenes/Captura%20006%20apartado%205.1.png)

Pulsamos **Enviar** e **Iniciar la instalación**.

### 5.2. Configurar WordPress

Elegimos el título de la web, nuestro usuario administrador, una contraseña segura y un correo electrónico.

También marcamos la opción de pedir a los buscadores que no indexen el sitio, ya que es una práctica.

![Configuración del sitio](./Imagenes/Captura%20007%20apartado%205.2.png)

Pulsamos **Instalar WordPress**, accedemos con nuestro usuario y comprobamos que aparece el escritorio.

![Escritorio de WordPress](./Imagenes/Captura%20008%20apartado%205.2.png)

**❓ ¿Qué hemos hecho en la terminal y qué en el navegador?**

En la terminal preparamos Apache, MariaDB y los archivos. En el navegador terminamos la instalación. El servidor no necesita escritorio porque la interfaz se muestra en nuestro ordenador.

[Volver al índice](#índice)

## Apartado 6. Comprobación de WordPress

### 6.1. Web pública y administración

Comprobamos que funcionan estas direcciones:

- Web: `http://192.168.56.23`
- Administración: `http://192.168.56.23/wp-admin`

### 6.2. Publicar una entrada

Desde **Entradas → Añadir nueva** creamos una entrada de prueba, la publicamos y comprobamos que aparece en la web.

![Entrada publicada](./Imagenes/Captura%20009%20apartado%206.3.png)

### 6.3. Comprobar las tablas

Ejecutamos:

```bash id="pkanbq"
sudo mariadb wordpress -e "SHOW TABLES;"
```

Este comando muestra las tablas que WordPress ha creado en MariaDB.

![Tablas de WordPress](./Imagenes/Captura%20010%20apartado%206.4.png)

Las principales son `wp_posts` (entradas), `wp_users` (usuarios) y `wp_options` (configuración).

Los archivos están en `/var/www/html`, mientras que los temas, plugins y medios están en `wp-content`.

**❓ ¿Dónde se guardan las entradas?**

En MariaDB, concretamente en `wp_posts`. Hemos comprobado que existe esta tabla utilizando `SHOW TABLES;`.

[Volver al índice](#índice)

## Apartado 7. Configuración de la IP

WordPress guarda la dirección del sitio. Para evitar problemas si cambia la IP de nuestra máquina virtual, modificamos:

```bash id="8h95mx"
sudo nano /var/www/html/wp-config.php
```

Antes de la línea `That's all, stop editing!` añadimos:

```php id="dk88ln"
define( 'WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] );
define( 'WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] );
```

Guardamos y comprobamos que no hay errores:

```bash id="1bnsgb"
php -l /var/www/html/wp-config.php
```

Debe aparecer `No syntax errors detected`.

![Comprobación de la configuración](./Imagenes/Captura%20011%20apartado%207.3.png)

[Volver al índice](#índice)

## Apartado 8. Ajustes de Apache y PHP

### 8.1. Enlaces permanentes

Primero creamos la configuración de Apache:

```bash id="53ihzr"
sudo nano /etc/apache2/conf-available/wordpress.conf
```

Añadimos:

```apache id="xs3136"
<Directory /var/www/html>
    AllowOverride All
</Directory>
```

Después activamos los cambios:

```bash id="hrnri2"
sudo a2enmod rewrite
sudo a2enconf wordpress
sudo apache2ctl configtest
sudo systemctl restart apache2
```

| Comando | Función |
|---|---|
| `a2enmod rewrite` | Activa la reescritura de direcciones. |
| `a2enconf wordpress` | Activa nuestra configuración. |
| `apache2ctl configtest` | Comprueba la configuración. |
| `systemctl restart apache2` | Reinicia Apache. |

En WordPress entramos en **Ajustes → Enlaces permanentes**, seleccionamos **Nombre de la entrada** y guardamos.

Comprobamos que la dirección de nuestra entrada contiene su nombre.

```bash id="owdg1e"
ls -la /var/www/html/.htaccess
```

Este comando comprueba que existe `.htaccess`.

### 8.2. Aumentar la subida a 64 MB

Ejecutamos:

```bash id="lf5mje"
printf 'upload_max_filesize = 64M\npost_max_size = 64M\n' | sudo tee /etc/php/8.3/apache2/conf.d/99-wordpress.ini
sudo systemctl restart apache2
```

El primer comando configura el límite de subida y el segundo reinicia Apache.

En **Medios → Añadir nuevo** comprobamos que WordPress permite subir archivos de hasta 64 MB.

### Capturas del apartado

![Configuración de Apache](./Imagenes/Captura%20012%20apartado%208.2.png)

![Enlaces permanentes](./Imagenes/Captura%20013%20apartado%208.2.png)

![Límite de subida](./Imagenes/Captura%20014%20apartado%208.2.png)

**❓ ¿Por qué aparece un error 404 si Apache no permite `.htaccess`?**

Porque WordPress necesita las reglas de `.htaccess` para interpretar las direcciones bonitas. Si Apache las ignora, algunas direcciones no funcionan.

[Volver al índice](#índice)

## Apartado 9. Error de MariaDB

### 9.1. Detener el servicio

Ejecutamos:

```bash id="skg0zv"
sudo systemctl stop mariadb
```

Recargamos WordPress y comprobamos que aparece un error de conexión con la base de datos.

![Error de conexión](./Imagenes/Captura%20015%20apartado%209.1.png)

### 9.2. Recuperar el servicio

Volvemos a iniciar MariaDB:

```bash id="qw7ymx"
sudo systemctl start mariadb
systemctl is-active mariadb
```

El resultado debe ser `active`. Después recargamos la web y comprobamos que vuelve a funcionar.

![WordPress recuperado](./Imagenes/Captura%20016%20apartado%209.2.png)

**❓ ¿Qué comprobaríamos si WordPress no conecta con MariaDB?**

Primero comprobaríamos si MariaDB está activo. Si sigue fallando, revisaríamos el usuario y la contraseña de conexión.

[Volver al índice](#índice)

## Apartado 10. Comprobación final

Por último, comprobamos los servicios, PHP, el sitio activo y las tablas:

```bash id="7lb0wu"
systemctl is-active apache2 mariadb
php -v
ls /etc/apache2/sites-enabled/
sudo mariadb wordpress -e "SHOW TABLES;"
```

El resultado esperado es que ambos servicios estén activos, PHP sea 8.3, solo esté habilitado `000-default.conf` y aparezcan las tablas de WordPress.

| Pieza | Programa | Estado esperado |
|---|---|---|
| Linux | Ubuntu 24.04 | Instalado |
| Apache | Apache2 | Funcionando |
| MariaDB | MariaDB 10.11 | Funcionando |
| PHP | PHP 8.3 | Funcionando |
| WordPress | `/var/www/html` | Accesible desde el navegador |

Guardamos en privado el usuario y la contraseña de administración. El panel está en `http://192.168.56.23/wp-admin`.

También se recomienda crear una instantánea de VirtualBox para conservar la configuración.

[Volver al índice](#índice)

## Conclusiones

En esta práctica hemos instalado WordPress sobre nuestro servidor Ubuntu, utilizando Apache, MariaDB y PHP. Hemos aprendido a preparar la base de datos, configurar los permisos, instalar WordPress y publicar una entrada.

El principal problema que tuve fue al descargar WordPress, porque inicialmente el servidor no resolvía el dominio. Después de comprobar la red y el DNS conseguí solucionarlo.

También hemos configurado los enlaces permanentes, aumentado el límite de subida y comprobado qué ocurre cuando MariaDB deja de funcionar.

Finalmente dejamos WordPress preparado para las próximas prácticas.

[Volver al índice](#índice)
