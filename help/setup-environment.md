---
title: Establezca el entorno de desarrollo necesario para [!DNL Asset Compute Service]
description: Configuración del entorno del desarrollador para [!DNL Asset Compute Service] para empezar a crear y probar código personalizado.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 2b690ce37c0bac58abcb745e0b82018541434659
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# Configuración de un entorno de desarrollador {#create-dev-environment}

Para crear una configuración que le permita desarrollarse para [!DNL Asset Compute Service], siga estos requisitos e instrucciones.

1. [Adquirir acceso y credenciales](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) para [!DNL Project Firefly].

1. [Configuración del entorno local](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) y las herramientas necesarias.

1. Algunas herramientas más que le ayudarán a empezar a desarrollarse sin problemas son:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, no se recomiendan versiones extrañas) y [NPM](https://www.npmjs.com). El usuario de OSX HomeBrew puede hacer lo siguiente: `brew install node` para instalar ambos. De lo contrario, descárguelo desde la [Página de descarga de NodeJS](https://nodejs.org/en/)
   * Un IDE que sea adecuado para NodeJS, recomendamos [Código de Visual Studio (Código VS)](https://code.visualstudio.com) ya que es el IDE admitido para Debugger. Puede utilizar cualquier otro IDE como editor de código, pero el uso avanzado (por ejemplo, depurador) aún no es compatible
   * Instale la última[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Asegúrese de cumplir con el [requisitos previos](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configuración de un proyecto de App Builder {#create-App-Builder-project}

1. Asegúrese de que el administrador del sistema o el desarrollador cumplen la función de [!DNL Experience Cloud] organización. Esto lo configura un administrador del sistema en la variable [Admin Console](https://adminconsole.adobe.com/overview).

1. Inicie sesión en el [Consola de Adobe Developer](https://console.adobe.io/). Asegúrese de que forma parte del mismo [!DNL Experience Cloud] organización como [!DNL Experience Manager] como [!DNL Cloud Service] integración. Para obtener más información sobre la consola de Adobe Developer, consulte [Documentación de la consola](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Creación de un proyecto de App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Haga clic en **[!UICONTROL Crear nuevo proyecto]** > **[!UICONTROL Proyecto a partir de una plantilla]**. Seleccione Creador de aplicaciones. Se crea un nuevo proyecto de App Builder con dos espacios de trabajo: `Production` y `Stage`. Agregar espacios de trabajo adicionales, por ejemplo `Development`, según sea necesario.

1. En el proyecto de App Builder, seleccione un espacio de trabajo y suscríbase a los servicios necesarios para el Asset compute. Haga clic en **Agregar a proyecto** > **API** y agregue `Asset Compute`, `IO Events`y `IO Events Management` servicios. Al añadir la primera API, se solicita la creación de una clave privada. Guarde esta información en el equipo, ya que necesita esta clave para probar la aplicación personalizada con la herramienta para desarrolladores.

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
