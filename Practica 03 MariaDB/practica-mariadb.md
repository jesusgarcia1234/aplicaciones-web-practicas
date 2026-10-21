
# Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04

**Alumno:** Jesús García Ibáñez  
**Curso:** 2.º SMX  
**Asignatura:** Aplicaciones Web  
**Fecha:** 21-10-2026

## Índice
1. [Preparación del sistema](#apartado-1-preparación-del-sistema)
2. [Instalación de MariaDB](#apartado-2-instalación-de-mariadb)
3. [Comprobación del funcionamiento](#apartado-3-comprobación-del-funcionamiento)
4. [Comandos principales](#apartado-4-comandos-principales)
5. [¿Qué pasa si MariaDB está parado?](#apartado-5-qué-pasa-si-mariadb-está-parado)
6. [Instalación de PHP](#apartado-6-instalación-de-php)
7. [Comprobación final](#apartado-7-comprobación-final)
8. [Conclusiones](#conclusiones)

---

<br>
<br>
<br>
<br>
<br>
<br>

## Apartado 1. Preparación del sistema

Primero desactivamos los dos sitios virtuales de la práctica anterior y volvemos a activar el sitio predeterminado de Apache:

```bash
sudo a2dissite sitio1.conf sitio2.conf
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
```

`a2dissite` desactiva los sitios, `a2ensite` activa el predeterminado y `reload` aplica la configuración sin reiniciar completamente Apache.

Después actualizamos la lista de paquetes e instalamos las actualizaciones disponibles:

```bash
sudo apt update
sudo apt upgrade -y
```

`update` consulta las versiones disponibles y `upgrade` actualiza los paquetes instalados.

Durante este paso tuve un problema porque el proceso de actualizaciones automáticas `unattended-upgrade` bloqueó el gestor de paquetes. Explico cómo lo solucioné en el apartado 2.

Por último comprobamos si Apache sigue activo:

```bash
systemctl is-active apache2
```

El resultado esperado es `active`.

![Comprobación de Apache activo](./Imagenes/Captura%20001%20apartado%201.png)

[Volver al índice](#índice)

## Apartado 2. Instalación de MariaDB

Para instalar el servidor de bases de datos ejecutamos:

```bash
sudo apt install mariadb-server -y
```

El parámetro `-y` acepta automáticamente la confirmación de instalación.

### Problema durante la instalación

Al principio no pude instalar MariaDB porque `unattended-upgrade` tenía bloqueado el gestor de paquetes. Revisé los procesos y los registros del sistema y descubrí que la actualización se había quedado atascada en `needrestart`.

Después de comprobar que no había ningún proceso `dpkg` trabajando, cerré los procesos bloqueados. También tuve otro problema con `debconf`, que seguía bloqueando la instalación, así que tuve que cerrar los procesos que habían quedado activos.

La instalación había quedado a medias, por lo que ejecuté:

```bash
sudo dpkg --configure -a
sudo apt --fix-broken install
```

Con el primer comando intenté terminar de configurar los paquetes pendientes y con el segundo conseguí reparar las dependencias y completar la instalación de `mariadb-server`. Finalmente pude solucionarlo sin reinstalar Ubuntu.

Una vez instalado, comprobamos la versión:

```bash
sudo apt install mariadb-server -y
mariadb --version
```

En mi caso se instaló **MariaDB 10.11.14**. El último `apt install` me confirmó que ya tenía la versión más reciente disponible.

![Versión de MariaDB](./Imagenes/Captura%20002%20apartado%202.png)

[Volver al índice](#índice)

## Apartado 3. Comprobación del funcionamiento

### 3.1. Estado del servicio

Comprobamos que el servicio de MariaDB está funcionando:

```bash
sudo systemctl status mariadb
```

El estado correcto es `active (running)`. Para salir de esta pantalla pulsamos `q`.

![Estado activo de MariaDB](./Imagenes/Captura%20003%20aprtado%203.png)

Después comprobamos si se inicia automáticamente al encender Ubuntu:

```bash
systemctl is-enabled mariadb
```

El resultado esperado es `enabled`.

![Comprobación del arranque automático de MariaDB](./Imagenes/Captura%20004%20apartado%203.png)

### 3.2. Entrar en MariaDB

Accedemos a la consola de la base de datos con:

```bash
sudo mariadb
```

Si funciona, aparece el mensaje de bienvenida y el indicador `MariaDB [(none)]>`.

![Entrada en MariaDB](./Imagenes/Captura%20005%20apartado%203.png)

Salimos escribiendo:

```sql
exit;
```

[Volver al índice](#índice)

## Apartado 4. Comandos principales

Probamos los principales comandos para administrar el servicio. En esta tabla explicamos qué hace cada uno e incluimos su captura correspondiente.

| Comando | Función | Captura |
|---|---|---|
| `sudo systemctl start mariadb` | Inicia MariaDB. | ![Iniciar MariaDB](./Imagenes/Tabla/Captura%20001%20tabla%20apartado%204.png) |
| `sudo systemctl stop mariadb` | Detiene MariaDB. | ![Detener MariaDB](./Imagenes/Tabla/Captura%20002%20tabla%20apartado%204.png) |
| `sudo systemctl restart mariadb` | Reinicia el servicio. | ![Reiniciar MariaDB](./Imagenes/Tabla/Captura%20003%20tabla%20apartado%204.png) |
| `sudo systemctl status mariadb` | Muestra si está funcionando y sus mensajes. | ![Estado de MariaDB](./Imagenes/Tabla/Captura%20004%20tabla%20apartado.png) |
| `sudo systemctl enable mariadb` | Activa el inicio automático al arrancar Ubuntu. | No hay captura subida para este comando. |

[Volver al índice](#índice)

## Apartado 5. ¿Qué pasa si MariaDB está parado?

Primero detenemos MariaDB para comprobar qué ocurre cuando una aplicación intenta conectarse a una base de datos que no está funcionando:

```bash
sudo systemctl stop mariadb
systemctl is-active mariadb
sudo mariadb
```

`is-active` muestra `inactive`. Al intentar acceder con `sudo mariadb`, aparece un error de conexión como `ERROR 2002`, porque el servicio está parado.

![Error de conexión con MariaDB detenido](./Imagenes/Captura%20006%20apartado%205.png)

Después volvemos a iniciar el servicio y comprobamos que funciona correctamente:

```bash
sudo systemctl start mariadb
systemctl is-active mariadb
```

Ahora el resultado esperado es `active`.

![Recuperación de MariaDB](./Imagenes/Captura%20007%20apartado%205.png)

**❓Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos usarías para comprobar si MariaDB está parado?**

Utilizaría `systemctl is-active mariadb` para comprobar rápidamente su estado y `sudo systemctl status mariadb` para ver más detalles del servicio.

[Volver al índice](#índice)

## Apartado 6. Instalación de PHP

WordPress utiliza PHP para ejecutar el código de la web. Instalamos PHP y las extensiones necesarias:

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y
```

### 6.1. Comprobar la versión y los paquetes

Consultamos la versión instalada:

```bash
php -v
```

En Ubuntu 24.04 se espera **PHP 8.3.x**.

![Versión de PHP](./Imagenes/Captura%20008%20apartado%206.png)

Estos son los paquetes instalados y sus funciones:

| Paquete | Función |
|---|---|
| `php` | Lenguaje en el que está programado WordPress. |
| `libapache2-mod-php` | Permite ejecutar PHP desde Apache. |
| `php-mysql` | Permite a PHP conectarse a MariaDB. |
| `php-curl` | Realiza conexiones con otros servidores. |
| `php-gd` | Permite tratar imágenes. |
| `php-mbstring` | Maneja textos y caracteres especiales. |
| `php-xml` | Trabaja con archivos XML. |
| `php-zip` | Permite trabajar con ZIP, plugins y temas. |
| `php-intl` | Gestiona idiomas y formatos regionales. |

### 6.2. Comprobar los módulos

Reiniciamos Apache y comprobamos que tiene cargado el módulo de PHP:

```bash
sudo systemctl restart apache2
sudo apache2ctl -M | grep -E "mpm|php"
```

Deben aparecer `mpm_prefork_module` y `php_module`.

![Módulos de Apache y PHP](./Imagenes/Captura%20009%20apartado%206.png)

Después comprobamos las extensiones que permiten conectarse a MariaDB:

```bash
php -m | grep -i mysql
```

Deben aparecer `mysqli`, `mysqlnd` y `pdo_mysql`.

![Módulos de conexión MySQL](./Imagenes/Captura%20010%20apartado%206.png)

### 6.3. Probar PHP en el navegador

Creamos una página temporal para comprobar que Apache ejecuta PHP:

```bash
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php
```

Desde el navegador entramos en `http://IP_DE_TU_SERVIDOR/info.php`, cambiando el texto por la dirección IP de Ubuntu. Si funciona, aparece una página con la configuración de PHP.

![Página de información de PHP](./Imagenes/Captura%20011%20apartado%206.png)

**Importante:** después de hacer la captura eliminamos el archivo de prueba:

```bash
sudo rm /var/www/html/info.php
ls /var/www/html
```

Así comprobamos que ya no está disponible.

**❓¿Por qué es peligroso dejar `info.php` en un servidor real?**  
Porque muestra información interna sobre PHP, los módulos y la configuración, que podría ayudar a alguien a buscar vulnerabilidades.

**❓¿Para qué sirve el paquete `php-mysql`?**  
Permite que PHP se comunique con MariaDB o MySQL. Es necesario para que WordPress pueda utilizar su base de datos.

[Volver al índice](#índice)

## Apartado 7. Comprobación final

Para terminar, comprobamos que Apache y MariaDB están activos:

```bash
systemctl is-active apache2 mariadb
```

El resultado esperado es `active` en las dos líneas. También verificamos la versión de PHP:

```bash
php -v
```

El estado final del servidor debe ser el siguiente:

| Pieza del LAMP | Programa | Estado |
|---|---|---|
| Linux | Ubuntu 24.04 | Instalado. |
| Apache | Apache2 | Funcionando. |
| MariaDB | MariaDB 10.11 | Funcionando. |
| PHP | PHP 8.3 | Instalado y preparado para MariaDB. |

Por último, la práctica pide crear una **instantánea de VirtualBox** para tener un punto de restauración antes de instalar WordPress. **Pendiente de confirmar que se haya guardado la instantánea.**

[Volver al índice](#índice)

## Conclusiones

En esta práctica completamos el entorno LAMP con la instalación de MariaDB y PHP sobre el servidor Ubuntu que ya tenía Apache. También aprendemos a detectar cuándo MariaDB está parado y cómo recuperar el servicio.

Lo más complicado para mí fue instalar MariaDB, ya que tuve un problema con las actualizaciones automáticas que bloqueó el gestor de paquetes. Tuve que revisar los procesos, detener los que estaban bloqueados y reparar las dependencias hasta que conseguí completar la instalación sin reinstalar Ubuntu.

Con esta práctica dejamos el servidor preparado para instalar WordPress en la siguiente.

[Volver al índice](#índice)
