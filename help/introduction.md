---
title: Introducción a [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] es un servicio de procesamiento de recursos nativo de la nube que reduce la complejidad y mejora la escalabilidad".'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 6%

---

# Información general sobre [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] es un servicio ampliable y escalable de [!DNL Adobe Experience Cloud] para procesar recursos digitales. Puede transformar formatos de imagen, vídeo, documento y otros archivos en distintas representaciones, incluidas miniaturas, texto extraído y metadatos, y archivos.

Los desarrolladores pueden agregar aplicaciones de recursos personalizados (también denominadas trabajadores personalizados) para tratar casos de uso personalizados. El servicio funciona en la variable [!DNL Adobe I/O] tiempo de ejecución. Se puede ampliar mediante [!DNL Adobe Developer App Builder] aplicaciones sin encabezado escritas en Node.js. Pueden realizar operaciones personalizadas, como llamar a API externas para realizar operaciones de imagen o aprovechar [!DNL Adobe Sensei] asistencia técnica.

[!DNL Adobe Developer App Builder] es un marco para crear e implementar aplicaciones web personalizadas en [!DNL Adobe I/O] tiempo de ejecución para ampliar las soluciones de Adobe Experience Cloud. Para crear aplicaciones personalizadas, los desarrolladores pueden aprovechar [!DNL React Spectrum] (Kit de herramientas de la interfaz de Adobe), cree microservicios, cree eventos personalizados y organice API. Consulte [documentación de Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>Actualmente, la variable [!DNL Asset Compute Service] solo se puede usar mediante [!DNL Experience Manager] como [!DNL Cloud Service]. Los administradores crean perfiles de procesamiento que pueden llamar a la función [!DNL Asset Compute Service] para pasar recursos para su procesamiento. Consulte [usar microservicios de recursos y perfiles de procesamiento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=es).

## Casos de uso admitidos de [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] admite algunos casos de uso empresarial comunes, como el procesamiento básico de imágenes; conversiones específicas de la aplicación de Adobe; y la creación de aplicaciones personalizadas que orquesten requisitos empresariales complejos.

Puede usar [!DNL Asset Compute] servicio web para generar miniaturas para diferentes tipos de archivo y renderizaciones de imágenes de alta calidad para el [formatos de archivo compatibles](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Consulte [casos de uso admitidos mediante configuración personalizada](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=es).

>[!NOTE]
>
>El servicio no proporciona almacenamiento de recursos. Los usuarios la proporcionan y proporcionan referencias a ubicaciones de archivos de origen y representación en el almacenamiento en la nube.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Resumen del procesamiento de recursos con microservicios de recursos en [!DNL Adobe Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=es).
>* [Documentación de Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Formatos de archivo compatibles con el procesamiento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
