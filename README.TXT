#Squid : Varsión 1.1
#


OPTIMIZACION DEL PROXY SQUID

1. INTRODUCCION

	El siguiente documento esta diseñado para que los Administradores de red de 
	la Provincia de Santa Fe implementen en los servidores del Plan Conectar 
	Igualdad. En este caso optimizaremos el uso del proxy Squid.

2. CONFIGURACION

	El archivo de configuración de Squid se encuentra en /etc/squid el mismo se 
	llama squid.conf, para cambiar el comportamiento del proxy debemos editar 
	dicho archivo.

2.1. Cambio en el tamaño del cache de disco

	Toda petición que pasa por el proxy generalmente es guardada en un cache 
	(espacio reservado de disco) para futuras consultas, en caso que un usuario 
	solicite información guardada en el cache la misma se entregará desde ahí 
	evitando el acceso a Internet y por consiguiente reduciendo el consumo de 
	ancho de banda.

	El parámetro de configuración está definido por la variable cache_dir que 
	cuenta con los siguientes parámetros

	cache_dir Type Directory-Name Mbytes L1 L2 

	Type: Especifica el tipo de sistema de almacenamiento para usar por defecto 
	es ufs

	Directory-Name: Especifica el nombre del directorio donde se almacenará el 
	cache

	Mbytes: Especifica el tamaño en megas del cache por defecto viene seteado en 
	100Mb

	L1: Cantidad de directorios en el primer nivel del cache por defecto viene 
	seteado en 16

	L2: Cantidad de directorios en el segundo nivel del cache por defecto viene 
	seteado en 256

	Para modificar esta opción de squid debemos tener en cuenta que si cambiamos 
	los parámetros L1 y L2 deberemos reconstruir el cache y por lo tanto 
	perderemos todo lo cacheado hasta el momento.

	Algunos ejemplos solo cambiar los valores remarcados:	

	a)  Agrandar el cache a 5Gb aproximadamente

	cache_dir ufs /var/cache/squid 5000 16 256

	Luego para aplicar los cambios ejecutar el comando

	#squid -k reconfigure


	b) Agrandar el cache a 5Gb aproximadamente y la canditad de directorios del 
	primer nivel

	cache_dir ufs /var/cache/squid 5000 32 256

	Luego para aplicar los cambios ejecutar los siguientes comandos

	Detenemos el proxy
	#/etc/init.d/squid stop

	Borramos el cache actual. ATENCION: el nombre del direcotirio a borrar es el 
	que se encuentra definido en el parámetro cache_dir.
	#rm -rf /var/cache/squid

	Regeneramos el cache
	#squid -z

	Reiniciamos el proxy
	#/etc/init.d/squid start

	NOTA: El tamaño de cache a seleccionar dependerá del espacio que el 
	administrador de red considere necesario en función de los recursos 
	disponibles y de la infraestructura con la que cuenta. Para un cache mayor a 
	2Gb es conveniente setear el parámetro L1 en 32

	2.2. Cambio en el tamaño de cache de memoria

	Este parámetro sirve para definir la cantidad de memoria RAM que se le 
	asignara a squid para mantener los objetos en memoria, No es la memoria 
	asignada al proceso

	El parámetro de configuración está definido por la variable cache_mem que 
	cuenta con los siguientes parámetros

	cache_mem bytes

	bytes: Especifica la cantidad de memoria ram asignada para mantener objetos 
	en cache de memoria, por defecto viene configurado en 256 Mb

	Algunos ejemplos solo cambiar los valores remarcados:	

	cache_mem 512 Mb

	Luego para aplicar los cambios ejecutar el comando

	#squid -k reconfigure

	NOTA: El tamaño de cache a seleccionar dependerá de la memoria libre con la 
	que cuente el servidor el administrador de red deberá optar por el valor 
	necesario en función de los recursos disponibles y de la infraestructura con 
	la que cuenta

2.3. Cambio en el tamaño máximo de objeto cacheado

	Este parámetro sirve para asignar el tamaño máximo de los archivos a guardar 
	en la cache.

	El parámetro de configuración está definido por la variable 
	maximun_object_size que cuenta con los siguientes parámetros

	maximun_object_size bytes

	bytes: Es el tamaño máximo de los archivos que se pueden guardar en cache, 
	por defecto el valor es de 4Mb

	Algunos ejemplos solo cambiar los valores remarcados:

	maximun_object_size 200 Mb

	Con un valor de 200 Mb nos aseguramos que archivos de hasta 200 Mb de tamaño 
	se van a poder almacenar en el cache como por ejemplo archivos de patrones 
	de detección de virus de los Antivirus, si muchos usuario tienen instalado 
	el mismo antivirus solo uno lo descargará de Internet el resto lo bajará 
	del cache optimizando el uso de ancho de banda.

	Luego para aplicar los cambios ejecutar el comando

	#squid -k reconfigure

2.4. Cacheado Windows Update

	Para cachear las actualizaciones del sistema operativo Windows debemos 
	setear los siguientes parámetros.

	range_offset_limit -1
	maximum_object_size 200 MB
	quick_abort_min -1

	refresh_pattern -i microsoft.com/.*\.(cab|exe|ms[i|u|f]|asf|wm[v|a]|dat|zip) 
	4320 80% 43200 reload-into-ims

	refresh_pattern -i 
	windowsupdate.com/.*\.(cab|exe|ms[i|u|f]|asf|wm[v|a]|dat|zip) 4320 80% 
	43200 reload-into-ims

	refresh_pattern \^ftp:        		1440	20%	10080
	refresh_pattern \^gopher:        	1440	0%	1440
	refresh_pattern -i (/cgi-bin/|\?) 	0		0%	0
	refresh_pattern .					0		20%	4320

	Luego para aplicar los cambios ejecutar el comando

	#squid -k reconfigure

2.5. Control de ancho de banda para videos

	Para controlar el ancho de banda que consumen la descarga de videos, 
	independientemente del dominio de descarga, utilizaremos delay pools basado 
	en la identificación del archivo de descarga a través de cabeceras MIME.

	Como primer paso debemos crear un archivo que contendrá las cabeceras MIME 
	que queremos analizar, llamaremos al archivo videos_mime el cual ubicaremos 
	dentro del directorio /etc/squid, utilizando cualquier editor de texto 
	agregaremos al mismo la siguiente lista de cabeceras MIME.

	Para crear el archivo podemos utilizar alguno e los siguientes comandos:

	#nano /etc/squid/videos_mime

	o

	#vi /etc/squid/videos_mime

	Luego pegamos el siguiente contenido. NOTA: La lista de cabeceras MIME 
	detallada a continuación es totalmente descriptiva ya que los 
	Administradores de Red deberán adaptarla a las necesidades de las 
	Establecimientos Educativos.

	application/x-videolan
	video/3gpp
	video/annodex
	video/dl
	video/dv
	video/fli
	video/flv
	video/gl
	video/mp4
	video/mp4v-es
	video/mpeg
	video/ogg
	video/parityfec
	video/pointer
	video/quicktime
	video/x-ms-wmx
	video/x-ms-wvx
	video/x-sgi-movie
	video/vnd.fvt
	video/vnd.motorola.video
	video/vnd.motorola.videop
	video/vnd.mpegurl
	video/vnd.mts
	video/vnd.nokia.interleaved-multimedia
	video/vnd.vivo
	video/x-flv
	video/x-la-asf
	video/x-matroska
	video/x-mng
	video/x-ms-asf
	video/x-ms-asx
	video/x-msvideo
	video/x-ms-wm
	video/x-ms-wmv

	Procederemos a continuación en la definición del delay pool para controlar 
	el ancho de banda utilizado en la descarga de videos, para ello debemos 
	editar el archivo /etc/squid/squid.conf y agregar las siguientes lineas.

	Creamos la delay pool (1 es el la cantidad de delay pools a definir)

	delay_pools 1

	Definimos la clase del de delay pool en este caso clase 3 para el delay pool 
	número uno. (delay pools individuales)

	delay_class 1 3

	Definimos el ancho de banda que aplicará este delay pool (delay pool número 
	uno). En este caso permitimos una descarga a máxima velocidad de los 
	primeros 1,7 Mb y luego limitamos la descarga del resto del video en 45 Kb. 
	Estos valores los debe adaptar el Administrador de Red según los recursos 
	con los que cuente.

	delay_parameters 1 -1/-1 -1/-1 46080/1782579

	Definimos al ACL que implementaremos con el delay pool que definimos 
	anteriormente.

	acl formatos_video req_mime_type -i "/etc/squid/videos_mime"

	Aplicamos a la ACL el delay pool que definimos (delay pool número uno)

	delay_access 1 allow videos_mime

	Luego para aplicar los cambios ejecutar el comando

	#squid -k reconfigure

2.6. Control de ancho de banda por puesto de trabajo

	Para controlar el ancho de banda que consumen cada puesto de trabajo 
	implementaremos el siguiete  delay pools en el archivo /etc/squid/squid.conf 
	y agregar las siguientes lineas. Si ya tenemos implementados delay pools 
	debemos aumentar la cantidad de delay pools definidos. En nuestro caso ya 
	contamos con el dalay pool para el control de ancho de banda de video, por 
	lo tanto cambiamos el contenido de la variable delay_pools. 

	delay_pools 2

	Definimos la clase del de delay pool en este caso clase 3 (delay pools 
	individuales para el delay poll identificado como dos)

	delay_class 2 3

	Definimos el ancho de banda que aplicará este delay pool. En este caso 
	permitimos una descarga a máxima velocidad de los primeros 2 Mb y luego 
	limitamos el resto de la descarga 10 Kb. Estos valores los debe adaptar el 
	Administrador de Red según los recursos con los que cuente.

	delay_parameters 2 -1/-1 -1/-1 10240/2097152

	Definimos al ACL que implementaremos con el delay pool que definimos 
	anteriormente, como queremos que cada puesto de trabajo quede limitado en 
	el ancho de banda que pueda utilizar, debemos aplicar el delay pool a toda 
	la red interna, Squid ya debe tener la ACL correspondiente al la red local 
	normal mente definida como “localnet” la cual debe tener el siguiente 
	formato, no hace falta definir una nueva acl si la misma no coincide con el 
	ejemplo mostrado.

	acl localnet src 172.16.0.0/12

	Aplicamos a la ACL el delay pool que definimos

	delay_access 2 allow localnet

	Luego para aplicar los cambios ejecutar el comando

	#squid -k reconfigure

	NOTA: El orden en que se definen los “delay_access” dentro del archivo de 
	configuración de squid, es el orden en que se aplican.
	

2.7. Bloqueo TOR Browser 

	ES NECESARIO TENER IMPLEMENTADO LA VERSION DE FIREWALL 3.1

	Para bloquear el Browser TOR debemos realizar los siguientes pasos:

	Ir al directorio de configuración del squid

	# cd /etc/squid

	Editar el achivo de configuración del squid

	# nano squid.conf

	Ahora debemos crear las ACLs correspondientes para que funcione el bloqueo. 
	Verificar que la acl que vamos a definir no exista.

	acl ips_habilitadas url_regex "/etc/squid/ips_habilitadas"
	acl redes_habilitadas dst "/etc/squid/redes_habilitadas"
	acl torweb dstdom_regex torproject.org
	acl direccion_ip url_regex 
	^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})

	Una vez definidas las ACLs debemos activarlas. Tener en cuenta que las 
	reglas de activación de las ACLs se ejecutan en el orden que están definidas 
	por lo tanto estas regla deberían ser unas de las primeras. Es decir las 
	reglas a continuación deben colocarse luego del texto “INSERT YOUR OWN 
	RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS”

	http_access deny torweb
	http_access allow ips_habilitadas
	http_access allow redes_habilitadas
	http_access deny  direccion_ip 

	Cerramos la configuración del squid y recargamos su configuración para que 
	se implementen las reglas definidas.

	#squid -k reconfigure

2.8. Buscando información para el cache

	Las configuración del cache dependen de las necesidades que tiene cada 
	Establecimiento Educativo, por lo tanto cada Administrador de Red deberá 
	parametrizar el proxy de acuerdo a las necesidades. Para ello una buena 
	práctica de búsqueda de información es accediendo a la página oficial de 
	Squid , donde se encuentra la wiki correspondiente a preguntas y respuestas.

				http://wiki.squid-cache.org/SquidFaq


