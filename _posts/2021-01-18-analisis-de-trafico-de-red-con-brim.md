---
title: "Análisis de tráfico de red con BRIM"
author: Adan Alvarez
classes: wide
excerpt: "Brim es una herramienta open source que nos permite realizar análisis de tráfico de red combinando lo mejor de las herramientas Zeek y Wireshark. Con Brim tenemos la visibilidad de Zeek pero manteniendo el detalle de Wireshark a un clic de distancia. Esto hace que podamos analizar capturas que con Wireshark serían muy lentas y difíciles de analizar con mucha rapidez."
categories:
  - Threat Hunting
  - Herramientas
tags:
  - Brim
  - Herramientas
  - Zeek
  - Network
  - Blue Team
---
[Brim](https://www.brimsecurity.com/) es una herramienta open source que nos permite realizar análisis de tráfico de red combinando lo mejor de las herramientas [Zeek](https://zeek.org/) y [Wireshark](https://www.wireshark.org/). Con Brim tenemos la visibilidad de Zeek pero manteniendo el detalle de Wireshark a un clic de distancia. Esto hace que podamos analizar capturas que con Wireshark serían muy lentas y difíciles de analizar con mucha rapidez.
{: style="text-align: justify;"}

En esta entrada veremos algunas de las funcionalidades que nos ofrece Brim para en próximas entradas utilizar esta herramienta para Threat Hunting.
{: style="text-align: justify;"}

La captura que vamos a utilizar para mostrar las funcionalidades es la siguiente: <https://www.malware-traffic-analysis.net/2020/09/02/2020-09-01-Emotet-epoch-3-infection-with-Trickbot-gtag-mor119.pcap.zip>
{: style="text-align: justify;"}

La captura está cifrada con la contraseña que aparece en [About](https://www.malware-traffic-analysis.net/about.html)
{: style="text-align: justify;"}

En primer lugar, podemos descargar Brim [aquí.](https://www.brimsecurity.com/download/)
{: style="text-align: justify;"}

La guía de instalación está [aquí.](https://github.com/brimsec/brim/wiki/Installation)
{: style="text-align: justify;"}

Una vez instalado, al ejecutarlo nos aparecerá la siguiente pantalla donde podemos seleccionar el archivo o simplemente arrastrarlo:
{: style="text-align: justify;"}

[![análisis de tráfico](https://donttouchmynet.github.io/assets/images/old/Inicio-300x180.png)](https://donttouchmynet.github.io/assets/images/old/Inicio.png)
{: style="text-align: justify;"}

Al cargar el fichero nos aparecerá la siguiente ventana:
{: style="text-align: justify;"}

[![análisis de tráfico](https://donttouchmynet.github.io/assets/images/old/Principal-300x138.png)](https://donttouchmynet.github.io/assets/images/old/Principal.png)

En esta podemos observar:
{: style="text-align: justify;"}

1.  En la parte superior tendremos, como si de un navegador se tratase, diferentes pestañas para diferentes capturas que estemos analizando.
2.  En la parte superior central tendremos varios botones de configuración que nos permitirán entre otras cosas cambiar la vista, exportar datos o en el caso de tener seleccionada una conexión ver la conexión en Wireshark.
3.  Debajo de esto tendremos una barra que nos permitirá realizar búsquedas utilizando el lenguaje [ZQL](https://github.com/brimsec/zq/tree/master/zql/docs).
4.  Debajo de esta barra de búsqueda tendremos el histograma con el que podremos interactuar seleccionando partes de este para de esta forma solo ver los datos durante un determinado periodo de tiempo. Además, al pasar el cursor por encima podremos ver un resumen de la actividad durante cada columna de tiempo.
5.  Bajo el histograma tenemos los datos de la captura de tráfico. Aquí no veremos cada uno de los paquetes sino que veremos los logs de Zeek y las alertas de [Emerging Threats OPEN ruleset](https://rules.emergingthreats.net/OPEN_download_instructions.html) de [Suricata](https://suricata-ids.org/). Tendremos un extenso conjunto de archivos de registro de la actividad de red a alto nivel. Estos registros incluyen no solo un registro completo de cada conexión, sino también transcripciones de la capa de aplicación, como, por ejemplo, todas las sesiones HTTP con sus URI solicitadas, encabezados de clave, MIME types y respuestas del servidor; Solicitudes de DNS con respuestas; Certificados SSL; contenido clave de las sesiones SMTP; y mucho más.
6.  En el lateral derecho, si tenemos activada esta vista (La podemos desactivar en el botón View del punto 2). Podremos ver en la parte superior los especio de trabajo. Aquí haciendo clic en + podremos añadir nuevas capturas.
7.  En la parte inferior tenemos consultas predefinidas muy útiles. Simplemente clicando en cada una de ellas la consulta se autocompletará en la barra del punto 3 y podremos ver los resultados. Estas consultas están preparadas para agilizar el análisis.
8.  Por último, debajo de las consultas predefinidas, tendremos un historial de todas las consultas realzadas para que podamos volver a una consulta anterior de forma rápida.
{: style="text-align: justify;"}

Veamos ahora diferentes ejemplos que nos mostrarán la potencia y la utilidad de Brim para el análisis de tráfico.
{: style="text-align: justify;"}

Si hacemos clic en la consulta predefinida: Unique DNS Queries veremos que la barra superior se completa con la consulta y vemos los resultados del siguiente modo:
{: style="text-align: justify;"}

[![análisis de tráfico](https://donttouchmynet.github.io/assets/images/old/DNS-requests-300x178.png)](https://donttouchmynet.github.io/assets/images/old/DNS-requests.png)

De este modo veremos todos los dominios que han sido consultados, si sospechamos de alguno de ellos podemos consultarlo de forma sencilla en Virtustotal haciendo clic con el botón derecho en el dominio y después VirusTotal Lookup.
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/VirusTotal-1024x744.png)](https://donttouchmynet.github.io/assets/images/old/VirusTotal.png)

Si queremos ver más detalles de una de estas consultas podemos hacier lick derecho, Pivot to logs y volveremos a la pantalla inicial con un filtro para esa la DNS específica:
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/pivottologs-1024x670.png)](https://donttouchmynet.github.io/assets/images/old/pivottologs.png)

[![análisis de tráfico](https://donttouchmynet.github.io/assets/images/old/consultaDNS-1024x304.png)](https://donttouchmynet.github.io/assets/images/old/consultaDNS.png)

Para ver el detalle del paquete podemos ver esta conexión en Wireshark de dos maneras. Seleccionando la conexión y haciendo click en el icono Packets en la parte superior:
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/whiresarkclicpackets-1024x316.png)](https://donttouchmynet.github.io/assets/images/old/whiresarkclicpackets.png)

o haciendo doble clic para ver toda la información y haciendo clic en el icono de Wireshark:
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/whiresarkclic2packets.png)](https://donttouchmynet.github.io/assets/images/old/whiresarkclic2packets.png)

Independientemente del método elegido se nos abrirá wireshak únicamente con esa conexión:
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/whireshark-1.png)](https://donttouchmynet.github.io/assets/images/old/whireshark-1.png)

Vamos a ver ahora la consulta predefinida File Activity
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/fileactivity-1024x392.png)](https://donttouchmynet.github.io/assets/images/old/fileactivity.png)

Aquí podemos ver los ficheros que aparecen en esta captura, podemos hacer doble clic en el fichero para ver más información.
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/file.png)](https://donttouchmynet.github.io/assets/images/old/file.png)

Dentro de la información que nos aparece tenemos el hash MD5 y SHA1 del fichero que podemos utilizar para hacer una consulta en VirusTotal:
{: style="text-align: justify;"}

[![](https://donttouchmynet.github.io/assets/images/old/lookupvirustotalfile.png)](https://donttouchmynet.github.io/assets/images/old/lookupvirustotalfile.png)

Como podéis observar Brim nos permite de forma muy rápida obtener información importante sobre la captura y realizar un análisis de tráfico a alto nivel permitiéndonos después ir a ver los detalles de la captura en wireshark cargando solo los datos necesarios para evitar la lentitud de wireshark al tratar con capturas muy grandes.
{: style="text-align: justify;"}