---
title: Establezca el entorno de desarrollo necesario para [!DNL Asset Compute Service].
description: Configuración de entorno de programadores [!DNL Asset Compute Service] para inicio que crea y prueba código personalizado.
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 3%

---


# Configuración de un entorno para desarrolladores {#create-dev-environment}

Para crear una configuración que le permita desarrollarse para [!DNL Asset Compute Service], siga estos requisitos e instrucciones.

1. [Adquiera acceso y credenciales](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) para Project Firefly.

1. [Configure el entorno](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) local y las herramientas necesarias.

1. Algunas herramientas más que le ayudan a empezar a desarrollarse sin problemas son:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org) (v10 a v12 LTS, no se recomiendan versiones impares) y [NPM](https://www.npmjs.com). El usuario de OSX HomeBrew puede hacer `brew install node` la instalación de ambos. De lo contrario, descárguelo de la página [de descarga de](https://nodejs.org/en/)NodeJS.
   * Un IDE que sea adecuado para NodeJS, recomendamos Código [Visual Studio (Código VS)](https://code.visualstudio.com) ya que es el IDE admitido para el depurador. Puede utilizar cualquier otro IDE como editor de código, pero aún no se admite el uso avanzado (por ejemplo, depurador).
   * [CLI](https://github.com/adobe/aio-cli) de AIO (`aio`): instalación mediante `npm install -g @adobe/aio-cli`.

1. Asegúrese de cumplir los [requisitos previos](/help/understand-extensibility.md#prerequisites-and-provisioning).

## Configuración de un proyecto de Firefly {#create-firefly-project}

1. Obtenga acceso a la función de administrador del sistema o de desarrollador en la organización de experiencias. Esto lo puede establecer un administrador del sistema en el [Admin Console](https://adminconsole.adobe.com/overview).

1. Inicie sesión en [Adobe Developer Console](https://console.adobe.io/). Asegúrese de que forma parte de la misma organización de Adobe Experience Cloud que el AEM como integración de Cloud Service. Para obtener más información sobre Adobe Developer Console, consulte la documentación [de](https://www.adobe.io/apis/experienceplatform/console/docs.html)la consola.

1. [Crear un proyecto](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)de luciérnagas. Haga clic en **[!UICONTROL Crear nuevo proyecto]** > **[!UICONTROL Proyecto a partir de plantilla]**. Seleccione Firefly. Crea un nuevo proyecto de luciérnagas con dos espacios de trabajo: `Production` y `Stage`. Añada espacios de trabajo adicionales, por ejemplo `Development`, según sea necesario.

1. En el proyecto de luciérnagas, seleccione un espacio de trabajo y suscríbase a los servicios necesarios para el cálculo de recursos. Haga clic en **Añadir a Proyecto** > **API** y agregar `Asset Compute`, `IO Events`y `IO Events Management` servicios. Al agregar la primera API, se le solicita que cree una clave privada. Guarde esta información en el equipo cuando necesite esta clave para probar la aplicación personalizada con la herramienta para desarrolladores.

## Next step {#next-step}

Ahora que el entorno está configurado, está listo para [crear una aplicación](develop-custom-application.md)personalizada.

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
