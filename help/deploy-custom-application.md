---
title: Implementar [!DNL Asset Compute Service] aplicación personalizada
description: Implementar [!DNL Asset Compute Service] aplicación personalizada.
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 0%

---


# Implementar una aplicación personalizada {#deploy-custom-application}

Para implementar la aplicación, utilice el comando [implementación de la aplicación de aio](https://github.com/adobe/aio-cli#aio-appdeploy). En el terminal, el comando muestra una dirección URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma dirección URL sin volver a implementar la aplicación, utilice el comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action).

Utilice la dirección URL en un [Perfil de procesamiento en [!DNL Experience Manager] como [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) para integrar la aplicación con [!DNL Experience Manager] como [!DNL Cloud Service].

Asegúrese de que el proyecto y el espacio de trabajo de Firefly se correspondan con el [!DNL Experience Manager] como un entorno [!DNL Cloud Service] donde desee utilizar la acción. Tiene diferentes entornos de desarrollo, ensayo y producción. Puede comprobar el entorno comprobando las `AIO_runtime_*` credenciales definidas dentro del archivo ENV en la raíz de la aplicación Firefly. Por ejemplo, para implementar en un espacio de trabajo `Stage`, `AIO_runtime_namespace` tiene el formato `xxxxxx_xxxxxxxxx_stage`. Para integrarse con [!DNL Experience Manager] como [!DNL Cloud Service] entorno de producción, utilice las URL de las aplicaciones del espacio de trabajo Firefly `Production`.

>[!CAUTION]
>
>No utilice un espacio de trabajo personal en entornos [!DNL Experience Manager] críticos.

>[!MORELIKETHIS]
>
>* [Comprender y administrar entornos  [!DNL Experience Manager] inas a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

