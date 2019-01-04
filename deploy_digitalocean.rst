==============================================================================================================
Cómo hacer un despliegue de Django 2.1 en DigitalOcean usando nginx, gunicorn y supervisor
==============================================================================================================

Crear un droplet en DigitalOcean
---------------------------------------------

Vamos a recibir un mail con el usuario root y el password que utilizaremos para conectarnos al servidor a través de ssh. Ejecutamos el siguiente comando en la consola:


.. code-block:: sh
	$ ssh root@server_ip

Al ser la primera vez que hacemos login nos va a pedir que cambiemos la contraseña del usuario root.

**Actualizar el sistema** 

Una vez que ya estamos adentro del sistema el primer paso es crear otro usuario para no trabajar usando root.

.. code-block:: sh
	$ adduser webapps

Agregamos el user webapps al grupo ‘sudo’.

.. code-block:: sh
	$ gpasswd -a webapps sudo

Ya nos podemos loguear con el usuario webapps. El siguiente paso es actualizar el sistema utilizando aptitude.

.. code-block:: sh
	$ su webapps
	$ sudo apt-get install aptitude
	$ sudo aptitude update
	$ sudo aptitude upgrade

Python y virtualenv
-------------------------

Instalamos python y creamos el entorno virtual

.. code-block:: sh
	$ sudo aptitude install python-virtualenv
	$ sudo aptitude install -y build-essential
	$ sudo aptitude install -y python3.5-dev


Creamos el directorio donde vamos a guardar el entorno virtual.
Ejecutamos virtualenv con la opción -p para definir nosotros que versión de python queremos utilizar. Escribir el path completo puede ser reemplazado por `which python3`.
Activamos el entorno virtual:

.. code-block:: sh
	$ mkdir ~/.virtualenvs
	$ virtualenv -p /usr/bin/python3.5 ~/.virtualenvs/dreamjob
	$ source ~/.virtualenvs/dreamjob/bin/activate

Descargamos el proyecto
.. code-block:: sh
	(dreamjob)$ mkdir /home/webapps/projects
	(dreamjob)$ git clone 'url_de_tu_repositorio'
	(dreamjob)$ cd dreamjob

Instalamos las dependencias
.. code-block:: sh
	(dreamjob)$ pip install -r requirements.txt

Creamos las carpetas para los archivos estáticos:
.. code-block:: sh
	(dreamjob)$ mkdir ~/projects/dreamjob/static
	(dreamjob)$ mkdir ~/projects/dreamjob/media
	(dreamjob)$ mkdir ~/projects/dreamjob/static/css ~/projects/dreamjob/static/js ~/projects/dreamjob/static/fonts

Instalamos bower para manejar las dependencias del frontend.
.. code-block:: sh
	(dreamjob)$ sudo apt-get install nodejs
	(dreamjob)$ sudo ln -s /usr/bin/nodejs /usr/bin/node
	(dreamjob)$ npm -g install bower
	(dreamjob)$ mkdir ~/projects/dreamjob/components #la carpeta que va a utilizar bower
	(dreamjob)$ python manage.py bower install
	(dreamjob)$ python manage.py collectstatic


Base de datos
-------------------------

**Instalamos postgreSQL**
.. code-block:: sh
	(dreamjob)$ sudo aptitude install postgresql postgresql-contrib postgresql-server-dev-9.5

**Instalamos PostGIS**
PostGIS convierte al sistema de administración de bases de datos PostgreSQL en una base de datos espacial mediante la adición de tres características: tipos de datos espaciales, índices espaciales y funciones que operan sobre ellos. 

.. code-block:: sh
	(dreamjob)$ sudo apt-get install postgresql-9.5-postgis-2.0
	(dreamjob)$ sudo -u postgres psql
	postgres=# CREATE ROLE owner LOGIN PASSWORD '' NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;
	postgres=# CREATE DATABASE WITH OWNER = owner;
	postgres=# CREATE EXTENSION postgis;

Creamos y aplicamos las migraciones:
.. code-block:: sh
	(dreamjob)$ python manage.py makemigrations
	(dreamjob)$ python manage.py migrate

gunicorn
-------------------------
Instalamos con pip:
.. code-block:: sh
	(dreamjob)$ pip install gunicorn

Creamos el script de arranque y lo hacemos ejecutable:
.. code-block:: sh
	(dreamjob)$ vim ~/bin/gunicorn_start

Contenido del archivo
.. code-block:: bash
	#!/bin/bash
	NAME="dreamjob_app" # Name of the application
	DJANGODIR=/home/webapps/projects/dreamjobbackend/ # Django project directory
	SOCKFILE=/home/webapps/projects/dreamjobbackend/run/gunicorn.sock # we will communicte using this unix socket
	USER=webapps # the user to run as
	#GROUP=webapp # the group to run as
	NUM_WORKERS=3 # how many worker processes should Gunicorn spawn
	DJANGO_SETTINGS_MODULE=dreamjobbackend.settings # which settings file should Django use
	DJANGO_WSGI_MODULE=dreamjobbackend.wsgi # WSGI module name
	 
	echo "Starting $NAME as `whoami`"
	 
	# Activate the virtual environment
	cd $DJANGODIR
	source /home/webapps/.virtualenvs/dreamjob/bin/activate
	export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
	export PYTHONPATH=$DJANGODIR:$PYTHONPATH
	 
	# Create the run directory if it doesn't exist
	RUNDIR=$(dirname $SOCKFILE)
	test -d $RUNDIR || mkdir -p $RUNDIR
	 
	# Start your Django Unicorn
	# Programs meant to be run under supervisor should not daemonize themselves (do not use --daemon)
	exec /home/webapps/.virtualenvs/dreamjob/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
	--name $NAME \
	--workers $NUM_WORKERS \
	--user=$USER \
	--bind=unix:$SOCKFILE \
	--env DJANGO_SETTINGS_MODULE=dreamjobbackend.settings \
	--log-level=debug \
	--log-file=-


.. code-block:: sh
	(dreamjob)$ sudo chmod +x ~/bin/gunicorn_start


Y creamos la carpeta donde se va a crear el archivo .sock
.. code-block:: sh
	(dreamjob)$ mkdir ~/projects/dreamjob/run


Tip: Cómo reemplazar texto en Vim
.. code-block:: sh
	:g/texto_a_sustituir/s//texto_nuevo/g

supervisor
-------------------------
Instalamos con aptitude:
.. code-block:: sh
	(dreamjob)$ pip install setproctitle
	(dreamjob)$ sudo aptitude install supervisor

Creamos el archivo de configuración donde le indicamos que corra el script de arranque de gunicorn:
.. code-block:: sh
	(dreamjob)$ sudo vim /etc/supervisor/conf.d/dreamjob

Creamos los archivos de logs y reiniciamos supervisor:
.. code-block:: sh
	(dreamjob)$ mkdir ~/logs
	(dreamjob)$ touch ~/logs/gunicorn_supervisor.log
	(dreamjob)$ sudo service supervisor start
	(dreamjob)$ sudo supervisorctl reread
	(dreamjob)$ sudo supervisorctl update
	(dreamjob)$ sudo supervisorctl restart

nginx
-------------------------
Instalamos con aptitude:
.. code-block:: sh
	(dreamjob)$ sudo aptitude install nginx

Creamos el archivo de configuración de nuestro sitio:
.. code-block:: sh
	(dreamjob)$ sudo vim /etc/nginx/sites-available/dreamjob.conf

Creamos un enlace simbólico:
.. code-block:: sh
	(dreamjob)$ sudo ln -s /etc/nginx/sites-available/ /etc/nginx/sites-
enabled/

Creamos los archivos de log para nginx:
.. code-block:: sh
	(dreamjob)$ touch ~/logs/nginx-access.log
	(dreamjob)$ touch ~/logs/nginx-error.log

Reiniciamos el servicio:
.. code-block:: sh
	(dreamjob)$ sudo service nginx restart

