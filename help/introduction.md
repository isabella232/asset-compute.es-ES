---
title: Introducción a las [!DNL Asset Compute Service].
description: '[!DNL Asset Compute Service] es un servicio de procesamiento de recursos nativo de la nube que reduce la complejidad y mejora la escalabilidad.'
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '332'
ht-degree: 2%

---


# Información general de [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] es un servicio ampliable y ampliable de [!DNL Adobe Experience Cloud] para procesar recursos digitales. Puede transformar formatos de imagen, vídeo, documento y otros archivos en distintas representaciones, incluidas miniaturas, texto extraído y metadatos, y archivos.

Los desarrolladores pueden plugin aplicaciones de recursos personalizados (también denominadas trabajadores personalizados) para tratar casos de uso personalizados. El servicio funciona en tiempo de ejecución [!DNL Adobe I/O] . Se puede ampliar mediante aplicaciones [!DNL Project Firefly] sin encabezado escritas en Node.js. Pueden realizar operaciones personalizadas, como llamar a API externas para realizar operaciones de imagen o aprovechar la [!DNL Adobe Sensei] compatibilidad.

[!DNL Project Firefly] es un marco para crear e implementar aplicaciones web personalizadas en tiempo de ejecución para ampliar las [!DNL Adobe I/O] soluciones de Adobe Experience Cloud. Para crear aplicaciones personalizadas, los desarrolladores pueden aprovechar [!DNL React Spectrum] (kit de herramientas de la interfaz de usuario de Adobe), crear microservicios, crear eventos personalizados y orquestar API. Consulte la [documentación de Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).

>[!NOTE]
>
>Actualmente, el [!DNL Asset Compute Service] solo se puede usar [!DNL Experience Manager] como Cloud Service. Los administradores crean perfiles de procesamiento que pueden llamar a [!DNL Asset Compute Service] para pasar recursos para su procesamiento. See [use asset microservices and processing profiles](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Casos de uso admitidos de [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] admite algunos casos comunes de uso empresarial, como el procesamiento de imágenes básicas; conversiones específicas de la aplicación de Adobe; y la creación de aplicaciones personalizadas que orquesten requisitos comerciales complejos.

Puede utilizar [!DNL Asset Compute] el servicio Web para generar miniaturas para diferentes tipos de archivos y representaciones de imágenes de alta calidad para los formatos [de archivo](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html)admitidos. Consulte casos [de uso admitidos mediante la configuración](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html#custom-config)personalizada.

>[!NOTE]
>
>El servicio no proporciona almacenamiento de recursos. Los usuarios lo proporcionan y proporcionan referencias a las ubicaciones de archivos de origen y representación en el almacenamiento en la nube.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use Adobe I/O Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Información general sobre el procesamiento de recursos con microservicios de recursos en Adobe Experience Manager como Cloud Service](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Documentación del Proyecto Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html).
>* [Formatos de archivo compatibles para su procesamiento](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html).
>* [Notas de la versión del servicio de cómputo de recursos](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->