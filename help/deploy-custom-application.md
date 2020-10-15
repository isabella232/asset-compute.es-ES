---
title: ' [!DNL Asset Compute Service] Implementar aplicación personalizada.'
description: ' [!DNL Asset Compute Service] Implementar aplicación personalizada.'
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '201'
ht-degree: 8%

---


# Implementar una aplicación personalizada {#deploy-custom-application}

Para implementar la aplicación, utilice el comando [de implementación](https://github.com/adobe/aio-cli#aio-appdeploy) de la aplicación de AIO. En el terminal, el comando muestra una dirección URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma dirección URL sin volver a implementar la aplicación, utilice [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) command.

Utilice la URL en un Perfil [de procesamiento en Experience Manager como Cloud Service](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) para integrar la aplicación con [!DNL Experience Manager] como Cloud Service.

Asegúrese de que el proyecto y el espacio de trabajo de Firefly se correspondan con el [!DNL Experience Manager] como entorno Cloud Service en el que desea utilizar la acción. Tiene diferentes entornos de desarrollo, ensayo y producción. Puede comprobar el entorno comprobando `AIO_runtime_*` las credenciales definidas dentro del archivo ENV en la raíz de la aplicación Firefly. Por ejemplo, para implementar en un `Stage` espacio de trabajo, el formato `AIO_runtime_namespace` es el `xxxxxx_xxxxxxxxx_stage`. Para realizar la integración con [!DNL Experience Manager] como entorno de producción Cloud Service, utilice las URL de la aplicación desde el espacio de trabajo de Firefly `Production` .

>[!CAUTION]
>
>No utilice un espacio de trabajo personal en [!DNL Experience Manager] entornos críticos.

>[!MORELIKETHIS]
>
>* [Comprender y administrar entornos en Experience Manager como Cloud Service](https://docs.adobe.com/content/help/es-ES/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

