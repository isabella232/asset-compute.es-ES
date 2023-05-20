---
title: Ampliación de [!DNL Asset Compute Service]
description: Cuándo y cómo ampliar el [!DNL Asset Compute Service] para realizar el procesamiento de recursos personalizado.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 6%

---

# Introducción a la extensibilidad {#introduction-to-extensibilty}

Muchos de los requisitos de representación, como la conversión a formatos y el cambio de tamaño de imágenes, los aborda [Procesamiento de perfiles en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=es). Los requisitos empresariales más complejos pueden necesitar una solución personalizada que se adapte a las necesidades de una organización. [!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas a las que se llama desde Perfiles de procesamiento en [!DNL Experience Manager]. Estas aplicaciones personalizadas se adaptan a [casos de uso admitidos](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=es).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con [!DNL Experience Manager] as a [!DNL Cloud Service].

Las aplicaciones personalizadas no tienen encabezado [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) aplicaciones. Ampliación [!DNL Asset Compute Service] con aplicaciones personalizadas se simplifica a través de la [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) y las herramientas para desarrolladores de Adobe Developer App Builder. Esto permite a los desarrolladores centrarse en la lógica empresarial. Crear aplicaciones personalizadas es tan sencillo como crear un servidor sin formato [!DNL Adobe I/O] Acción de tiempo de ejecución. Es una función JavaScript de Node.js única. El [ejemplo básico de aplicación personalizada](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo ilustra.

## Requisitos previos y requisitos de aprovisionamiento {#prerequisites-and-provisioning}

Asegúrese de cumplir los siguientes requisitos previos:

* Las herramientas del Generador de aplicaciones de Adobe Developer están instaladas en el equipo.
* Un [!DNL Experience Cloud] organización. Más información [aquí](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* La organización de experiencia debe tener [!DNL Experience Manager] as a [!DNL Cloud Service] activado.
* [!DNL Adobe Experience Cloud] La organización forma parte de [!DNL Adobe Developer App Builder] programa de vista previa para desarrolladores. Consulte [cómo solicitar acceso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Asegúrese de tener una función de desarrollador o permisos de administrador en la organización para el desarrollador.
* Asegúrese de que [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) se instala localmente.

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
