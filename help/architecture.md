---
title: Arquitectura de [!DNL Asset Compute Service].
description: Cómo [!DNL Asset Compute Service] API, aplicaciones y SDK funcionan juntos para proporcionar un servicio de procesamiento de recursos nativo de la nube.
translation-type: tm+mt
source-git-commit: c392b8588929f7b13db13e42a3f17bbc4f68a376
workflow-type: tm+mt
source-wordcount: '494'
ht-degree: 0%

---


# Arquitectura de [!DNL Asset Compute Service] {#overview}

El [!DNL Asset Compute Service] está construido sobre la plataforma Adobe I/O Runtime sin servidor. Proporciona compatibilidad con los servicios de contenido de Adobe Sensei para los recursos. El cliente que invoca (solo se admite [!DNL Experience Manager] como [!DNL Cloud Service]) recibe la información generada por Adobe Sensei que buscó para el recurso. La información devuelta está en formato JSON.

[!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas basadas en  [!DNL Project Firefly]. Estas aplicaciones personalizadas son [!DNL Project Firefly] aplicaciones sin encabezado y realizan tareas como agregar herramientas de conversión personalizadas o llamar a API externas para realizar operaciones de imagen.

[!DNL Project Firefly] es un marco para crear e implementar aplicaciones web personalizadas en  [!DNL Adobe I/O] tiempo de ejecución. Para crear aplicaciones personalizadas, los desarrolladores pueden aprovechar [!DNL React Spectrum] (el kit de herramientas de la interfaz de usuario de Adobe), crear microservicios, crear eventos personalizados y orquestar API. Consulte [documentación de Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

La base en la que se basa la arquitectura es:

* La modularidad de las aplicaciones, que sólo contienen lo necesario para una tarea determinada, permite desacoplar las aplicaciones entre sí y mantenerlas ligeras.

* El concepto sin servidor de Adobe I/O Runtime ofrece numerosos beneficios: el procesamiento asincrónico, altamente escalable, aislado y basado en trabajos, que es perfecto para el procesamiento de recursos.

* El almacenamiento de nube binaria proporciona las funciones necesarias para almacenar y acceder a archivos y representaciones de recursos de forma individual, sin necesidad de permisos de acceso total al almacenamiento, mediante referencias de URL prefirmadas. La aceleración de transferencia, el almacenamiento en caché de CDN y la ubicación conjunta de aplicaciones de cómputo con almacenamiento en la nube permiten un acceso óptimo al contenido de baja latencia. Se admiten las nubes AWS y Azure.

![Arquitectura del servicio de Asset compute](assets/architecture-diagram.png)

*Figura: Arquitectura  [!DNL Asset Compute Service] y forma en que se integra con la aplicación de procesamiento, almacenamiento  [!DNL Experience Manager]y procesamiento.*

La arquitectura consta de las siguientes partes:

* **Una** capa de orquestación y API recibe solicitudes (en formato JSON) que indican al servicio que transforme un recurso de origen en varias representaciones. Estas solicitudes son asincrónicas y se devuelven con un identificador de activación (también conocido como &quot;id. de trabajo&quot;). Las instrucciones son puramente declarativas y, para todo el trabajo de procesamiento estándar (por ejemplo, la generación de miniaturas, la extracción de texto), los consumidores solo especifican el resultado deseado, pero no las aplicaciones que gestionan determinadas representaciones. Las funciones genéricas de API, como autenticación, análisis, limitación de velocidad, se gestionan mediante la puerta de enlace de API de Adobe delante del servicio y administran todas las solicitudes que van al motor de ejecución de E/S. El enrutamiento de la aplicación lo realiza dinámicamente la capa de orquestación. Los clientes pueden especificar la aplicación personalizada para representaciones específicas e incluir parámetros personalizados. La ejecución de la aplicación se puede paralizar completamente, ya que son funciones independientes sin servidor en el tiempo de ejecución de E/S.

* **Aplicaciones para procesar** recursos especializados en determinados tipos de formatos de archivo o representaciones de destinatario. Conceptualmente, una aplicación es como el concepto de tubería Unix: un archivo de entrada se transforma en uno o varios archivos de salida.

* **Una  [biblioteca de aplicaciones común ](https://github.com/adobe/asset-compute-sdk)** gestiona tareas comunes como descargar el archivo de origen, cargar las representaciones, el sistema de informes de errores, el envío y la supervisión de eventos. Esto está diseñado para que el desarrollo de una aplicación siga siendo lo más sencillo posible, siguiendo la idea sin servidor, y se pueda restringir a las interacciones locales del sistema de archivos.

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
