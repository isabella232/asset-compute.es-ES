---
title: Implementación [!DNL Asset Compute Service] aplicación personalizada
description: Implementación [!DNL Asset Compute Service] aplicación personalizada.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 129651ba432b75703bc27baa7081da60302f828d
workflow-type: tm+mt
source-wordcount: '184'
ht-degree: 3%

---

# Implementar una aplicación personalizada {#deploy-custom-application}

Para implementar la aplicación, utilice [implementación de aplicaciones de aio](https://github.com/adobe/aio-cli#aio-appdeploy) comando. En el terminal, el comando muestra una dirección URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma URL sin volver a implementar la aplicación, utilice [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) comando.

Use la dirección URL en un [Perfil de procesamiento en [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=es) para integrar su aplicación con [!DNL Experience Manager] como [!DNL Cloud Service].

Asegúrese de que el proyecto y el espacio de trabajo de App Builder se correspondan con la variable [!DNL Experience Manager] como [!DNL Cloud Service] entorno en el que desea utilizar la acción. Tiene diferentes entornos para desarrollo, ensayo y producción. Puede verificar el entorno comprobando `AIO_runtime_*` credenciales que se definen dentro del archivo ENV en la raíz de la aplicación Firefly. Por ejemplo, para implementar en un `Stage` espacio de trabajo, la variable `AIO_runtime_namespace` es del formato `xxxxxx_xxxxxxxxx_stage`. Para integrar con [!DNL Experience Manager] como [!DNL Cloud Service] Entorno de producción, utilice URL de aplicación de Firefly `Production` espacio de trabajo.

>[!CAUTION]
>
>No utilice un espacio de trabajo personal en [!DNL Experience Manager] entornos.

>[!MORELIKETHIS]
>
>* [Explicación y administración de entornos en [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

