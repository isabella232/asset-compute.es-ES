---
title: Arquitectura de [!DNL Asset Compute Service]
description: Cómo [!DNL Asset Compute Service] la API, las aplicaciones y el SDK trabajan juntos para proporcionar un servicio de procesamiento de recursos nativo de la nube.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '485'
ht-degree: 0%

---

# Arquitectura de [!DNL Asset Compute Service] {#overview}

El [!DNL Asset Compute Service] está construido sobre la plataforma [!DNL Adobe I/O] Runtime sin servidor. Proporciona compatibilidad con los servicios de contenido de Adobe Sensei para los recursos. El cliente que invoca (solo se admite [!DNL Experience Manager] como [!DNL Cloud Service]) recibe la información generada por Adobe Sensei que buscó para el recurso. La información devuelta está en formato JSON.

[!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas basadas en  [!DNL Project Firefly]. Estas aplicaciones personalizadas son [!DNL Project Firefly] aplicaciones sin encabezado y realizan tareas como agregar herramientas de conversión personalizadas o llamar a API externas para realizar operaciones de imágenes.

[!DNL Project Firefly] es un marco para crear e implementar aplicaciones web personalizadas en  [!DNL Adobe I/O] tiempo de ejecución. Para crear aplicaciones personalizadas, los desarrolladores pueden aprovechar [!DNL React Spectrum] (kit de herramientas de interfaz de Adobe), crear microservicios, crear eventos personalizados y organizar API. Consulte la [documentación del proyecto Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

Los fundamentos en los que se basa la arquitectura incluyen:

* La modularidad de las aplicaciones (que solo contienen lo necesario para una tarea determinada) permite desvincular las aplicaciones entre sí y mantenerlas ligeras.

* El concepto sin servidor de [!DNL Adobe I/O] Runtime ofrece numerosos beneficios: el procesamiento asincrónico, altamente escalable, aislado y basado en trabajos, que es perfecto para el procesamiento de recursos.

* El almacenamiento en la nube binaria proporciona las funciones necesarias para almacenar y acceder a los archivos y representaciones de recursos de forma individual, sin necesidad de permisos de acceso completo al almacenamiento, utilizando referencias de URL prefirmadas. La aceleración de transferencia, el almacenamiento en caché de CDN y la ubicación conjunta de aplicaciones de computación con almacenamiento en la nube permiten un acceso óptimo al contenido de baja latencia. Se admiten nubes de AWS y Azure.

![Arquitectura del servicio de Asset compute](assets/architecture-diagram.png)

*Figura: Arquitectura de  [!DNL Asset Compute Service] y cómo se integra con  [!DNL Experience Manager], almacenamiento y aplicación de procesamiento.*

La arquitectura consta de las siguientes partes:

* **Una API y una** capa de orquestación reciben solicitudes (en formato JSON) que indican al servicio que transforme un recurso de origen en varias representaciones. Las solicitudes son asíncronas y se devuelven con un ID de activación, es decir, un ID de trabajo. Las instrucciones son puramente declarativas y, para todo el trabajo de procesamiento estándar (por ejemplo, la generación de miniaturas o la extracción de texto), los consumidores solo especifican el resultado deseado, pero no las aplicaciones que administran determinadas representaciones. Las funciones genéricas de API, como la autenticación, el análisis, la limitación de velocidad, se gestionan mediante la puerta de enlace de la API de Adobe delante del servicio y administran todas las solicitudes que van a [!DNL Adobe I/O] Runtime. El enrutamiento de la aplicación se realiza dinámicamente mediante la capa de orquestación. Los clientes pueden especificar la aplicación personalizada para representaciones específicas e incluir parámetros personalizados. La ejecución de la aplicación se puede paralizar completamente, ya que son funciones separadas sin servidor en [!DNL Adobe I/O] Runtime.

* **Aplicaciones para procesar** recursos especializados en ciertos tipos de formatos de archivo o representaciones de destino. Conceptualmente, una aplicación es como el concepto de tubo Unix: un archivo de entrada se transforma en uno o más archivos de salida.

* **Una biblioteca de aplicaciones  [común ](https://github.com/adobe/asset-compute-sdk)** gestiona tareas comunes como descargar el archivo de origen, cargar las representaciones, los informes de errores, la entrega de eventos y la monitorización . Esto está diseñado para que el desarrollo de una aplicación siga siendo lo más sencillo posible, siguiendo la idea sin servidor, y puede restringirse a las interacciones del sistema de archivos local.

<!-- TBD:

* About the YAML file?
* See [https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
