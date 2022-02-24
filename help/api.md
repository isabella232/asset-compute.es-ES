---
title: '"[!DNL Asset Compute Service] API HTTP"'
description: '"[!DNL Asset Compute Service] API HTTP para crear aplicaciones personalizadas."'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---

# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

El uso de la API se limita a fines de desarrollo. La API se proporciona como contexto al desarrollar aplicaciones personalizadas. [!DNL Adobe Experience Manager] como [!DNL Cloud Service] utiliza la API para pasar la información de procesamiento a una aplicación personalizada. Para obtener más información, consulte [Uso de microservicios de recursos y perfiles de procesamiento](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con [!DNL Experience Manager] como [!DNL Cloud Service].

Cualquier cliente de [!DNL Asset Compute Service] La API HTTP debe seguir este flujo de alto nivel:

1. Un cliente se aprovisiona como [!DNL Adobe Developer Console] en una organización IMS. Cada cliente independiente (sistema o entorno) requiere su propio proyecto para separar el flujo de datos del evento.

1. Un cliente genera un token de acceso para la cuenta técnica mediante la variable [Autenticación JWT (cuenta de servicio)](https://www.adobe.io/authentication/auth-methods.html).

1. Una llamada de cliente [`/register`](#register) solo una vez para recuperar la URL del historial.

1. Una llamada de cliente [`/process`](#process-request) para cada recurso para el que desea generar representaciones. La llamada de es asincrónica.

1. Un cliente sondea regularmente el diario a [recibir eventos](#asynchronous-events). Recibe eventos para cada representación solicitada cuando la representación se procesa correctamente (`rendition_created` tipo de evento) o si hay un error (`rendition_failed` tipo de evento).

La variable [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) facilita el uso de la API en el código Node.js.

## Autenticación y autorización {#authentication-and-authorization}

Todas las API requieren autenticación de token de acceso. Las solicitudes deben establecer los siguientes encabezados:

1. `Authorization` encabezado con token al portador, que es el token de cuenta técnica, recibido mediante [Intercambio JWT](https://www.adobe.io/authentication/auth-methods.html) desde el proyecto de Adobe Developer Console. La variable [ámbitos](#scopes) están documentados a continuación.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` con el ID de organización de IMS.

1. `x-api-key` con el ID de cliente de [!DNL Adobe Developers Console] proyecto.

### Ámbitos {#scopes}

Asegúrese de los siguientes ámbitos para el token de acceso:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Estos requieren que la variable [!DNL Adobe Developer Console] proyecto al que suscribirse `Asset Compute`, `I/O Events`y `I/O Management API` servicios. El desglose de ámbitos individuales es:

* Básico
   * ámbitos: `openid,AdobeID`

* asset compute
   * metascope: `asset_compute_meta`
   * ámbitos: `asset_compute,read_organizations`

* [!DNL Adobe I/O] Sucesos
   * metascope: `event_receiver_api`
   * ámbitos: `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] API de administración
   * metascope: `ent_adobeio_sdk`
   * ámbitos: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registro {#register}

Cada cliente de la variable [!DNL Asset Compute service] - un [!DNL Adobe Developer Console] proyecto suscrito al servicio: debe [register](#register-request) antes de realizar solicitudes de procesamiento. El paso de registro devuelve el diario de eventos único que es necesario para recuperar los eventos asincrónicos del procesamiento de representaciones.

Al final de su ciclo de vida, un cliente puede [cancelar registro](#unregister-request).

### Solicitud de registro {#register-request}

Esta llamada de API configura un [!DNL Asset Compute] y proporciona la dirección URL del diario de eventos. Se trata de una operación idempotente que solo debe llamarse una vez por cada cliente. Se puede volver a llamar para recuperar la URL del diario.

| Parámetro | Value |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/register` |
| Encabezado `Authorization` | Todo [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Encabezado `x-request-id` | De forma opcional, los clientes pueden configurarlo para un identificador único de extremo a extremo de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de la solicitud | Debe estar vacío. |

### Registrar respuesta {#register-response}

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | O igual que la variable `X-Request-Id` encabezado de solicitud o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `journal`, `ok` y/o `requestId` campos. |

Los códigos de estado HTTP son:

* **200 Éxito**: Cuando la solicitud se realiza correctamente. Contiene el `journal` URL a la que se notifica sobre cualquier resultado del procesamiento asincrónico activado mediante `/process` (como tipo de evento) `rendition_created` cuando se realiza correctamente, o `rendition_failed` cuando falle).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401 No autorizado**: se produce cuando la solicitud no tiene un valor válido [autenticación](#authentication-and-authorization). Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene un valor válido [autorización](#authentication-and-authorization). Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **429 Demasiadas solicitudes**: se produce cuando el sistema está sobrecargado por este cliente o por cualquier otro motivo. Los clientes deben volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error 4xx**: Cuando hubo algún otro error de cliente y error de registro. Normalmente se devuelve una respuesta JSON como esta, aunque no se garantiza para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Error 5xx**: se produce cuando se produce cualquier otro error del lado del servidor y falla el registro. Normalmente se devuelve una respuesta JSON como esta, aunque no se garantiza para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Cancelar registro de solicitud {#unregister-request}

Esta llamada de API cancela los registros y [!DNL Asset Compute] cliente. Después de esto ya no es posible llamar a `/process`. El uso de la llamada de API para un cliente no registrado o un cliente aún no registrado devuelve un valor `404` error.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/unregister` |
| Encabezado `Authorization` | Todo [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Encabezado `x-request-id` | De forma opcional, los clientes pueden configurarlo para un identificador único de extremo a extremo de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de la solicitud | Vacío. |

### Cancelar registro de respuestas {#unregister-response}

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | O igual que la variable `X-Request-Id` encabezado de solicitud o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` y `requestId` campos. |

Los códigos de estado son:

* **200 Éxito**: se produce cuando se encuentra y elimina el registro y el diario.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401 No autorizado**: se produce cuando la solicitud no tiene un valor válido [autenticación](#authentication-and-authorization). Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene un valor válido [autorización](#authentication-and-authorization). Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **404 No encontrado**: se produce cuando no hay registro actual para las credenciales dadas.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Demasiadas solicitudes**: se produce cuando el sistema está sobrecargado. Los clientes deben volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.

* **Error 4xx**: se produce cuando hay algún otro error de cliente y la cancelación del registro falla. Normalmente se devuelve una respuesta JSON como esta, aunque no se garantiza para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Error 5xx**: se produce cuando se produce cualquier otro error del lado del servidor y falla el registro. Normalmente se devuelve una respuesta JSON como esta, aunque no se garantiza para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Solicitud de proceso {#process-request}

La variable `process` La operación envía un trabajo que transforma un recurso de origen en varias representaciones, según las instrucciones de la solicitud. Notificaciones sobre la finalización correcta (tipo de evento) `rendition_created`) o cualquier error (tipo de evento) `rendition_failed`) se envían a un diario de eventos que debe recuperarse mediante [/register](#register) una vez antes de realizar cualquier número de `/process` solicitudes. Las solicitudes formadas incorrectamente fallan inmediatamente con un código de error 400.

Se hace referencia a los binarios mediante direcciones URL, como URL prefirmadas de Amazon AWS S3 o URL SAS de almacenamiento de blob de Azure, para leer ambas `source` recurso (`GET` URL) y escribir las representaciones (`PUT` URL). El cliente es responsable de generar estas direcciones URL prefirmadas.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/process` |
| Tipo MIME | `application/json` |
| Encabezado `Authorization` | Todo [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Encabezado `x-request-id` | De forma opcional, los clientes pueden configurarlo para un identificador único de extremo a extremo de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de la solicitud | Debe estar en el formato JSON de solicitud de proceso como se describe a continuación. Proporciona instrucciones sobre qué recurso procesar y qué representaciones generar. |

### JSON de solicitud de proceso {#process-request-json}

El cuerpo de la solicitud de `/process` es un objeto JSON con este esquema de alto nivel:

```json
{
    "source": "",
    "renditions" : []
}
```

Los campos disponibles son:

| Nombre | Tipo | Descripción | Ejemplo |
|--------------|----------|-------------|---------|
| `source` | `string` | URL del recurso de origen que se va a procesar. Opcional en función del formato de representación solicitado (p. ej. `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descripción del recurso de origen que se va a procesar. Consulte la descripción de [Campos de objeto de origen](#source-object-fields) más abajo. Opcional en función del formato de representación solicitado (p. ej. `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representaciones que se van a generar desde el archivo de origen. Cada objeto de representación admite [instrucción de representación](#rendition-instructions). Requerido. | `[{ "target": "https://....", "fmt": "png" }]` |

La variable `source` puede ser `<string>` que se ve como una dirección URL o puede ser una `<object>` con un campo adicional. Las siguientes variantes son similares:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Campos de objeto de origen {#source-object-fields}

| Nombre | Tipo | Descripción | Ejemplo |
|-----------|----------|-------------|---------|
| `url` | `string` | URL del recurso de origen que se va a procesar. Requerido. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nombre del archivo de recurso de origen. Si no se detecta ningún tipo MIME, puede utilizarse la extensión de archivo en el nombre. Tiene prioridad sobre el nombre de archivo en la ruta URL o el nombre de archivo en `content-disposition` encabezado del recurso binario. El valor predeterminado es &quot;archivo&quot;. | `"image.jpg"` |
| `size` | `number` | Tamaño del archivo de recurso de origen en bytes. Tiene prioridad sobre `content-length` encabezado del recurso binario. | `10234` |
| `mimetype` | `string` | Tipo MIME del archivo de recurso de origen. Tiene prioridad sobre la variable `content-type` encabezado del recurso binario. | `"image/jpeg"` |

### Una finalización `process` ejemplo de solicitud {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Respuesta del proceso {#process-response}

La variable `/process` la solicitud se devuelve inmediatamente con un éxito o un error basado en la validación básica de la solicitud. El procesamiento real de recursos se produce asincrónicamente.

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | O igual que la variable `X-Request-Id` encabezado de solicitud o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` y `requestId` campos. |

Códigos de estado:

* **200 Éxito**: Si la solicitud se envió correctamente. Respuesta: JSON incluye `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Solicitud no válida**: Si la solicitud está formada incorrectamente, como los campos obligatorios que faltan en el JSON de la solicitud. Respuesta: JSON incluye `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401 No autorizado**: Cuando la solicitud no tiene validez [autenticación](#authentication-and-authorization). Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.
* **403 Prohibido**: Cuando la solicitud no tiene validez [autorización](#authentication-and-authorization). Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.
* **429 Demasiadas solicitudes**: Cuando el sistema está sobrecargado por este cliente o en general. Los clientes pueden volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error 4xx**: Cuando hubo algún otro error de cliente. Normalmente se devuelve una respuesta JSON como esta, aunque no se garantiza para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Error 5xx**: Cuando hubo algún otro error del lado del servidor. Normalmente se devuelve una respuesta JSON como esta, aunque no se garantiza para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

Es probable que la mayoría de los clientes estén inclinados a reintentar la misma solicitud con [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff) en cualquier error *except* problemas de configuración como 401 o 403, o solicitudes no válidas como 400. Aparte de la limitación regular de la tasa a través de las respuestas 429, una interrupción o limitación temporal del servicio podría resultar en errores 5xx. A continuación, sería aconsejable volver a intentarlo después de un periodo de tiempo.

Todas las respuestas JSON (si están presentes) incluyen la variable `requestId` que es el mismo valor que la variable `X-Request-Id` encabezado. Se recomienda leer desde el encabezado, ya que siempre está presente. La variable `requestId` también se devuelve en todos los eventos relacionados con el procesamiento de solicitudes como `requestId`. Los clientes no deben asumir el formato de esta cadena, es un identificador de cadena opaco.

## Inclusión en el procesamiento posterior {#opt-in-to-post-processing}

La variable [SDK de asset compute](https://github.com/adobe/asset-compute-sdk) admite un conjunto de opciones básicas de posprocesamiento de imágenes. Los trabajadores personalizados pueden incluirse explícitamente en el posprocesamiento estableciendo el campo `postProcess` en el objeto de representación a `true`.

Los casos de uso admitidos son:

* Recorte una representación en un rectángulo cuyos límites estén definidos por recortar.w, recortar.h, recortar.x y recortar.y. Se define mediante `instructions.crop` en el objeto de representación.
* Cambie el tamaño de las imágenes mediante la anchura, la altura o ambas opciones. Se define mediante `instructions.width` y `instructions.height` en el objeto de representación. Para cambiar el tamaño utilizando solo la anchura o la altura, establezca solo un valor. Compute Service conserva la relación de aspecto.
* Establezca la calidad de una imagen de JPEG. Se define mediante `instructions.quality` en el objeto de representación. La mejor calidad se indica con `100` y los valores más pequeños indican una calidad reducida.
* Cree imágenes entrelazadas. Se define mediante `instructions.interlace` en el objeto de representación.
* Ajuste la DPI para ajustar el tamaño procesado con fines de publicación de escritorio ajustando la escala aplicada a los píxeles. Se define mediante `instructions.dpi` en el objeto de representación para cambiar la resolución de dpi. Sin embargo, para cambiar el tamaño de la imagen de modo que tenga el mismo tamaño con una resolución diferente, use la variable `convertToDpi` instrucciones.
* Cambie el tamaño de la imagen de forma que su anchura o altura representadas permanezca igual que la original en la resolución de destino especificada (DPI). Se define mediante `instructions.convertToDpi` en el objeto de representación.

## Recursos de marca de agua {#add-watermark}

La variable [SDK de asset compute](https://github.com/adobe/asset-compute-sdk) admite la adición de una marca de agua a los archivos de imagen PNG, JPEG, TIFF y GIF. La marca de agua se agrega siguiendo las instrucciones de representación de la variable `watermark` en la representación.

La marca de agua se realiza durante el procesamiento posterior de la representación. Para marcar recursos con agua, el programa de trabajo personalizado [opta por posprocesar](#opt-in-to-post-processing) estableciendo el campo `postProcess` en el objeto de representación a `true`. Si el trabajador no opta por la inclusión, no se aplica la marca de agua, aunque el objeto de marca de agua esté establecido en el objeto de representación de la solicitud.

## Instrucciones de representación {#rendition-instructions}

Estas son las opciones disponibles para la variable `renditions` matriz en [/process](#process-request).

### Campos comunes {#common-fields}

| Nombre | Tipo | Descripción | Ejemplo |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | El formato de destino de las representaciones también puede ser `text` para extracción de texto y `xmp` para extraer XMP metadatos como xml. Consulte [formatos admitidos](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL de un [aplicación personalizada](develop-custom-application.md). Debe ser `https://` URL. Si este campo está presente, la representación se crea mediante una aplicación personalizada. Cualquier otro campo de representación definido se utiliza en la aplicación personalizada. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Dirección URL a la que se debe cargar la representación generada mediante el PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Información de carga de URL prefirmada de varias partes para la representación generada. Esto es para [Carga binaria AEM/Oak Direct](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con esto [comportamiento de carga multiparte](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Fields:<ul><li>`urls`: matriz de cadenas, una para cada URL de pieza prefirmada</li><li>`minPartSize`: el tamaño mínimo que se debe usar para una parte = url</li><li>`maxPartSize`: el tamaño máximo que se debe usar para una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Espacio reservado opcional controlado por el cliente y transferido como está a los eventos de representación. Permite a los clientes añadir información personalizada para identificar eventos de representación. No debe modificarse ni basarse en aplicaciones personalizadas, ya que los clientes pueden cambiarlo en cualquier momento. | `{ ... }` |

### Campos específicos de representación {#rendition-specific-fields}

Para obtener una lista de los formatos de archivo compatibles actualmente, consulte [formatos de archivo compatibles](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

| Nombre | Tipo | Descripción | Ejemplo |
|-------------------|----------|-------------|---------|
| `*` | `*` | Los campos personalizados y avanzados se pueden agregar con un [aplicación personalizada](develop-custom-application.md) comprende. |  |
| `embedBinaryLimit` | `number` en bytes | Si este valor está establecido y el tamaño del archivo de la representación es menor que este valor, la representación se incrusta en el evento que se envía una vez que se completa la generación de la representación. El tamaño máximo permitido para la incrustación es de 32 KB (32 x 1024 bytes). Si una representación es más grande que el tamaño de `embedBinaryLimit` límite, se coloca en una ubicación de almacenamiento en la nube y no se incrusta en el evento. | `3276` |
| `width` | `number` | Anchura en píxeles. solo para representaciones de imágenes. | `200` |
| `height` | `number` | Altura en píxeles. solo para representaciones de imágenes. | `200` |
|  |  | La relación de aspecto siempre se mantiene si: <ul> <li> Ambas `width` y `height` se especifican, la imagen se ajusta al tamaño mientras se mantiene la relación de aspecto </li><li> Solo `width` o solo `height` se especifica, la imagen resultante utiliza la dimensión correspondiente y mantiene la relación de aspecto</li><li> Si `width` nor `height` se especifica, se utiliza el tamaño de píxel de la imagen original. Depende del tipo de origen. Para algunos formatos, como los archivos PDF, se utiliza un tamaño predeterminado. Puede haber un límite de tamaño máximo.</li></ul> |  |
| `quality` | `number` | Especifique la calidad de jpeg en el rango de `1` a `100`. Aplicable solo para representaciones de imágenes. | `90` |
| `xmp` | `string` | Solo se utiliza en XMP reescritura de metadatos, es XMP codificado base64 para volver a escribir en la representación especificada. |  |
| `interlace` | `bool` | Cree un PNG entrelazado o un GIF o un JPEG progresivo configurándolo en `true`. No afecta a otros formatos de archivo. |  |
| `jpegSize` | `number` | Tamaño aproximado del archivo JPEG en bytes. Anula cualquier `quality` configuración. No afecta a otros formatos. |  |
| `dpi` | `number` o `object` | Establezca x e y DPI. Para simplificar, también se puede establecer en un solo número que se utilice tanto para x como para y. No tiene ningún efecto en la imagen misma. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` o `object` | x e y DPI vuelven a tomar muestras de los valores manteniendo al mismo tiempo el tamaño físico. Para simplificar, también se puede establecer en un solo número que se utilice tanto para x como para y. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de archivos que se incluirán en el archivo ZIP (`fmt=zip`). Cada entrada puede ser una cadena URL o un objeto con los campos:<ul><li>`url`: URL para descargar archivo</li><li>`path`: Almacene el archivo en esta ruta en el ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Tratamiento de duplicados para archivos ZIP (`fmt=zip`). De forma predeterminada, varios archivos almacenados en la misma ruta en el ZIP generan un error. Configuración `duplicate` a `ignore` solo genera el primer recurso que se va a almacenar y el resto que se va a ignorar. | `ignore` |
| `watermark` | `object` | Contiene instrucciones sobre la variable [marca de agua](#watermark-specific-fields). |  |

### Campos específicos de marca de agua {#watermark-specific-fields}

El formato PNG se utiliza como marca de agua.

| Nombre | Tipo | Descripción | Ejemplo |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala de la marca de agua, entre `0.0` y `1.0`. `1.0` significa que la marca de agua tiene su escala original (1:1) y los valores inferiores reducen el tamaño de la marca de agua. | Un valor de `0.5` significa la mitad del tamaño original. |
| `image` | `url` | URL del archivo PNG que se va a usar para marcar con agua. |  |

## Eventos asincrónicos {#asynchronous-events}

Una vez finalizado el procesamiento de una representación o cuando se produce un error, se envía un evento a una [[!DNL Adobe I/O] Diario de eventos](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md). Los clientes deben escuchar la dirección URL del diario proporcionada a través de [/register](#register). La respuesta del diario incluye un `event` matriz formada por un objeto para cada evento, del cual la variable `event` incluye la carga útil de evento real.

La variable [!DNL Adobe I/O] Tipo de evento para todos los eventos del [!DNL Asset Compute Service] es `asset_compute`. El diario se suscribe automáticamente solo a este tipo de evento y no hay ningún otro requisito para filtrar según la variable [!DNL Adobe I/O] Tipo de evento. Los tipos de eventos específicos del servicio están disponibles en la `type` propiedad del evento.

### Tipos de eventos {#event-types}

| Evento | Descripción |
|---------------------|-------------|
| `rendition_created` | Enviado para cada representación procesada y cargada correctamente. |
| `rendition_failed` | Enviado para cada representación que no se haya podido procesar ni cargar. |

### Atributos de evento {#event-attributes}

| Atributo | Tipo | Evento | Descripción |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Marca de tiempo cuando se envía un evento en extensión simplificada [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) , tal como se define en JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | ID de solicitud de la solicitud original a `/process`, igual que `X-Request-Id` encabezado. |
| `source` | `object` | `*` | La variable `source` del `/process` solicitud. |
| `userData` | `object` | `*` | La variable `userData` de la representación desde el `/process` solicitud si está establecido. |
| `rendition` | `object` | `rendition_*` | El objeto de representación correspondiente transferido `/process`. |
| `metadata` | `object` | `rendition_created` | La variable [metadata](#metadata) propiedades de la representación. |
| `errorReason` | `string` | `rendition_failed` | Fallo de representación [reason](#error-reasons) si existe. |
| `errorMessage` | `string` | `rendition_failed` | Texto que proporciona más información sobre el error de representación, si lo hay. |

### Metadatos {#metadata}

| Propiedad | Descripción |
|--------|-------------|
| `repo:size` | El tamaño de la representación en bytes. |
| `repo:sha1` | El resumen sha1 de la representación. |
| `dc:format` | Tipo MIME de la representación. |
| `repo:encoding` | La codificación charset de la representación en caso de que sea un formato basado en texto. |
| `tiff:ImageWidth` | Anchura de la representación en píxeles. Solo está presente para representaciones de imágenes. |
| `tiff:ImageLength` | Longitud de la representación en píxeles. Solo está presente para representaciones de imágenes. |

### Motivos del error {#error-reasons}

| Motivo | Descripción |
|---------|-------------|
| `RenditionFormatUnsupported` | El formato de representación solicitado no es compatible con el origen dado. |
| `SourceUnsupported` | El origen específico no es compatible aunque el tipo sea compatible. |
| `SourceCorrupt` | Los datos de origen están dañados. Incluye archivos vacíos. |
| `RenditionTooLarge` | La representación no se pudo cargar con las direcciones URL prefirmadas que se proporcionan en `target`. El tamaño de representación real está disponible como metadatos en `repo:size` y el cliente puede usar para volver a procesar esta representación con el número correcto de direcciones URL prefirmadas. |
| `GenericError` | Cualquier otro error inesperado. |
