# ☁️ Un hogar seguro para todos tus datos ☁️

![alt text](https://github.com/JuanRodenas/Nextcloud_server/blob/main/NextcloudHub.jpg)

# Nextcloud
Seguridad🔒 Rendimiento🚀 Control☑️

* 📁 **Accede a tus datos** Puedes almacenar tus archivos, contactos, calendarios y más en un servidor de tu elección.
* 🔄 **Sincroniza tus datos** Mantén tus archivos, contactos, calendarios y más sincronizados entre tus dispositivos.
* 🙌 **Comparte tus datos** ...dando a otros acceso a las cosas que quieres que vean o con las que quieren colaborar.
* 🚀 [Documentación oficial](https://docs.nextcloud.com/server/stable/user_manual/es/)


## PREPARACIÓN DE LOS ARCHIVOS Y DIRECTORIOS

#### Crear los directorios donde se montarán los volúmenes de persistencia
Creamos los directorios donde se montarán los volúmenes de persistencia
~~~
mkdir /patch/to/data/nextcloud/nextcloud
mkdir /patch/to/data/nextcloud/mysql
mkdir /patch/to/data/nextcloud/redis
~~~~

Directorios:
* **files** Contendrá los archivos almacenados en nuestra nube. También contendrá los ficheros de configuración, ficheros de las aplicaciones instaladas, etc. Es importante realizar una copia de seguridad de este directorio/volumen de persistencia.
* **mysql** Contendrá la totalidad de ficheros de nuestra base de datos MySQL.
* **redis** Contiene las bases de datos que genera el servidor Redis. Obviamente también es interesante realizar una copia de seguridad de este directorio.

#
<blockquote class="is-info"><p>Los pasos que se explican a continuación están basados en una red que puede diferir de la que tú tienes montada. Si sigues al pie de la letra todos los pasos, pueden no coincidir con la configuración de tu <em>red</em> y dejarla inservible. Adapta en todo momento lo que a continuación se expone para que cuadre con tu red.</p></blockquote>

#### Crear la red interna para comunicar con los demás contenedores
Creada la red interna, ya podemos levantar el contenedor
~~~~
docker network create nextcloud_internal
~~~~
<sup>Anotar la red de la network para después.</sup>

#
## LEVANTAR EL CONTENEDOR DE NEXTCLOUD
En la misma ubicación que hemos indicado la carpeta Nextcloud, descargamos los archivos:
☑️ [files](https://github.com/JuanRodenas/Nextcloud_server/tree/main/files)
```
  - Modificamos la red `networks:` en el docker-compose
  - Modificamos las passwords y usuarios del `nx.env`
  - Modificamos el dominio en el `.env`
  - Introducimos la red que levante en la red internal de docker-compose: `TRUSTED_PROXIES=172.19.0.0/16`
```

* Levantamos el contenedor con:
~~~
docker-compose up -d
~~~

Una vez ejecutado el comando se descargarán las imagenes del docker-compose y se crearán, levantarán los contenedores.

#### Ver el log del contenedor
* Vemos el contenedor:
~~~
docker logs nextcloud
~~~
* Vemos todos los contenedores del docker-compose:
~~~
docker-compose logs -f
~~~

#
## Configurar configuración de nextcloud
Para poder configurar el archivo `config.php` debemos acceder al contenedor de nextcloud como `root`
~~~
docker exec -u root -t -i nextcloud /bin/bash
~~~
* Una vez que hemos accedido al contenedor, tenemos que actualizar e instalar `sudo` y `nano` para poder modificar el archivo
~~~
apt update && apt install nano imagemagick sudo
~~~
<sup>Cuidado con instalar vim, da problemas a la hora de pegar.</sup>

### Ya instalado los paquetes accedemos al contenedor como `www-data` para poder modificar los archivos
~~~
docker exec -u www-data -t -i nextcloud /bin/bash
~~~

* Creamos la carpeta para los archivos temporales
~~~
mkdir -p /var/www/html/var/tmp
~~~
* Una vez instalado abrimos el archivo en la ruta
~~~
nano config/config.php
~~~
#### Configuración del archivo config:
Añadimos al final del archivo
~~~
  'filelocking.enabled' => true,
  'overwritehost' => 'your_domain',
  'overwrite.cli.url' => 'https://your_domain',
  'htaccess.RewriteBase' => '/',
  'default_phone_region' => 'ES',
  'force_language' => 'es',
  'default_locale' => 'es_ES',
  'force_locale' => 'es_ES',
  'updater.release.channel' => 'stable',
  'trashbin_retention_obligation' => 'auto',
  'tempdirectory' => '/var/www/html/var/tmp',
~~~
* Comprobamos el php si está correctamente:
```
php -i | grep memory_limit
```

#### Una vez realizado los cambios y comprobamos que esté correctamente, reestructuramos de nuevo nextcloud:
~~~
php occ maintenance:repair && \
php occ maintenance:update:htaccess && \
php occ maintenance:mimetype:update-db
~~~

#
### Añadir tarea en el cron.
Para generar la tarea en el cron que se ejecute cada cierto tiempo:
<sup>Comprobar el nombre del contenedor, en el docker-compose se llama nextcloud.</sup>
- Probamos si funciona con este comando:
~~~
docker exec --user www-data nextcloud php -f /var/www/html/cron.php
~~~
- Programa con crontab, en la maquina host, añadiremos el siguiente cron:
~~~
*/5  * * * * docker exec --user www-data nextcloud php -f /var/www/html/cron.php
~~~
Ahora cada 5 minutos, ejecutará cron.php.

#
### ACCEDER A LA WEB O DASHBOARD DE NEXTCLOUD
Con el contenedor levantado tan solo tenemos que abrir el navegador web e ingresar a la URL que indicamos en `TRUSTED_DOMAINS`.
Una vez ingresadas la credenciales tendremos acceso al panel de control. Fíjense que estamos accediendo de forma segura mediante https y TLS.
![alt text](https://github.com/JuanRodenas/Nextcloud_server/blob/main/nextcloud-interfaz-web.png)

### COPIAS DE SEGURIDAD
Si queremos realizar una copias de seguridad de la configuración o recuperar el backup, Pulsa en la imagen para visitar el repositorio de copias de seguridad.
<p align="center">
    <a href="https://github.com/JuanRodenas/Backup/blob/main/README.md">
        <img src="https://github.com/JuanRodenas/Pi-hole_list/blob/main/cloud-backup.png" width="400" height="200">
    </a>
    <br>
    <strong>Pulsa en la imagen para visitar el repositorio de copias de seguridad.</strong>
</p>

### HELP ME
<p> &nbsp;Si quieres contribuir a mejorar Nextcloud o tienes algún error, abre un <code>issue</code> y te ayudaré a solucionarlo:</p>
<sup>ábreme un problema aquí <A HREF="https://github.com/JuanRodenas/Nextcloud_server/issues"> ISSUE </A>.</sup>


### Credits
Si la guía te ha gustado y tienes funcionando un NAS personal, invítame a un café por mi trabajo.
#
<a href="https://www.paypal.com/donate/?hosted_button_id=HVJT2YDSHRZY2" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

## 🎉 ¡Ready!
