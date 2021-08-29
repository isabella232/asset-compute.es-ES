---
title: Comprender cómo ampliar [!DNL Asset Compute Service]
description: Cuándo y cómo ampliar la funcionalidad [!DNL Asset Compute Service] para realizar un procesamiento de recursos personalizado.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '254'
ht-degree: 1%

---

# Introducción a la extensibilidad {#introduction-to-extensibilty}

[Processing Profiles in [!DNL Experience Manager] as a2/>](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html) tratan los requisitos de representación, como la conversión a formatos y el cambio de tamaño de imágenes.  [!DNL Cloud Service] Los requisitos comerciales más complejos pueden necesitar una solución personalizada que se adapte a las necesidades de una organización. [!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas a las que se llama desde Perfiles de procesamiento en  [!DNL Experience Manager]. Estas aplicaciones personalizadas se ocupan de los [casos de uso admitidos](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con  [!DNL Experience Manager] as a  [!DNL Cloud Service].

Las aplicaciones personalizadas son aplicaciones [Project Firefly](https://github.com/AdobeDocs/project-firefly) sin encabezado. La ampliación de [!DNL Asset Compute Service] con aplicaciones personalizadas se hace sencilla mediante las herramientas para desarrolladores de [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) y Project Firefly. Esto permite a los desarrolladores centrarse en la lógica empresarial. La creación de aplicaciones personalizadas es tan sencilla como crear una acción [!DNL Adobe I/O] Runtime sin servidor. Es una función JavaScript de Node.js única. El [ejemplo básico de la aplicación personalizada](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo ilustra.

## Requisitos previos y requisitos de aprovisionamiento {#prerequisites-and-provisioning}

Asegúrese de cumplir los siguientes requisitos previos:

* Las herramientas de Project Firefly están instaladas en su equipo.
* Una organización [!DNL Experience Cloud]. Más información [aquí](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials).
* La organización de experiencias debe tener [!DNL Experience Manager] como [!DNL Cloud Service] habilitado.
* [!DNL Adobe Experience Cloud] forma parte del programa de vista previa para  [!DNL Project Firefly] desarrolladores. Consulte [cómo solicitar acceso](https://www.adobe.io/project-firefly/docs/overview/getting_access/).
* Asegúrese de que haya un rol de desarrollador o permisos de administrador en la organización para el desarrollador.
* Asegúrese de que [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) esté instalado localmente.

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
