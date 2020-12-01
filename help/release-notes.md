---
title: Notas de la versión de [!DNL Asset Compute Service].
description: Nuevas funciones, mejoras y problemas conocidos en [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: c57867cd896e4ccb9402e6eeb0eea133faaa0e5d
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---


# Notas de la versión de [!DNL Asset Compute Service] {#release-notes}

La última versión de [!DNL Asset Compute Service] se publicó el 30 de julio de 2020.

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## Novedades {#what-is-new}

Esta es la primera versión de [!DNL Asset Compute Service]. Es un servicio escalable y extensible de [!DNL Adobe Experience Cloud] para procesar activos digitales. Puede transformar formatos de imagen, vídeo, documento y otros archivos en distintas representaciones, incluidas miniaturas, texto extraído y metadatos, y archivos.

Actualmente, el [!DNL Asset Compute Service] sólo se puede usar en [!DNL Experience Manager] como [!DNL Cloud Service].

## Limitaciones y problemas conocidos {#known-limitations}

Para probar la aplicación personalizada con la [herramienta para desarrolladores](https://github.com/adobe/asset-compute-devtool), necesita acceder a un [contenedor de almacenamiento en la nube](https://github.com/adobe/asset-compute-devtool#prerequisites).

* El acceso al almacenamiento en la nube (distinto del almacén de blobs [!DNL Experience Manager]) solo es necesario para la herramienta para desarrolladores. Aún puede crear, probar e implementar aplicaciones personalizadas sin la herramienta para desarrolladores.
* Puede ser un contenedor compartido utilizado por varios desarrolladores en diferentes proyectos.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] la extensibilidad se desarrolla bajo un modelo de desarrollo abierto en  [github.com/](https://github.com/adobe) adobeque da la bienvenida a las contribuciones de los desarrolladores de extensiones. Todos los componentes relevantes para desarrollar, crear, probar e implementar aplicaciones personalizadas son de código abierto. Consulte [cómo y dónde contribuir al servicio de cómputo](contribute-to-compute-service.md).

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
