# Práctica: Instalación y configuración de Apache en Ubuntu 24.04

**Alumno:** Jesús García Ibáñez  
**Curso:** 2.º SMX  
**Fecha:** 16-10-26

## Índice

1. [Preparación del sistema](#apartado-1-preparación-del-sistema)
2. [Instalación de Apache](#apartado-2-instalación-de-apache)
3. [Comprobación del funcionamiento](#apartado-3-comprobación-del-funcionamiento)
4. [Comandos principales de administración](#apartado-4-comandos-principales-de-administración)
5. [Ficheros y directorios importantes](#apartado-5-ficheros-y-directorios-importantes)
6. [Modificaciones típicas del servicio](#apartado-6-modificaciones-típicas-del-servicio)
7. [Módulos de Apache](#apartado-7-módulos-de-apache)
8. [Varios sitios en un mismo servidor (Virtual Hosts)](#apartado-8-varios-sitios-en-un-mismo-servidor-virtual-hosts)
9. [Permisos y propietarios](#apartado-9-permisos-y-propietarios)
10. [Conclusiones](#conclusiones)
---

<br>

## Apartado 1. Preparación del sistema

Primero actualizamos la lista de paquetes y después instalamos las actualizaciones disponibles:
```bash
sudo apt update
sudo apt upgrade -y
```
`update` actualiza la información de los repositorios y `upgrade` actualiza los paquetes instalados. El `-y` acepta automáticamente las confirmaciones.

A continuación comprobamos la versión de Ubuntu:
```bash
lsb_release -a
```
En mi caso aparece **Ubuntu 24.04.5 LTS**.

![Versión de Ubuntu](./Imagenes/Captura%20001%20aplicaciones.png)
## Apartado 2. Instalación de Apache

Una vez actualizado Ubuntu, instalamos Apache, que es el servidor encargado de responder a las peticiones de las páginas web:
```bash
sudo apt install apache2 -y
```
Después consultamos la versión instalada:
```bash
apache2 -v
```
También revisamos los paquetes de los que depende Apache:
```bash
apt-cache depends apache2
```
![Dependencias de Apache](./Imagenes/Captura%20002%20aplicaciones.png)

**❓¿Qué paquetes adicionales se instalaron?**

Para verlos usamos este comando:
```bash
grep -A 20 "Commandline: apt install apache2" /var/log/apt/history.log
```
Y a mi me aparece `apache2-bin`, `apache2-data`, `apache2-utils`, `ssl-cert`, `libapr1t64`, `libaprutil1t64`, `libaprutil1-ldap`, `libaprutil1-dbd-sqlite3` y `liblua5.4-0`. Son archivos, librerías y herramientas instalados junto con Apache.
## Apartado 3. Comprobación del funcionamiento

### 3.1. Estado del servicio

Primero comprobamos si Apache estaba encendido:
```bash
sudo systemctl status apache2
```
Si aparece `active (running)`, el servicio está funcionando.

### 3.2. Puertos en escucha

Después miramos el puerto que utiliza Apache:
```bash
sudo ss -tulpn | grep apache2
```
En mi caso escucha en el puerto 80, que es el habitual de HTTP.

### 3.3. Prueba desde terminal y navegador

Para comprobar la respuesta del servidor ejecutamos:
```bash
curl -I http://localhost
```
`localhost` es el propio equipo y `-I` muestra solo las cabeceras HTTP.

![Puerto 80 y respuesta 200 OK](./Imagenes/Captura%20003.1%20aplicaciones.png)

Luego entramos desde el navegador a la IP del servidor y apareció la página predeterminada de Apache.

![Página predeterminada de Apache](./Imagenes/Captura%20003%20aplicaciones.png)

### 3.4. Firewall

Comprobamos el estado del Firewall:
```bash
sudo ufw status
```
En mi caso estaba desactivado. El siguiente comando permite las conexiones del perfil Apache, aunque no activa el firewall por sí solo:
```bash
sudo ufw allow 'Apache'
```
**❓¿Qué diferencia hay entre los perfiles Apache, Apache Full y Apache Secure?**

| Perfil | Puertos | Función |
|---|---|---|
| Apache | 80 | Permite HTTP. |
| Apache Full | 80 y 443 | Permite HTTP y HTTPS. |
| Apache Secure | 443 | Permite HTTPS. |
## Apartado 4. Comandos principales de administración

| Comando | Función | Captura |
|---|---|---|
| `sudo systemctl start apache2` | Inicia el servicio. | ![Captura de sudo systemctl start apache2 ](./Imagenes/Tabla%201/Captura%20001%20tabla%201aplicaciones%20.png) |
| `sudo systemctl stop apache2` | Detiene el servicio. | ![Captura de sudo systemctl stop apache2 ](./Imagenes/Tabla%201/Captura%20002%20tabla%201%20aplicaciones.png) |
| `sudo systemctl restart apache2` | Reinicia Apache y puede cortar conexiones. | ![Captura de sudo systemctl restart apache2 ](./Imagenes/Tabla%201/Captura%20003%20tabla%201%20aplicaciones.png) |
| `sudo systemctl reload apache2` | Recarga la configuración sin reiniciar todo el servicio. Conviene usarlo al aplicar cambios. | ![Captura de sudo systemctl reload apache2 ](./Imagenes/Tabla%201/Captura%20004%20tabla%201%20aplicaciones.png) |
| `sudo systemctl enable apache2` | Activa el arranque automático de Apache. | ![Captura de sudo systemctl enable apache2 ](./Imagenes/Tabla%201/Captura%20006%20tabla%201%20aplicaciones.png) |
| `sudo systemctl disable apache2` | Desactiva el arranque automático. | ![Captura de sudo systemctl disable apache2 ](./Imagenes/Tabla%201/Captura%20007%20tabla%201%20aplicaciones.png) |
| `apache2ctl configtest` | Comprueba la sintaxis de configuración (`Syntax OK`). | ![Captura de apache2ctl configtest ](./Imagenes/Tabla%201/Captura%20008%20tabla%201%20aplicaciones.png) |
| `apache2ctl -S` | Muestra los Virtual Hosts configurados. | ![Captura de apache2ctl -S ](./Imagenes/Tabla%201/Captura%20009%20tabla%201%20aplicaciones.png) |
| `apache2ctl -M` | Muestra los módulos cargados. | ![Captura de apache2ctl -M ](./Imagenes/Tabla%201/Captura%20010%20tabla%201%20aplicaciones.png) |
| `a2enmod` / `a2dismod` | Activa o desactiva módulos. En la captura aparecen escritos juntos y por eso dan error. | ![Captura de a2enmod / a2dismod ](./Imagenes/Tabla%201/Captura%20011%20tabla%201%20aplicaciones.png) |
| `a2ensite` / `a2dissite` | Activa o desactiva sitios. La captura muestra el error al escribir ambos juntos. | ![Captura de a2ensite / a2dissite ](./Imagenes/Tabla%201/Captura%20012%20tabla%201%20aplicaciones%20.png) |
| `a2enconf` / `a2disconf` | Activa o desactiva configuraciones. La captura muestra el error al escribir ambos juntos. | ![Captura de a2enconf / a2disconf ](./Imagenes/Tabla%201/Captura%20013%20tabla%201%20aplicaciones.png) |

❓**Cuándo conviene usar reload en lugar de restart?**

Conviene usar `reload` cuando hacemos cambios en la configuración de Apache, ya que los aplica sin interrumpir las conexiones activas. En cambio, `restart` reinicia completamente el servicio y puede provocar una breve interrupción.

## Apartado 5. Ficheros y directorios importantes

Primero consultamos la estructura de Apache:
```bash
ls -l /etc/apache2/
```
Esta carpeta contiene los archivos de configuración del servidor.

![Carpetas de Apache](./Imagenes/Captura%20004%20aplicaciones.png)

| Ruta | Para qué sirve |
|---|---|
| `/etc/apache2/apache2.conf` | Configuración principal. |
| `/etc/apache2/ports.conf` | Puertos que utiliza Apache. |
| `/etc/apache2/sites-available/` | Sitios configurados y disponibles. |
| `/etc/apache2/sites-enabled/` | Sitios activados. |
| `/etc/apache2/mods-available/` | Módulos disponibles. |
| `/etc/apache2/mods-enabled/` | Módulos activados. |
| `/etc/apache2/conf-available/` | Configuraciones adicionales disponibles. |
| `/etc/apache2/conf-enabled/` | Configuraciones adicionales activadas. |
| `/etc/apache2/envvars` | Variables como el usuario y grupo de Apache. |
| `/var/www/html/` | Carpeta de la web predeterminada. |
| `/var/log/apache2/access.log` | Registro de accesos. |
| `/var/log/apache2/error.log` | Registro de errores. |

Después comprobamos los enlaces de los sitios activos:
```bash
ls -l /etc/apache2/sites-enabled/
```
**¿Por qué se usan enlaces simbólicos entre `available` y `enabled`?**

Porque `available` guarda las configuraciones y `enabled` enlaza solo las que están activas. Así podemos activar o desactivar sitios sin borrar sus archivos originales.
## Apartado 6. Modificaciones típicas del servicio

Antes de empezar hacemos una copia de seguridad de la configuración principal:
```bash
sudo cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.bak
```
### 6.1. Cambiar la página de inicio

Primero sustituimos la web inicial por un título personalizado:
```bash
echo "<h1>Servidor de Jesús</h1>" | sudo tee /var/www/html/index.html
```
`echo` escribe el contenido HTML y `tee` lo guarda en el archivo `index.html`.

### 6.2. Cambiar el puerto de escucha

Después cambiamos temporalmente el puerto 80 por el **8080**. Abrimos estos archivos:
```bash
sudo nano /etc/apache2/ports.conf
sudo nano /etc/apache2/sites-available/000-default.conf
```
En el primero cambiamos `Listen 80` por `Listen 8080` y en el segundo `<VirtualHost *:80>` por `<VirtualHost *:8080>`.

Comprobamos y aplicamos los cambios:
```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -I http://localhost:8080
```
También entramos desde el navegador utilizando `:8080` y funcionó.

![Página en el puerto 8080](./Imagenes/Captura%20005%20apartado%206%20apliciones.png)

### 6.3. Definir el nombre del servidor

Para evitar el aviso de que Apache no identifica su nombre, creamos y activamos esta configuración:
```bash
echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/servername.conf
sudo a2enconf servername
sudo systemctl reload apache2
```
`ServerName` establece el nombre global del servidor; **no cambia el título de la página web**.

![Configuración del nombre del servidor](./Imagenes/Captura%20006%20apartado%206%20aplicaciones.png)

### 6.4. Cambiar el correo del administrador

Después abrimos el archivo del sitio predeterminado:
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Cambiamos `ServerAdmin` pof nuestro correo.

![Cambio de ServerAdmin](./Imagenes/Captura%20007%20apartado%206%20apicaciones.png)

### 6.5. Personalizar el error 404

Por último creamos una página para cuando alguien solicite una dirección que no existe:
```bash
echo "<h1>Error 404 - Pagina no encontrada</h1><p>La pagina que buscas no existe.</p>" | sudo tee /var/www/html/error404.html
```
Después editamos `000-default.conf`:
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Y añadimos dentro del Virtual Host:
```apache
ErrorDocument 404 /error404.html
```
Tras recargar Apache, probamos una dirección inexistente y apareció nuestro mensaje personalizado.

![Error 404 personalizado](./Imagenes/Captura%20008%20apartado%206%20aplicaciones.png)
## Apartado 7. Módulos de Apache

Los **módulos** añaden funciones a Apache. Primero consultamos cuáles estaban cargados y cuáles teníamos disponibles:
```bash
apache2ctl -M
ls /etc/apache2/mods-available/
```
A continuación, estos comandos permiten activar `rewrite`, `headers` y `ssl`:
```bash
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod ssl
sudo systemctl restart apache2
apache2ctl -M | grep -E "rewrite|headers|ssl"
```
El último comando sirve para comprobar si esos módulos aparecen cargados.

| Módulo | Función |
|---|---|
| `mod_rewrite` | Reescribe direcciones URL. |
| `mod_headers` | Modifica cabeceras HTTP. |
| `mod_ssl` | Da soporte a HTTPS. |
| `mod_dir` | Gestiona páginas de inicio como `index.html`. |
| `mod_alias` | Crea alias y redirecciones. |
| `mod_auth_basic` | Permite autenticación básica. |
| `mod_status` | Muestra el estado del servidor. |
| `mod_autoindex` | Puede listar archivos de directorios. |

Finalmente, desactivamos el módulo `status` y reininiciamos:
```bash
sudo a2dismod status
sudo systemctl restart apache2
```
**❓¿Qué ocurre si desactivamos un módulo del que depende otro?**

La función que depende de él puede dejar de funcionar o Apache puede avisar del problema. Por eso hay que comprobar las dependencias antes de desactivarlo.
## Apartado 8. Varios sitios en un mismo servidor (Virtual Hosts)

Los **Virtual Hosts** permiten alojar varias webs en el mismo Apache. Creamos `sitio1.local` y `sitio2.local`.

### 8.1. Crear los directorios y páginas

Primero creamos las carpetas de las dos webs:
```bash
sudo mkdir -p /var/www/sitio1 /var/www/sitio2
```
Después creamos una página diferente en cada una:
```bash
echo "<h1>Bienvenido a SITIO 1</h1>" | sudo tee /var/www/sitio1/index.html
echo "<h1>Bienvenido a SITIO 2</h1>" | sudo tee /var/www/sitio2/index.html
```
### 8.2. Configurar los Virtual Hosts

Abrimos el archivo de configuración del primer sitio:
```bash
sudo nano /etc/apache2/sites-available/sitio1.conf
```
Y añadimos:
```apache
<VirtualHost *:80>
    ServerName sitio1.local
    ServerAdmin admin@sitio1.local
    DocumentRoot /var/www/sitio1
    <Directory /var/www/sitio1>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/sitio1_error.log
    CustomLog ${APACHE_LOG_DIR}/sitio1_access.log combined
</VirtualHost>
```
![Configuración de sitio1](./Imagenes/Captura%20009%20apartado%208%20aplicaciones.png)

Después creamos `sitio2.conf` cambiando el nombre y las rutas:
```bash
sudo nano /etc/apache2/sites-available/sitio2.conf
```
```apache
<VirtualHost *:80>
    ServerName sitio2.local
    ServerAdmin admin@sitio2.local
    DocumentRoot /var/www/sitio2
    <Directory /var/www/sitio2>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/sitio2_error.log
    CustomLog ${APACHE_LOG_DIR}/sitio2_access.log combined
</VirtualHost>
```
![Configuración de sitio2](./Imagenes/Captura%20010%20apartado%208%20aplicaciones.png)

`ServerName` indica el nombre de la web, `DocumentRoot` su carpeta, `<Directory>` los permisos de acceso y los archivos `Log` sus registros. En la captura de sitio2 aparecen por error los nombres de logs de sitio1; arriba están las líneas correctas para separarlos.

### 8.3. Activar los sitios

Activamos los dos sitios, desactivamos el predeterminado y comprobamos la configuración:
```bash
sudo a2ensite sitio1.conf sitio2.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
apache2ctl -S
```
La salida de `apache2ctl -S` muestra los dos sitios y que `sitio1.local` es el predeterminado.

![Virtual Hosts activos](./Imagenes/Captura%20011%20apartado%208%20aplicaciones.png)

### 8.4. Resolución de nombres

Editamos el archivo de nombres del propio Ubuntu:
```bash
sudo nano /etc/hosts
```
Y añadimos:
```text
127.0.0.1 sitio1.local sitio2.local
```
`127.0.0.1` apunta al propio Ubuntu, donde está Apache.

### 8.5. Pruebas

Por último comprobamos las dos páginas:
```bash
curl http://sitio1.local
curl http://sitio2.local
```
Cada una devolvió su mensaje de bienvenida.

![Pruebas de sitio1 y sitio2](./Imagenes/Captura%20013%20apartado%208%20aplicaciones.png)

Al principio sitio1 mostraba el texto de sitio2 porque nos equivocamos al editar un `index.html`. Lo corregimos y las dos webs funcionaron.

**❓¿Qué diferencia hay entre Virtual Hosts por nombre y por IP?**

Los de **nombre** diferencian las webs mediante `ServerName`, aunque compartan IP y puerto. Los basados en **IP** utilizan direcciones distintas. Nosotros hemos usado los **basados en nombre**.
## Apartado 9. Permisos y propietarios

Primero comprobamos con qué usuario trabaja Apache y qué permisos tienen los archivos:
```bash
ps aux | grep apache2
grep -E "APACHE_RUN_(USER|GROUP)" /etc/apache2/envvars
ls -ld /var/www/sitio1 /var/www/sitio1/index.html
```
Apache utiliza el usuario `www-data` para sus procesos de trabajo.

Después cambiamos el propietario del sitio y aplicamos los permisos que pide la práctica:
```bash
sudo chown -R www-data:www-data /var/www/sitio1
sudo find /var/www/sitio1 -type d -exec chmod 755 {} \;
sudo find /var/www/sitio1 -type f -exec chmod 644 {} \;
```
`chown` cambia el propietario; `find` aplica `755` a las carpetas y `644` a los archivos.

Para comprobar qué ocurre sin permisos, ejecutamos:
```bash
sudo chmod 000 /var/www/sitio1/index.html
curl http://sitio1.local
```
Apache devolvió el error **403 Forbidden**: el archivo existe, pero no se permite acceder a él.

![Error 403](./Imagenes/Captura%20014%20apartado%209%20aplicaciones.png)

A continuación consultamos el registro del error:
```bash
sudo tail -n 20 /var/log/apache2/sitio1_error.log
```
![Registro del error 403](./Imagenes/Captura%20015%20apartado%209%20aplicaciones.png)

Por último restauramos los permisos del archivo:
```bash
sudo chmod 644 /var/www/sitio1/index.html
```
**¿Qué significan `755` y `644`?**

| Permisos | Propietario | Grupo y otros |
|---|---|---|
| `755` | Leer, escribir y ejecutar | Leer y ejecutar |
| `644` | Leer y escribir | Solo leer |

En los directorios, ejecutar significa poder acceder a ellos.

**¿Por qué no se debe utilizar `777`?** Porque todos los usuarios podrían modificar los archivos, lo cual es un riesgo de seguridad.

**¿Por qué el contenido web no debe ser escribible por todos ni ejecutarse como root?** Porque hay que limitar los permisos de Apache para evitar cambios no autorizados. Un archivo puede pertenecer a `root` si Apache solo necesita leerlo; lo peligroso es darle privilegios innecesarios.
## Conclusiones

En esta práctica hemos aprendido a instalar Apache, administrarlo y modificar su configuración. Lo que más nos ha servido ha sido crear **dos webs en el mismo servidor** mediante Virtual Hosts y comprobar cómo los permisos afectan al acceso. También solucionamos un error en el HTML de sitio1 y provocamos un **403** para localizarlo en los registros. Ahora entendemos mejor cómo funciona un servidor web y cómo revisar sus fallos.
