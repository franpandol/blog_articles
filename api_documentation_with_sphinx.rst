==============================================================================================================
Documentar una API REST Django con Sphinx
==============================================================================================================

Instalación
---------------------------------------------

Instalamos Sphinx

	``$ pip install sphinx``

Creamos la carpeta donde vamos a guardar todos los archivos .rst de la documentación

	``mkdir ~/yourproject/docs/``
	``cd ~/yourproject/docs/``

Ejecutamos el wizard de sphinx

	``$ sphinx-quickstart``

Respondemos las preguntas y se van a crear los archivos básicos. Abrimos el que se llama conf.py y agregamos el siguiente código

::

	import os
	import sys

	sys.path.insert(0, os.path.abspath('..'))
	sys.path.append(os.path.dirname(__file__))
	os.environ.setdefault("DJANGO_SETTINGS_MODULE", "yourproject.settings.base")

	from django.conf import settings

	import django
	django.setup()

Creamos los archivos de nuestra documentación agregando en el index las secciones. 

::

	.. toctree::
	   :maxdepth: 4
	   :caption: Contents:

		api
		token
		authentication


Para ver un ejemplo completo de documentación revisar la del repositorio dreamjobs.
`DreamJob Docs <https://github.com/franpandol/dreamjob/tree/master/docs>`_.

Para compilar nuestrors archivos vamos al directorio docs/ y ejecutamos.

`make html`

Para terminar vamos a servir la documentación utilizando nuestro servidor Django, para ello vamos a utilizar una app llamada `django-docs <https://github.com/littlepea/django-docs>`_.

Pasos para utilizar django-docs

Instalación

	pip install django-docs

Agregamos docs a INSTALLED_APPS en settings.py:

	INSTALLED_APPS = (
	    ...
	    'docs',
	    ...
	)

Agregamos docs.urls a urls.py:

	urlpatterns = [
	    ...
	    url(r'^docs/', include('docs.urls')),
	    ...
	]

Editamos la configuración:

	DOCS_ROOT = os.path.join(PROJECT_PATH, '../docs/_build/html')
	DOCS_ACCESS = 'staff'