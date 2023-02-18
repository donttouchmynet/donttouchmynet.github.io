---
title: "Protegiendo los eventos de CloudTrail"
author: Adan Alvarez
classes: wide
excerpt: "En esta entrada vamos a ver la importancia de CloudTrail, como necesitamos proteger todos los servicios a los cuales se envían los eventos y como AWSTrailGuard puede ayudarnos."
categories:
  - AWS
tags:
  - Blue Team
  - Defense Evasion
---

En esta entrada vamos a hablar de [CloudTrail](https://docs.aws.amazon.com/cloudtrail/). Cloudtrail es sin duda una de las herramientas de seguridad más importantes que ofrece AWS, 
ya que registra todos los eventos en la cuenta de AWS. Los eventos incluyen las acciones llevadas a cabo en la AWS Management Console, 
AWS Command Line Interface y las API y los SDK de AWS.
{: style="text-align: justify;"}

Los registros de CloudTrail pueden ayudar a los administradores de la cuenta a detectar actividades sospechosas. 
En muchos casos, estos eventos son enviados o recogidos por un SIEM en el cual se configurarán alertas de seguridad. 
Por esta razón, deshabilitar CloudTrail de algún modo es una de las tácticas de evasión utilizadas por los adversarios. 
Con Cloudtrail fuera de juego, un adversario podrá realizar acciones que generarían alertas en condiciones normales, como ganar persistencia creando nuevos usuarios, 
enumerar la cuenta con múltiples llamadas a las API o eliminar datos o servicios importantes.
{: style="text-align: justify;"}

Existen formas de deshabilitar CloudTrail de forma silenciosa, por ejemplo, [añadiendo un event selector](https://github.com/RhinoSecurityLabs/Cloud-Security-Research/tree/master/AWS/cloudtrail_guardduty_bypass). 
Sin embargo, debido a la preocupación legítima de que un atacante pueda desactivar o modificar CloudTrail para ocultar sus actividades maliciosas, 
los cambios en la configuración de CloudTrail suelen estar bien protegidos, o al menos monitorizados.
{: style="text-align: justify;"}

El problema es que en la mayoría de los casos, los registros de CloudTrail no se consumen directamente, 
sino que se utilizan otros servicios para procesarlos y analizarlos. Por ejemplo, los registros de CloudTrail pueden enviarse a un bucket de Amazon S3, 
que genere una notificación de evento de S3, que se envíe a una cola de [Amazon SQS](https://docs.aws.amazon.com/sqs/index.html), y luego se analicen los registros tras consultar la cola de SQS. 
Este proceso que involucra varios servicios de AWS, lo hace más susceptible a la manipulación por parte de un atacante.
{: style="text-align: justify;"}

Si un atacante pudiera modificar un servicio al que se envían los registros de CloudTrail, 
este podría manipularlo de tal forma que se ocultaran sus actividades maliciosas. 
Por ejemplo, un atacante podría cambiar la configuración de la cola SQS para que el sistema de alerta o SIEM, que consulta esta cola, no vea que hay nuevos registros a procesar y, 
por lo tanto, no lea estos del S3. 
{: style="text-align: justify;"}

Otro ejemplo sería como un atacante podría añadir una [Lifecycle Rule en el S3](https://stratus-red-team.cloud/attack-techniques/AWS/aws.defense-evasion.cloudtrail-lifecycle-rule/) para que los eventos se eliminen pasado un corto periodo de tiempo. 
{: style="text-align: justify;"}

Es por esto que es importante supervisar todos los servicios a los que se envían los registros de CloudTrail
y estar alerta ante cualquier cambio en la configuración de estos.
{: style="text-align: justify;"}

Para hacer esta tarea algo más sencilla, hemos creado [AWSTrailGuard](https://github.com/adanalvarez/AWSTrailGuard). AWSTrailGuard es una herramienta de línea de comandos que nos permite visualizar los servicios a los que se envían los eventos de CloudTrail. 
La herramienta utiliza la API de AWS para obtener información sobre CloudTrail, con esta información nos indicará a qué S3 se envían los registros y si también se envían a CloudWatch. 
Para S3 verificará si hay configuradas “Event notifications” y para CloudWatch si hay configurados “Subscription filters”.
{: style="text-align: justify;"}

Esta información aparecerá por consola, además de información sobre posibles comandos que podrían afectar a los servicios detectados, 
tal y como se muestra en la siguiente captura:
{: style="text-align: justify;"}

![ConsoleOutput](https://github.com/adanalvarez/AWSTrailGuard/blob/main/consoleoutput.png?raw=true)

Por otro lado, AWSTrailGuard también generará un diagrama en formato DOT que podremos visualizar como PNG de forma sencilla utilizando [Graphviz](https://graphviz.org/) mediante el siguiente comando:
{: style="text-align: justify;"}

```dot -T png cloudtrail.dot > cloudtrail.png```

Este diagrama nos ayuda a entender la estructura de los servicios involucrados. Este es un ejemplo de diagrama:
{: style="text-align: justify;"}

![GraphVisualization](https://github.com/adanalvarez/AWSTrailGuard/blob/main/graphvisualization.png?raw=true)

Con esta información, ahora nos tocará asegurar que todos los servicios involucrados están bien protegidos y monitorizados. 
{: style="text-align: justify;"}
