==============================================================================================================
Errores comunes que pueden aparecer
==============================================================================================================

“Python Image Library fails with message “decoder JPEG not available” – PIL”
------------------------------------------------------------------------------------------
Se soluciona instalando los paquetes necesarios:
	$ sudo apt-get install libjpeg-dev

Si estas en Ubuntu 14.04, tenés que también instalar esto
	$ sudo apt-get install libjpeg8-dev


ImportError: No module named ‘apport’
------------------------------------------------------------------------------------------

Se soluciona corriendo los siguientes comandos:
	export LC_ALL="en_US.UTF-8"
	export LC_CTYPE="en_US.UTF-8"
	sudo dpkg-reconfigure locales

DatabaseError: permission denied to create extension “postgis”
------------------------------------------------------------------------------------------

HINT: Must be superuser to create this extension.
Solución:
	su postgres
	psql
	alter role user_name superuser;
En otra pantalla instalamos la extension usando este usuario, una vez instalada le quitamos los privilegios de superusuario.
	alter role user_name nosuperuser;
 
