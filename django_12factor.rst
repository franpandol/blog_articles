==============================================================================================================
Django settings con 12 factor
==============================================================================================================

`12 factor <https://12factor.net>`_ es una metodología de desarrollo orientada a SaaS. Uno de sus principios dice que las variables de configuración deben estar separados del código fuente, y sugiere utilizar variables de entorno. Es decir, en nuestro settings.py todas las claves secretas se deben obtener del entorno. 

¿Cómo lo aplicamos en Django? 
------------------

Usando `python-dotenv <https://github.com/theskumar/python-dotenv>`_.
	Reads the key,value pair from .env file and adds them to environment variable. It is great for managing app settings during development and in production using 12-factor principles.

Creamos un archivo .env donde vamos a guardar nuestras claves. Podemos crear uno para producción y otro para desarrollo. .prod_env y .dev_env

::

	SECRET_KEY="oqbqb&&1z*hhsqb&afas1235&!142*%bx-*2312"
	DATABASE_NAME="jobs_db"
	DATABASE_USER="jobs_db_user"
	DATABASE_PASSWORD="jobs_db_password"
	SENDGRID_API_KEY='SG.cyXasdfasdfRdfxZ3hDpsdf9t6s8'
	TWILIO_ACCOUNT_SID='ACc0fe8dfas1230b521e502d7f41ad481d'

Ahora en nuestro settings.py agregamos las siguientes lineas para que dotenv lea nuestro archivo .env y cargue las variables de entorno

::

	from dotenv import load_dotenv
	from os.path import join, dirname

	dotenv_path = join(dirname(__file__), '.env')
	load_dotenv(dotenv_path, verbose=True)


Lo que nos permite ahora obtener las variables haciendo:

::
	
	SECRET_KEY = os.getenv("SECRET_KEY")


Para utilizarlo en django hay que hacer un pequeño ajuste en los archivos manage.py y wsgi.py

Modificaciones en manage.py para usar las variables de entorno locales

::

	#!/usr/bin/env python
	import os
	import sys
	from os.path import join, dirname
	from dotenv import load_dotenv

	if __name__ == "__main__":
	    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "yourproject.settings.local")
	    dotenv_path = join(dirname(__file__), 'yourproject/settings/.env_local')
	    load_dotenv(dotenv_path, verbose=True)
	    try:
	        from django.core.management import execute_from_command_line
	    except ImportError:
	        # The above import may fail for some other reason. Ensure that the
	        # issue is really that Django is missing to avoid masking other
	        # exceptions on Python 2.
	        try:
	            import django
	        except ImportError:
	            raise ImportError(
	                "Couldn't import Django. Are you sure it's installed and "
	                "available on your PYTHONPATH environment variable? Did you "
	                "forget to activate a virtual environment?"
	            )
	        raise
	    execute_from_command_line(sys.argv)



Modificaciones en wsgy.py para usar las variables de entorno de producción

::

	import os

	from django.core.wsgi import get_wsgi_application
	from os.path import join, dirname
	from dotenv import load_dotenv

	os.environ.setdefault("DJANGO_SETTINGS_MODULE", "yourproject.settings.prod")

	dotenv_path = join(dirname(__file__), 'settings/.env')
	load_dotenv(dotenv_path, verbose=True)

	application = get_wsgi_application()
