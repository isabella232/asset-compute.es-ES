---
title: Establezca el entorno de desarrollo necesario para [!DNL Asset Compute Service]
description: Configuración del entorno de desarrollo para [!DNL Asset Compute Service] para empezar a crear y probar código personalizado.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 4%

---

# Configuración de un entorno de desarrollo {#create-dev-environment}

Para crear una configuración que le permita desarrollar para [!DNL Asset Compute Service], siga estos requisitos e instrucciones.

1. [Adquisición de acceso y credenciales](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) para [!DNL Adobe Developer App Builder].

1. [Configuración del entorno local](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) y las herramientas necesarias.

1. Algunas herramientas adicionales que le ayudan a empezar a desarrollar sin problemas son las siguientes:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, no se recomiendan versiones impares) y [NPM](https://www.npmjs.com). El usuario de OSX HomeBrew puede hacer `brew install node` para instalar ambos. De lo contrario, descárguelo desde el [Página de descarga de NodeJS](https://nodejs.org/en/)
   * Un IDE que sea bueno para NodeJS, recomendamos [Código de Visual Studio (código VS)](https://code.visualstudio.com) ya que es el IDE compatible con el depurador. Puede utilizar cualquier otro IDE como editor de código, pero aún no se admite el uso avanzado (por ejemplo, depurador)
   * Instale la última versión[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Asegúrese de cumplir con las [requisitos previos](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configuración de un proyecto de App Builder {#create-App-Builder-project}

1. Asegúrese de que el administrador del sistema o la función de desarrollador tengan la función [!DNL Experience Cloud] organización. Esto lo configura un administrador del sistema en la [Admin Console](https://adminconsole.adobe.com/overview).

1. Inicie sesión en [Consola de Adobe Developer](https://console.adobe.io/). Asegúrese de que forma parte del mismo [!DNL Experience Cloud] organización como [!DNL Experience Manager] as a [!DNL Cloud Service] integración. Para obtener más información sobre la consola de Adobe Developer, consulte [Documentación de consola](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [Creación de un proyecto de App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Clic **[!UICONTROL Crear nuevo proyecto]** > **[!UICONTROL Proyecto a partir de plantilla]**. Seleccione Generador de aplicaciones. Crea un nuevo proyecto de App Builder con dos espacios de trabajo: `Production` y `Stage`. Añada espacios de trabajo adicionales, por ejemplo `Development`, según sea necesario.

1. En el proyecto del Creador de aplicaciones, seleccione un espacio de trabajo y suscríbase a los servicios necesarios para el Asset compute. Clic **Agregar al proyecto** > **API** y agregue `Asset Compute`, `IO Events`, y `IO Events Management` servicios. Al añadir la primera API, se solicita crear una clave privada. Guarde esta información en el equipo, ya que necesita esta clave para probar la aplicación personalizada con la herramienta para desarrolladores.

## Siguiente paso {#next-step}

Ahora que su entorno está configurado, está listo para lo siguiente [crear una aplicación personalizada](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
