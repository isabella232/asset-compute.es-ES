---
title: Obtenga información sobre la extensión [!DNL Asset Compute Service].
description: Cuándo y cómo ampliar la funcionalidad [!DNL Asset Compute Service] para realizar el procesamiento personalizado de recursos.
translation-type: tm+mt
source-git-commit: c392b8588929f7b13db13e42a3f17bbc4f68a376
workflow-type: tm+mt
source-wordcount: '265'
ht-degree: 1%

---


# Introducción a la extensibilidad {#introduction-to-extensibilty}

Muchos requisitos de representación, como convertir a formatos y cambiar el tamaño de las imágenes, se resuelven mediante [Perfiles de procesamiento en [!DNL Experience Manager] como a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). Los requerimientos comerciales más complejos pueden necesitar una solución creada a medida que se adapte a las necesidades de una organización. [!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas a las que se llama desde Perfiles de procesamiento en  [!DNL Experience Manager]. Estas aplicaciones personalizadas se ocupan de los [casos de uso admitidos](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] sólo está disponible para su uso  [!DNL Experience Manager] como  [!DNL Cloud Service].

Las aplicaciones personalizadas son aplicaciones [Project Firefly](https://github.com/AdobeDocs/project-firefly) sin encabezado. La ampliación de [!DNL Asset Compute Service] con aplicaciones personalizadas se hace sencilla mediante las herramientas para desarrolladores de [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk) y Project Firefly. Esto permite a los desarrolladores centrarse en la lógica empresarial. Crear aplicaciones personalizadas es tan sencillo como crear una acción de Adobe I/O Runtime sin servidor. Es una única función JavaScript de Node.js. El [ejemplo de aplicación personalizada básica](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo ilustra.

## Requisitos previos y requisitos de aprovisionamiento {#prerequisites-and-provisioning}

Asegúrese de cumplir los siguientes requisitos previos:

* Las herramientas de Project Firefly están instaladas en su equipo.
* Una organización [!DNL Experience Cloud]. Más información [aquí](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials).
* La organización de experiencias debe tener [!DNL Experience Manager] habilitada como [!DNL Cloud Service].
* [!DNL Adobe Experience Cloud] forma parte del programa de previsualización para  [!DNL Project Firefly] desarrolladores. Consulte [cómo solicitar acceso](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md).
* Asegúrese de que el programador tenga permisos de administrador o función de desarrollador en la organización.
* Asegúrese de que [Adobe I/O CLI](https://github.com/adobe/aio-cli) se instale localmente.

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
