---
title: Comprender el funcionamiento de una aplicación personalizada
description: Trabajo interno de [!DNL Asset Compute Service] aplicación personalizada para comprender mejor cómo funciona.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 2af710443cdc2e5e25e105eca6e779eb58631ae9
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# Internos de una aplicación personalizada {#how-custom-application-works}

Utilice la siguiente ilustración para comprender el flujo de trabajo completo cuando un cliente procesa un recurso digital mediante una aplicación personalizada.

![Flujo de trabajo de aplicaciones personalizadas](assets/customworker.svg)

*Figura: Pasos para procesar un recurso mediante [!DNL Asset Compute Service].*

## Registro {#registration}

El cliente debe llamar a [`/register`](api.md#register) una vez antes de la primera solicitud para [`/process`](api.md#process-request) para configurar y recuperar la URL del diario para recibir [!DNL Adobe I/O] Eventos para el Asset compute de Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

La variable [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) La biblioteca JavaScript se puede utilizar en aplicaciones NodeJS para gestionar todos los pasos necesarios desde el registro, el procesamiento hasta la gestión asincrónica de eventos. Para obtener más información sobre los encabezados necesarios, consulte [Autenticación y autorización](api.md).

## Procesando {#processing}

El cliente envía un [procesamiento](api.md#process-request) solicitud.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

El cliente es responsable de dar formato correcto a las representaciones con direcciones URL prefirmadas. La variable [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) La biblioteca JavaScript se puede usar en aplicaciones NodeJS para prefirmar direcciones URL. Actualmente, la biblioteca solo admite Azure Blob Storage y Containers AWS S3.

La solicitud de procesamiento devuelve un valor `requestId` que se puede usar para sondeo [!DNL Adobe I/O] Eventos.

A continuación se muestra un ejemplo de solicitud de procesamiento de aplicaciones personalizada.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

La variable [!DNL Asset Compute Service] envía las solicitudes de representación de aplicaciones personalizadas a la aplicación personalizada. Utiliza un POST HTTP para la URL de la aplicación proporcionada, que es la URL de acción web segura de App Builder. Todas las solicitudes utilizan el protocolo HTTPS para maximizar la seguridad de los datos.

La variable [SDK de asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilizado por una aplicación personalizada gestiona la solicitud de POST HTTP. También gestiona la descarga del origen, la carga de representaciones, el envío [!DNL Adobe I/O] eventos y gestión de errores.

<!-- TBD: Add the application diagram. -->

### Código de aplicación {#application-code}

El código personalizado solo necesita proporcionar una llamada de retorno que tome el archivo de origen disponible localmente (`source.path`). La variable `rendition.path` es la ubicación para colocar el resultado final de una solicitud de procesamiento de recursos. La aplicación personalizada utiliza la rellamada para convertir los archivos de origen disponibles localmente en un archivo de representación con el nombre transferido (`rendition.path`). Una aplicación personalizada debe escribir en `rendition.path` para crear una representación:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Descargar archivos de origen {#download-source}

Una aplicación personalizada solo trata los archivos locales. La descarga del archivo de origen la gestiona el [SDK de asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Creación de representaciones {#rendition-creation}

El SDK llama a un [función de llamada de retorno de representación](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) para cada representación.

La función de llamada de retorno tiene acceso a la variable [source](https://github.com/adobe/asset-compute-sdk#source) y [representación](https://github.com/adobe/asset-compute-sdk#rendition) objetos. La variable `source.path` ya existe y es la ruta a la copia local del archivo de origen. La variable `rendition.path` es la ruta en la que se debe almacenar la representación procesada. A menos que [Indicador disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) está configurado, la aplicación debe utilizar exactamente la variable `rendition.path`. De lo contrario, el SDK no puede localizar ni identificar el archivo de representación y falla.

La simplificación excesiva del ejemplo se hace para ilustrar y centrarse en la anatomía de una aplicación personalizada. La aplicación solo copia el archivo de origen en el destino de representación.

Para obtener más información sobre los parámetros de devolución de llamada de representación, consulte [API del SDK de asset compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Cargar representaciones {#upload-rendition}

Después de crear y almacenar cada representación en un archivo con la ruta proporcionada por `rendition.path`, el [SDK de asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carga cada representación en un almacenamiento en la nube (AWS o Azure). Una aplicación personalizada obtiene varias representaciones al mismo tiempo si, y solo si, la solicitud entrante tiene varias representaciones que apuntan a la misma URL de la aplicación. La carga en el almacenamiento en la nube se realiza después de cada representación y antes de ejecutar la rellamada para la siguiente representación.

La variable `batchWorker()` tiene un comportamiento diferente, ya que en realidad procesa todas las representaciones y solo después de que todas se hayan procesado las carga.

## [!DNL Adobe I/O] Eventos {#aio-events}

El SDK envía [!DNL Adobe I/O] Eventos para cada representación. Estos eventos son de tipo `rendition_created` o `rendition_failed` según el resultado. Consulte [asset compute de eventos asincrónicos](api.md#asynchronous-events) para obtener más información sobre los eventos.

## Recibir [!DNL Adobe I/O] Eventos {#receive-aio-events}

El cliente sondea la variable [[!DNL Adobe I/O] Diario de eventos](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) según su lógica de consumo. La URL del diario inicial es la que se proporciona en la variable `/register` Respuesta de API. Los eventos se pueden identificar utilizando la variable `requestId` que está presente en los eventos y que es el mismo que se devuelve en `/process`. Cada representación tiene un evento independiente que se envía en cuanto la representación se ha cargado (o ha fallado). Una vez que recibe un evento que coincide con un evento, el cliente puede mostrar o gestionar de otro modo las representaciones resultantes.

La biblioteca JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) simplifica el sondeo del diario mediante el `waitActivation()` para obtener todos los eventos.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Para obtener más información sobre cómo obtener eventos de diario, consulte [[!DNL Adobe I/O] API de eventos](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
