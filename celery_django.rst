==============================================================================================================
Cómo integrar y hacer un despliegue de Celery con Django 2.1
==============================================================================================================

Instalación
---------------------------------------------

Instalamos redis
	``$ sudo aptitude install redis-server``

Chequeamos que esté corriendo 
	``$ redis-cli ping``

Instalamos celery con redis como broker
	``pip install -U "celery[redis]"``



Integración en Django
---------------------------------------------

Agregamos valores de settings para celery en settings.py

``vim your_project/your_project/settings/settings_production.py``

::
	
	CELERY_BROKER_URL = 'redis://localhost:6379/0'
	CELERY_RESULT_BACKEND = 'redis://localhost:6379'
	CELERY_ACCEPT_CONTENT = ['application/json']
	CELERY_TASK_SERIALIZER = 'json'
	CELERY_RESULT_SERIALIZER = 'json'

Creamos un archivo celery.py que instancie la app

``vim your_project/your_project/celery.py``

::

	from __future__ import absolute_import, unicode_literals
	import os
	from celery import Celery

	# set the default Django settings module for the 'celery' program.
	os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'your_project.settings')

	app = Celery('proj')

	# Using a string here means the worker doesn't have to serialize
	# the configuration object to child processes.
	# - namespace='CELERY' means all celery-related configuration keys
	#   should have a `CELERY_` prefix.
	app.config_from_object('django.conf:settings', namespace='CELERY')

	# Load task modules from all registered Django app configs.
	app.autodiscover_tasks()


Para asegurarnos que la app es cargada cuando inicie Django, la importamos en el archivo __init__.py

``vim your_project/your_project/__init__.py``

::

	from __future__ import absolute_import, unicode_literals

	# This will make sure the app is always imported when
	# Django starts so that shared_task will use this app.
	from .celery import app as celery_app

	__all__ = ('celery_app',)


Con la linea:
	:strong:`app.autodiscover_tasks()`

Celery puede descubrir automáticamente todas las tareas que esten en los archivos tasks.py de todas las apps en INSTALLED_APPS

- app1/
	- tasks.py
	- models.py
- app2/
	- tasks.py
	- models.py

 
Veamos un ejemplo práctico
---------------------------------------------

Dentro del archivo tasks.py de nuestra app dreambjobs creamos una task que llame al método update_jobs.

``vim your_project/dreamjobs/task.py``

::
	
	from celery.decorators import task
	from celery.task.schedules import crontab
	from celery.utils.log import get_task_logger
	from celery.decorators import periodic_task

	from dreamjobs.utils import update_jobs

	logger = get_task_logger(__name__)


	@task(name="async_update_jobs")
	def async_update_jobs(email, message):
		logger.info("async_update_jobs")
		update_jobs()


	@periodic_task(
		run_every=(crontab(minute='*/120')),
		name="async_update_jobs",
		ignore_result=True
	)
	def periodic_update_jobs(email, message):
		logger.info("async_update_jobs")
		update_jobs()


Despliegue en servidor linux
---------------------------------------------
Creamos el archivo que va a ejecutar el worker /home/user/bin/start_celery y le damos permisos de ejecución

``vim /home/user/bin/start_celery``
``sudo chmod +x /home/user/bin/start_celery``

::

	#!/bin/bash

	source /home/webapps/.virtualenvs/your_project/bin/activate
	cd /home/webapps/projects/your_project
	exec celery --app=your_project.celery:app worker --loglevel=DEBUG

Creamos un archivo de configuración para que supervisor lo gestione en /etc/supervisor/conf.d/celery.conf

::

	[program:celery_worker]
	command=/home/webapps/bin/celery_start
	stdout_logfile=/home/webapps/logs/celery_worker.log
	redirect_stderr=true
	autostart=true
	autorestart=true

Creamos el archivo de log y ejecutamos los comandos para que supervisor lea el archivo celery.conf

::

	touch /home/webapps/logs/celery_worker.log
	sudo supervisorctl reread
	sudo supervisorctl update



