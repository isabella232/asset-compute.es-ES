---
title: Comprender la ampliación [!DNL Asset Compute Service].
description: Cuándo y cómo ampliar [!DNL Asset Compute Service] la funcionalidad para realizar el procesamiento personalizado de recursos.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '275'
ht-degree: 4%

---


# Introducción a la extensibilidad {#introduction-to-extensibilty}

Muchos de los requisitos de representación, como convertir a formatos y cambiar el tamaño de las imágenes, son cumplidos por los Perfiles [de procesamiento [!DNL Experience Manager] como Cloud Service](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/asset-microservices-overview.html). Los requerimientos comerciales más complejos pueden necesitar una solución creada a medida que se adapte a las necesidades de una organización. [!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas a las que se llama desde Perfiles de procesamiento en [!DNL Experience Manager]. Estas aplicaciones personalizadas se ocupan de los casos [de uso](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)admitidos.

>[!NOTE]
>
>[!DNL Asset Compute Service] sólo está disponible para su uso con [!DNL Experience Manager] Cloud Service.

Las aplicaciones personalizadas son aplicaciones [de Project Firefly](https://github.com/AdobeDocs/project-firefly) sin encabezado. La ampliación [!DNL Asset Compute Service] con aplicaciones personalizadas se realiza de forma sencilla a través del SDK [de](https://github.com/adobe/asset-compute-sdk) Asset Compute y la herramienta para desarrolladores de Project Firefly. Esto permite a los desarrolladores centrarse en la lógica empresarial. Crear aplicaciones personalizadas es tan sencillo como crear una acción de Adobe I/O Runtime sin servidor. Es una única función JavaScript de Node.js. El ejemplo [](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) básico de la aplicación personalizada lo ilustra.

## Requisitos previos y requisitos de aprovisionamiento {#prerequisites-and-provisioning}

Asegúrese de cumplir los siguientes requisitos previos:

* Las herramientas de Project Firefly están instaladas en su equipo.
* Una [!DNL Experience Cloud] organización. Más información [aquí](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* La organización de experiencias debe tener [!DNL Experience Manager] como Cloud Service habilitado.
* [!DNL Adobe Experience Cloud] forma parte del programa de previsualización para [!DNL Project Firefly] desarrolladores. Consulte [cómo solicitar acceso](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* Asegúrese de que el programador tenga permisos de administrador o función de desarrollador en la organización.
* Asegúrese de que la CLI [de E/S de](https://github.com/adobe/aio-cli) Adobe esté instalada localmente.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
