Testing unitario con Django y pytest
==============================================================================================================

Vamos a crear un archivo tests por cada clase o función que vayamos a testear. 
Usemos de ejemplo real un test para un endpoint de la app dreamjobs. Este endpoint
tiene que devolver un QuerySet de DreamJob.
Para facilitar los tests usamos djet 

    |djet makes performing unit tests for your views easier by providing ViewTestCase. Instead of
    |self.client, you can use self.factory, which is an extended RequestFactory with overridden 
    |shortcuts for creating requests (eg. path is not required parameter).

Creamos una carpeta tests. ``mkdir dreamjobs/api/tests/``

``vim test_dreamjob_list_endpoint.py``

::

    from django.contrib.auth import get_user_model
    from djet import restframework, assertions
    from rest_framework import status
    from api.base.views import DreamJobListEndpoint

    User = get_user_model()

    class DreamjobListEndpointTest(restframework.APIViewTestCase,
                              assertions.StatusCodeAssertionsMixin):
        view_class = DreamJobListEndpoint

        def setUp(self):
            self.user = User.objects.create_user(
                username='gordon',
                password='secret',
            )
            self.user.is_active = True
            self.user.save()

        def test_response_200(self):
            request = self.factory.get()
            response = self.view(request)
            self.assert_status_equal(response, status.HTTP_200_OK)

Instalamos los paquetes que nos van a ayudar con el testing

::

    pip install coverage
    pip install pytest
    pip install pytest-cov
    pip install pytest-django

Crear el archivo pytest.ini con datos de configuración

::

    [pytest]
    addopts = --nomigrations
    DJANGO_SETTINGS_MODULE = jobs.settings.base


Ejecutamos los tests y vemos un reporte en la consola

::

    coverage run --source='.' -m pytest
    coverage report
    pytest --cov-report term-missing --cov=api.views
