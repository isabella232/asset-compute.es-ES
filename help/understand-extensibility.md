---
title: Comprender la ampliación [!DNL Asset Compute Service]
description: Cuándo y cómo ampliar la variable [!DNL Asset Compute Service] para realizar el procesamiento personalizado de recursos.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 6%

---

# Introducción a la extensibilidad {#introduction-to-extensibilty}

Muchos requisitos de representación, como la conversión a formatos y el cambio de tamaño de imágenes, se resuelven mediante [Procesamiento de perfiles en [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html?lang=es). Los requisitos comerciales más complejos pueden necesitar una solución personalizada que se adapte a las necesidades de una organización. [!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas a las que se llama desde Perfiles de procesamiento en [!DNL Experience Manager]. Estas aplicaciones personalizadas sirven para [casos de uso admitidos](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=es).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con [!DNL Experience Manager] como [!DNL Cloud Service].

Las aplicaciones personalizadas no tienen encabezado [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) aplicaciones. Ampliación [!DNL Asset Compute Service] con las aplicaciones personalizadas se hace simple a través del [SDK de asset compute](https://github.com/adobe/asset-compute-sdk) y herramientas para desarrolladores de Adobe Developer App Builder . Esto permite a los desarrolladores centrarse en la lógica empresarial. La creación de aplicaciones personalizadas es tan sencilla como crear sin servidor [!DNL Adobe I/O] Acción en tiempo de ejecución. Es una función JavaScript de Node.js única. La variable [ejemplo básico de aplicación personalizada](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo ilustra.

## Requisitos previos y requisitos de aprovisionamiento {#prerequisites-and-provisioning}

Asegúrese de cumplir los siguientes requisitos previos:

* Las herramientas de Adobe Developer App Builder están instaladas en el equipo.
* Un [!DNL Experience Cloud] organización. Más información [here](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* La organización de experiencias debe tener [!DNL Experience Manager] como [!DNL Cloud Service] activada.
* [!DNL Adobe Experience Cloud] forma parte de la [!DNL Adobe Developer App Builder] programa de vista previa para desarrolladores. Consulte [cómo solicitar acceso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Asegúrese de que haya un rol de desarrollador o permisos de administrador en la organización para el desarrollador.
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
