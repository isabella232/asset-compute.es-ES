---
title: Establezca el entorno de desarrollo necesario para [!DNL Asset Compute Service]
description: Configuración del entorno de desarrollador para [!DNL Asset Compute Service] comenzar a crear y probar código personalizado.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 9404ffcc66a3b6ba206155d1b1a5c16a43e22a39
workflow-type: tm+mt
source-wordcount: '370'
ht-degree: 3%

---

# Configuración de un entorno de desarrollador {#create-dev-environment}

Para crear una configuración que le permita desarrollarse para [!DNL Asset Compute Service], siga estos requisitos e instrucciones.

1. [Adquiera acceso y ](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) credenciales para  [!DNL Project Firefly].

1. [Configure el ](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) entorno local y las herramientas necesarias.

1. Algunas herramientas más que le ayudarán a empezar a desarrollarse sin problemas son:

   * [Git](https://git-scm.com/).
   * [Docker Desktop](https://www.docker.com/get-started).
   * [NodeJS](https://nodejs.org)  (v10 a v12 LTS, no se recomiendan las versiones impares) y  [NPM](https://www.npmjs.com). El usuario de OSX HomeBrew puede hacer `brew install node` para instalar ambos. De lo contrario, descárguelo desde la [página de descarga de NodeJS](https://nodejs.org/en/).
   * Un IDE que sea adecuado para NodeJS, recomendamos [Visual Studio Code (VS Code)](https://code.visualstudio.com) ya que es el IDE admitido para Debugger. Puede utilizar cualquier otro IDE como editor de código, pero el uso avanzado (por ejemplo, depurador) aún no es compatible.
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)  (`aio`): instalación mediante  `npm install -g @adobe/aio-cli@7.1.0`.

1. Asegúrese de cumplir los [requisitos previos](/help/understand-extensibility.md#prerequisites-and-provisioning).

>[!NOTE]
>
>Por ahora, utilice [!DNL Adobe I/O] CLI v7.1.0 de y no utilice [!DNL Adobe I/O] CLI v8.

## Configuración de un proyecto de Firefly {#create-firefly-project}

1. Asegúrese de la función de administrador del sistema o desarrollador en la organización [!DNL Experience Cloud]. Esto lo configura un administrador del sistema en el [Admin Console](https://adminconsole.adobe.com/overview).

1. Inicie sesión en [Adobe Developer Console](https://console.adobe.io/). Asegúrese de que forma parte de la misma organización [!DNL Experience Cloud] que la [!DNL Experience Manager] como integración [!DNL Cloud Service]. Para obtener más información sobre Adobe Developer Console, consulte [Documentación de la consola](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Crear un proyecto](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md) de Firefly. Haga clic en **[!UICONTROL Crear nuevo proyecto]** > **[!UICONTROL Proyecto a partir de la plantilla]**. Seleccione Firefly. Crea un nuevo proyecto de Firefly con dos espacios de trabajo: `Production` y `Stage`. Agregue espacios de trabajo adicionales, por ejemplo `Development`, según sea necesario.

1. En el Proyecto Firefly, seleccione un espacio de trabajo y suscríbase a los servicios necesarios para el Asset compute. Haga clic en **Agregar al proyecto** > **API** y añada los servicios `Asset Compute`, `IO Events` y `IO Events Management`. Al añadir la primera API, se solicita la creación de una clave privada. Guarde esta información en el equipo, ya que necesita esta clave para probar la aplicación personalizada con la herramienta para desarrolladores.

## Paso siguiente {#next-step}

Ahora que el entorno está configurado, está listo para [crear una aplicación personalizada](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
