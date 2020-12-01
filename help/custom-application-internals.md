---
title: Comprender el funcionamiento de una aplicación personalizada.
description: Trabajo interno de [!DNL Asset Compute Service] aplicación personalizada para ayudarle a comprender cómo funciona.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '774'
ht-degree: 0%

---


# Internos de una aplicación personalizada {#how-custom-application-works}

Utilice la siguiente ilustración para comprender el flujo de trabajo de extremo a extremo cuando un cliente procesa un recurso digital mediante una aplicación personalizada.

![Flujo de trabajo personalizado de la aplicación](assets/customworker.png)

*Figura: Pasos necesarios para procesar un recurso mediante  [!DNL Asset Compute Service].*

## Registro {#registration}

El cliente debe llamar [`/register`](api.md#register) una vez antes de la primera solicitud a [`/process`](api.md#process-request) para configurar y recuperar la dirección URL de historial para recibir Eventos de Adobe I/O para la Asset compute de Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

La [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) biblioteca JavaScript se puede utilizar en las aplicaciones NodeJS para administrar todos los pasos necesarios, desde el registro, el procesamiento hasta la administración asincrónica de eventos. Para obtener más información sobre los encabezados requeridos, consulte [Autenticación y autorización](api.md).

## Procesando {#processing}

El cliente envía una solicitud [de procesamiento](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

El cliente es responsable de dar formato correcto a las representaciones con direcciones URL prefirmadas. La biblioteca [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript se puede utilizar en aplicaciones NodeJS para prefirmar direcciones URL. Actualmente, la biblioteca solo admite Contenedores Azure Blob Almacenamiento y AWS S3.

La solicitud de procesamiento devuelve un `requestId` que se puede utilizar para sondear Eventos de Adobe I/O.

A continuación se muestra una solicitud de procesamiento de aplicación personalizada de muestra.

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

El [!DNL Asset Compute Service] envía las solicitudes de representación de la aplicación personalizada a la aplicación personalizada. Utiliza un POST HTTP a la URL de la aplicación proporcionada, que es la URL de acción web segura de Project Firefly. Todas las solicitudes utilizan el protocolo HTTPS para maximizar la seguridad de los datos.

El [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilizado por una aplicación personalizada gestiona la solicitud de POST HTTP. También gestiona la descarga del origen, la carga de representaciones, el envío de eventos de E/S y la gestión de errores.

<!-- TBD: Add the application diagram. -->

### Código de aplicación {#application-code}

El código personalizado solo necesita proporcionar una llamada de retorno que tome el archivo de origen disponible localmente (`source.path`). El `rendition.path` es la ubicación donde se coloca el resultado final de una solicitud de procesamiento de recursos. La aplicación personalizada utiliza la rellamada para convertir los archivos de origen disponibles localmente en un archivo de representación con el nombre pasado (`rendition.path`). Una aplicación personalizada debe escribir en `rendition.path` para crear una representación:

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

Una aplicación personalizada solo se ocupa de los archivos locales. La descarga del archivo de origen se controla mediante el [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### Creación de representación {#rendition-creation}

El SDK llama a una función de devolución de llamada asincrónica [de representación](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) para cada representación.

La función callback tiene acceso a los objetos [source](https://github.com/adobe/asset-compute-sdk#source) y [rendition](https://github.com/adobe/asset-compute-sdk#rendition). El `source.path` ya existe y es la ruta a la copia local del archivo de origen. La `rendition.path` es la ruta donde se debe almacenar la representación procesada. A menos que se establezca el indicador [disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional), la aplicación debe utilizar exactamente el `rendition.path`. De lo contrario, el SDK no puede localizar ni identificar el archivo de representación y se produce un error.

La simplificación excesiva del ejemplo se realiza para ilustrar y centrarse en la anatomía de una aplicación personalizada. La aplicación simplemente copia el archivo de origen en el destino de la representación.

Para obtener más información sobre los parámetros de llamada de retorno de la representación, consulte [API de SDK de Asset compute](https://github.com/adobe/asset-compute-sdk#api-details).

### Cargar representaciones {#upload-rendition}

Después de crear y almacenar cada representación en un archivo con la ruta proporcionada por `rendition.path`, el [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carga cada representación en un almacenamiento de nube (ya sea AWS o Azure). Una aplicación personalizada obtiene varias representaciones al mismo tiempo si la solicitud entrante tiene varias representaciones que apuntan a la misma dirección URL de la aplicación, y sólo si lo tiene. La carga en el almacenamiento de nube se realiza después de cada representación y antes de ejecutar la llamada de retorno para la siguiente representación.

El `batchWorker()` tiene un comportamiento diferente, ya que en realidad procesa todas las representaciones y sólo después de que se hayan procesado las carga.

## eventos de Adobe I/O {#aio-events}

El SDK envía Eventos de Adobe I/O para cada representación. Estos eventos son de tipo `rendition_created` o `rendition_failed` según el resultado. Consulte [eventos asincrónicos de Asset compute](api.md#asynchronous-events) para obtener más información sobre eventos.

## Recibir Eventos de Adobe I/O {#receive-aio-events}

El cliente sondea el [Historial de Eventos de Adobe I/O](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) según su lógica de consumo. La URL de historial inicial es la que se proporciona en la respuesta de API `/register`. Los eventos se pueden identificar utilizando el `requestId` que está presente en los eventos y es el mismo que se devuelve en `/process`. Cada representación tiene un evento independiente que se envía en cuanto se ha cargado (o ha fallado) la representación. Una vez que recibe un evento coincidente, el cliente puede mostrar o manejar de otro modo las representaciones resultantes.

La biblioteca de JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) simplifica el sondeo de historial mediante el método `waitActivation()` para obtener todos los eventos.

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

Para obtener más información sobre cómo obtener eventos de historial, consulte [Adobe I/O Eventos API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
