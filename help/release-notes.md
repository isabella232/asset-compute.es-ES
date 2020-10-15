---
title: Notas de la versión de [!DNL Asset Compute Service].
description: Nuevas funciones, mejoras y problemas conocidos en [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
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

## What is new {#what-is-new}

Esta es la primera versión de [!DNL Asset Compute Service]. Es un servicio ampliable y ampliable de [!DNL Adobe Experience Cloud] procesamiento de recursos digitales. Puede transformar formatos de imagen, vídeo, documento y otros archivos en distintas representaciones, incluidas miniaturas, texto extraído y metadatos, y archivos.

Actualmente, el [!DNL Asset Compute Service] solo se puede usar [!DNL Experience Manager] como Cloud Service.

## Limitaciones y problemas conocidos {#known-limitations}

Para probar la aplicación personalizada con la herramienta [para](https://github.com/adobe/asset-compute-devtool)desarrolladores, necesita acceder a un contenedor [de almacenamiento de](https://github.com/adobe/asset-compute-devtool#prerequisites)nube.

* El acceso al almacenamiento en la nube (distinto del almacén de [!DNL Experience Manager] blob) solo es necesario para la herramienta para desarrolladores. Aún puede crear, probar e implementar aplicaciones personalizadas sin la herramienta para desarrolladores.
* Puede ser un contenedor compartido utilizado por varios desarrolladores en diferentes proyectos.

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] la extensibilidad se desarrolla bajo un modelo de desarrollo abierto en [github.com/adobe](https://github.com/adobe) que acoge con satisfacción las contribuciones de los desarrolladores de extensiones. Todos los componentes relevantes para desarrollar, crear, probar e implementar aplicaciones personalizadas son de código abierto. Consulte [cómo y dónde contribuir al servicio](contribute-to-compute-service.md)de cómputo.

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
