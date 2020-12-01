---
title: Solución de problemas [!DNL Asset Compute Service].
description: Solucionar problemas y depurar aplicaciones personalizadas mediante [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 1%

---


# Solución de problemas {#troubleshoot}

Algunas sugerencias genéricas para la resolución de problemas que pueden ayudarle a solucionar problemas con el servicio de Asset compute son:

* Asegúrese de que la aplicación JavaScript no se bloquea al iniciar. Estos bloqueos se suelen relacionar con una biblioteca que falta o con una dependencia.
* Asegúrese de que se haga referencia a todas las dependencias que se instalarán en el archivo `package.json` de la aplicación.
* Asegúrese de que los errores que puedan surgir de la limpieza en caso de falla no generen sus propios errores que oculten el problema original.

* Al iniciar la herramienta para desarrolladores por primera vez con una nueva integración [!DNL Asset Compute Service], es posible que falle la primera solicitud de procesamiento porque es posible que el Historial de Eventos de Asset compute no esté completamente configurado. Espere un tiempo para que el historial se configure antes de enviar otra solicitud.
* Si se producen errores al enviar solicitudes de Asset compute `/register` o `/process`, asegúrese de que todas las API necesarias se agregan al proyecto y al espacio de trabajo de Adobe I/O, es decir, Asset compute, Eventos de E/S, administración de Eventos de E/S y tiempo de ejecución.

## Iniciar sesión mediante CLI de Adobe I/O {#login-via-aio-cli}

Si tiene problemas al iniciar sesión en [!DNL Adobe Developer Console] [mediante la CLI de Adobe I/O](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli), agregue manualmente las credenciales necesarias para desarrollar, probar e implementar su aplicación personalizada:

1. Vaya al proyecto y al espacio de trabajo de Firefly en la [Consola de programadores de Adobe](https://console.adobe.io/) y pulse **[!UICONTROL Descargar]** desde la esquina superior derecha. Abra este archivo y guarde este JSON en un lugar seguro de su equipo.

1. Vaya al archivo ENV de la aplicación Firefly.

1. Añada las credenciales de Adobe I/O Runtime. Obtenga las credenciales de Adobe I/O Runtime del JSON descargado. Las credenciales están en `project.workspace.services.runtime`. Añada las credenciales del motor de ejecución de E/S en las variables `AIO_runtime_XXX`:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Añada la ruta absoluta al JSON descargado en el paso 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure el resto de las [credenciales requeridas](develop-custom-application.md) necesarias para la herramienta para desarrolladores.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
