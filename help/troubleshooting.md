---
title: Solución de problemas [!DNL Asset Compute Service]
description: Solucionar problemas y depurar aplicaciones personalizadas usando [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '285'
ht-degree: 1%

---

# Solución de problemas {#troubleshoot}

Algunas sugerencias genéricas para la resolución de problemas que pueden ayudarle a solucionar problemas con el servicio de Asset compute son:

* Asegúrese de que la aplicación JavaScript no se bloquee al inicio. Estos bloqueos generalmente están relacionados con una biblioteca que falta o una dependencia.
* Asegúrese de que todas las dependencias que se van a instalar estén referenciadas en el archivo `package.json` de la aplicación.
* Asegúrese de que los errores que puedan surgir de la limpieza en caso de error no generen sus propios errores que oculten el problema original.

* Al iniciar la herramienta de desarrollo por primera vez con una nueva integración [!DNL Asset Compute Service], puede fallar la primera solicitud de procesamiento si el diario de eventos de Asset compute no está completamente configurado. Espere un poco para que el historial se configure antes de enviar otra solicitud.
* Si se producen errores al enviar solicitudes de Asset compute `/register` o `/process`, asegúrese de que todas las API necesarias se añaden al proyecto y al espacio de trabajo [!DNL Adobe I/O], es decir, al Asset compute, a3/> eventos, a4/> administración de eventos y a5/> tiempo de ejecución.[!DNL Adobe I/O][!DNL Adobe I/O][!DNL Adobe I/O]

## Iniciar sesión en problemas mediante [!DNL Adobe I/O] CLI {#login-via-aio-cli}

Si tiene problemas para iniciar sesión en [!DNL Adobe Developer Console] [a través de la  [!DNL Adobe I/O] CLI](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli), agregue manualmente las credenciales necesarias para desarrollar, probar e implementar su aplicación personalizada:

1. Vaya al proyecto y al espacio de trabajo de Firefly en la [Consola del desarrollador de Adobe](https://console.adobe.io/) y pulse **[!UICONTROL Descargar]** desde la esquina superior derecha. Abra este archivo y guarde este JSON en un lugar seguro de su equipo.

1. Vaya al archivo ENV de su aplicación Firefly.

1. Agregue las [!DNL Adobe I/O] Credenciales de tiempo de ejecución. Obtenga las [!DNL Adobe I/O] credenciales de A Runtime del JSON descargado. Las credenciales están en `project.workspace.services.runtime`. Agregue las credenciales de [!DNL Adobe I/O] tiempo de ejecución en las variables `AIO_runtime_XXX` :

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Añada la ruta absoluta al JSON descargado en el paso 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure el resto de las [credenciales necesarias](develop-custom-application.md) necesarias para la herramienta para desarrolladores.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
