---
title: Solución de problemas [!DNL Asset Compute Service]
description: Solucionar problemas y depurar aplicaciones personalizadas mediante [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# Solución de problemas {#troubleshoot}

Algunas sugerencias genéricas para la resolución de problemas que pueden ayudarle a solucionar problemas con el servicio de Asset compute son las siguientes:

* Asegúrese de que la aplicación JavaScript no se bloquee durante el inicio. Estos bloqueos suelen estar relacionados con la falta de una biblioteca o con una dependencia.
* Asegúrese de que se hace referencia a todas las dependencias que se van a instalar en el `package.json` archivo.
* Asegúrese de que los errores que puedan provenir de la limpieza en caso de error no generen sus propios errores que oculten el problema original.

* Al iniciar la herramienta para desarrolladores por primera vez con un nuevo [!DNL Asset Compute Service] integración, puede fallar la primera solicitud de procesamiento si el diario de eventos de Asset compute no está completamente configurado. Espere un poco para que el historial se configure antes de enviar otra solicitud.
* Si se producen errores al enviar el Asset compute `/register` o `/process` , asegúrese de que todas las API necesarias se añaden a la variable [!DNL Adobe I/O] Proyecto y Espacio de trabajo, es decir, Asset compute, [!DNL Adobe I/O] Eventos, [!DNL Adobe I/O] Gestión de eventos, y [!DNL Adobe I/O] Runtime.

## Problemas de inicio de sesión mediante [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Si tiene problemas para iniciar sesión en [!DNL Adobe Developer Console] [a través de [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)A continuación, agregue manualmente las credenciales necesarias para desarrollar, probar e implementar la aplicación personalizada:

1. Vaya al proyecto de App Builder de Adobe Developer y al espacio de trabajo en [Consola de Adobe Developer](https://console.adobe.io/) y pulse **[!UICONTROL Descargar]** desde la esquina superior derecha. Abra este archivo y guarde este JSON en un lugar seguro de su equipo.

1. Vaya al archivo ENV en la aplicación de Adobe Developer App Builder.

1. Añada el [!DNL Adobe I/O] Credenciales de Runtime. Obtenga la [!DNL Adobe I/O] Credenciales de tiempo de ejecución del JSON descargado. Las credenciales se encuentran en `project.workspace.services.runtime`. Añada el [!DNL Adobe I/O] Credenciales de tiempo de ejecución en `AIO_runtime_XXX` variables:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Añada la ruta absoluta al JSON descargado en el paso 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configurar el resto de [credenciales requeridas](develop-custom-application.md) necesaria para la herramienta de desarrollo de.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
