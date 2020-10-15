---
title: '[!DNL Asset Compute Service] API HTTP.'
description: '[!DNL Asset Compute Service] API HTTP para crear aplicaciones personalizadas.'
translation-type: tm+mt
source-git-commit: 18e97e544014933e9910a12bc40246daa445bf4f
workflow-type: tm+mt
source-wordcount: '2931'
ht-degree: 2%

---


# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

El uso de la API se limita a fines de desarrollo. La API se proporciona como contexto al desarrollar aplicaciones personalizadas. [!DNL Adobe Experience Manager] como Cloud Service utiliza la API para pasar la información de procesamiento a una aplicación personalizada. Para obtener más información, consulte [Uso de microservicios de recursos y Perfiles](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)de procesamiento.

>[!NOTE]
>
>[!DNL Asset Compute Service] sólo está disponible para su uso con [!DNL Experience Manager] Cloud Service.

Cualquier cliente de la API de [!DNL Asset Compute Service] HTTP debe seguir este flujo de alto nivel:

1. Un cliente se aprovisiona como [!DNL Adobe Developer Console] proyecto en una organización de IMS. Cada cliente independiente (sistema o entorno) requiere su propio proyecto para separar el flujo de datos de evento.

1. Un cliente genera un token de acceso para la cuenta técnica mediante la autenticación [](https://www.adobe.io/authentication/auth-methods.html)JWT (Service Account).

1. Un cliente llama [`/register`](#register) sólo una vez para recuperar la dirección URL de historial.

1. Un cliente llama [`/process`](#process-request) a cada recurso para el que desea generar representaciones. La llamada es asincrónica.

1. Un cliente sondea regularmente al historial para [recibir eventos](#asynchronous-events). Recibe eventos por cada representación solicitada cuando la representación se procesa correctamente (`rendition_created` tipo de evento) o si se produce un error (`rendition_failed` tipo de evento).

El módulo [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) facilita el uso de la API en el código Node.js.

## Autenticación y autorización {#authentication-and-authorization}

Todas las API requieren autenticación de token de acceso. Las solicitudes deben establecer los siguientes encabezados:

1. `Authorization` encabezado con token de portador, que es el token de cuenta técnica, recibido a través del intercambio [](https://www.adobe.io/authentication/auth-methods.html) JWT desde el proyecto de la consola de desarrollador de Adobe. Los [ámbitos](#scopes) se documentan a continuación.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` con el ID de organización de IMS.

1. `x-api-key` con el ID de cliente del [!DNL Adobe Developers Console] proyecto.

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

Estos requisitos requieren que el [!DNL Adobe Developer Console] proyecto esté suscrito a `Asset Compute`, `I/O Events`y `I/O Management API` servicios. El desglose de ámbitos individuales es:

* Básico
   * scopes: `openid,AdobeID`

* Cálculo de recursos
   * metascope: `asset_compute_meta`
   * scopes: `asset_compute,read_organizations`

* Eventos de E/S de Adobe
   * metascope: `event_receiver_api`
   * scopes: `event_receiver,event_receiver_api`

* API de administración de E/S de Adobe
   * metascope: `ent_adobeio_sdk`
   * scopes: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registro {#register}

Cada cliente del [!DNL Asset Compute service] - un [!DNL Adobe Developer Console] proyecto único suscrito al servicio- debe [registrarse](#register-request) antes de realizar solicitudes de procesamiento. El paso de registro devuelve el historial de evento único que se requiere para recuperar los eventos asincrónicos del procesamiento de la representación.

Al final de su ciclo vital, un cliente puede [anular el registro](#unregister-request).

### Solicitud de registro {#register-request}

Esta llamada de API configura un [!DNL Asset Compute] cliente y proporciona la URL de historial de evento. Se trata de una operación ideal y sólo se debe llamar una vez por cada cliente. Se puede volver a llamar para recuperar la dirección URL de historial.

| Parámetro | Value |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/register` |
| Encabezado `Authorization` | Todos los encabezados [relacionados con la](#authentication-and-authorization)autorización. |
| Encabezado `x-request-id` | Opcional: los clientes pueden configurar un identificador único de extremo a extremo de las solicitudes de procesamiento en todos los sistemas. |
| Cuerpo de solicitud | Debe estar vacío. |

### Registrar respuesta {#register-response}

| Parámetro | Value |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | Igual que el encabezado de la `X-Request-Id` solicitud o que uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas y/o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `journal`campos `ok` y/o `requestId` . |

Los códigos de estado HTTP son:

* **200**&#x200B;Éxito: Cuando la solicitud se realiza correctamente. Contiene la `journal` dirección URL que se notifica sobre los resultados del procesamiento asincrónico activado mediante `/process` (como tipo de evento `rendition_created` cuando se realiza correctamente o `rendition_failed` cuando se produce un error).

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **No autorizado**: se produce cuando la solicitud no tiene una [autenticación](#authentication-and-authorization)válida. Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene una [autorización](#authentication-and-authorization)válida. Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **429 Demasiadas solicitudes**: se produce cuando el sistema está sobrecargado por este cliente o por otro motivo. Los clientes deben volver a intentarlo con un [retroceso](https://en.wikipedia.org/wiki/Exponential_backoff)exponencial. El cuerpo está vacío.
* **Error** 4xx: Cuando hubo algún otro error de cliente y error en el registro. Normalmente se devuelve una respuesta JSON como ésta, aunque no está garantizada para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Error** 5xx: se produce cuando se produce cualquier otro error en el servidor y se produce un error en el registro. Normalmente se devuelve una respuesta JSON como ésta, aunque no está garantizada para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### Cancelar el registro de solicitudes {#unregister-request}

Esta llamada de API desregistra a un [!DNL Asset Compute] cliente. Después de esto ya no es posible llamar a `/process`. El uso de la llamada de API para un cliente no registrado o un cliente aún no registrado devuelve un `404` error.

| Parámetro | Value |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/unregister` |
| Encabezado `Authorization` | Todos los encabezados [relacionados con la](#authentication-and-authorization)autorización. |
| Encabezado `x-request-id` | Opcional: los clientes pueden configurar un identificador único de extremo a extremo de las solicitudes de procesamiento en todos los sistemas. |
| Cuerpo de solicitud | Vacío. |

### Cancelar el registro de la respuesta {#unregister-response}

| Parámetro | Value |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | Igual que el encabezado de la `X-Request-Id` solicitud o que uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas y/o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` campos y `requestId` . |

Los códigos de estado son:

* **200**&#x200B;Éxito: se produce cuando se encuentra y elimina el registro y el historial.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **No autorizado**: se produce cuando la solicitud no tiene una [autenticación](#authentication-and-authorization)válida. Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene una [autorización](#authentication-and-authorization)válida. Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **404 No encontrado**: se produce cuando no hay un registro actual para las credenciales dadas.

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429 Demasiadas solicitudes**: se produce cuando el sistema está sobrecargado. Los clientes deben volver a intentarlo con un [retroceso](https://en.wikipedia.org/wiki/Exponential_backoff)exponencial. El cuerpo está vacío.

* **Error** 4xx: se produce cuando se produce cualquier otro error de cliente y se produce un error al cancelar el registro. Normalmente se devuelve una respuesta JSON como ésta, aunque no está garantizada para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Error** 5xx: se produce cuando se produce cualquier otro error en el servidor y se produce un error en el registro. Normalmente se devuelve una respuesta JSON como ésta, aunque no está garantizada para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## Solicitud de proceso {#process-request}

La `process` operación envía un trabajo que transforma un recurso de origen en varias representaciones, según las instrucciones de la solicitud. Las notificaciones sobre la finalización correcta (tipo de evento `rendition_created`) o cualquier error (tipo de evento `rendition_failed`) se envían a un historial de Evento que debe recuperarse utilizando [/registrarse](#register) una vez antes de realizar cualquier número de `/process` solicitudes. Las solicitudes formadas incorrectamente fallan inmediatamente con un código de error 400.

Se hace referencia a los binarios mediante direcciones URL, como URL prefirmadas de Amazon AWS S3 o URL SAS de Almacenamiento de blob de Azure, tanto para leer el `source` recurso (`GET` URL) como para escribir las representaciones (`PUT` URL). El cliente es responsable de generar estas direcciones URL prefirmadas.

| Parámetro | Value |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/process` |
| Tipo MIME | `application/json` |
| Encabezado `Authorization` | Todos los encabezados [relacionados con la](#authentication-and-authorization)autorización. |
| Encabezado `x-request-id` | Opcional: los clientes pueden configurar un identificador único de extremo a extremo de las solicitudes de procesamiento en todos los sistemas. |
| Cuerpo de solicitud | Debe estar en el formato JSON de solicitud de proceso como se describe a continuación. Proporciona instrucciones sobre qué recurso procesar y qué representaciones generar. |

### JSON de solicitud de proceso {#process-request-json}

El cuerpo de solicitud de `/process` es un objeto JSON con este esquema de alto nivel:

```json
{
    "source": "",
    "renditions" : []
}
```

Los campos disponibles son:

| Nombre | Tipo | Descripción | Ejemplo |
|--------------|----------|-------------|---------|
| `source` | `string` | Dirección URL del recurso de origen que se va a procesar. Opcional según el formato de representación solicitado (p. ej. `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descripción del recurso de origen que se va a procesar. Consulte la descripción de los campos [del objeto](#source-object-fields) Source a continuación. Opcional según el formato de representación solicitado (p. ej. `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representaciones que se generarán a partir del archivo de origen. Cada objeto de representación admite instrucciones [de representación](#rendition-instructions). Requerido. | `[{ "target": "https://....", "fmt": "png" }]` |

El `source` puede ser un `<string>` que se vea como una dirección URL o un campo `<object>` con un campo adicional. Las siguientes variantes son similares:

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
| `url` | `string` | Dirección URL del recurso de origen que se va a procesar. Requerido. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nombre del archivo de recurso de origen. Si no se detecta ningún tipo MIME, puede utilizarse la extensión de archivo en el nombre. Tiene prioridad sobre el nombre de archivo en la ruta de URL o el nombre de archivo en el `content-disposition` encabezado del recurso binario. El valor predeterminado es &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Tamaño del archivo de recurso de origen en bytes. Tiene prioridad sobre `content-length` el encabezado del recurso binario. | `10234` |
| `mimetype` | `string` | Tipo MIME del archivo de recurso de origen. Tiene prioridad sobre el `content-type` encabezado del recurso binario. | `"image/jpeg"` |

### Un ejemplo de `process` solicitud completa {#complete-process-request-example}

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

## Respuesta de proceso {#process-response}

La `/process` solicitud se devuelve inmediatamente con un éxito o un error según la validación básica de la solicitud. El procesamiento real de recursos se realiza de forma asíncrona.

| Parámetro | Value |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | Igual que el encabezado de la `X-Request-Id` solicitud o que uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas y/o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` campos y `requestId` . |

Códigos de estado:

* **200**&#x200B;Éxito: Si la solicitud se envió correctamente. La respuesta JSON incluye `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400 Solicitud** no válida: Si la solicitud está formada incorrectamente, como los campos obligatorios que faltan en la solicitud JSON. La respuesta JSON incluye `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **No autorizado**: Cuando la solicitud no tiene una [autenticación](#authentication-and-authorization)válida. Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.
* **403 Prohibido**: Cuando la solicitud no tiene una [autorización](#authentication-and-authorization)válida. Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.
* **429 Demasiadas solicitudes**: Cuando el sistema está sobrecargado por este cliente o en general. Los clientes pueden volver a intentarlo con un retroceso [exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error** 4xx: Cuando hubo algún otro error de cliente. Normalmente se devuelve una respuesta JSON como ésta, aunque no está garantizada para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **Error** 5xx: Cuando hubo algún otro error en el servidor. Normalmente se devuelve una respuesta JSON como ésta, aunque no está garantizada para todos los errores:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

Es probable que la mayoría de los clientes se inclinen a reintentar la misma solicitud con un [retroceso](https://en.wikipedia.org/wiki/Exponential_backoff) exponencial en cualquier error, *excepto* en problemas de configuración como 401 o 403, o solicitudes no válidas como 400. Aparte de la limitación regular de la tasa a través de 429 respuestas, una interrupción o limitación temporal del servicio podría resultar en errores 5xx. Luego sería aconsejable volver a intentarlo después de un período de tiempo.

Todas las respuestas JSON (si están presentes) incluyen el `requestId` que es el mismo valor que el `X-Request-Id` encabezado. Se recomienda leer desde el encabezado, ya que siempre está presente. También `requestId` se devuelve en todos los eventos relacionados con el procesamiento de solicitudes como `requestId`. Los clientes no deben asumir el formato de esta cadena, es un identificador de cadena opaco.

## Inclusión en el posprocesamiento {#opt-in-to-post-processing}

El SDK [de cómputo de](https://github.com/adobe/asset-compute-sdk) recursos admite un conjunto de opciones básicas de posprocesamiento de imágenes. Los trabajadores personalizados pueden adhesión explícitamente al postprocesamiento estableciendo el campo `postProcess` del objeto de representación en `true`.

Los casos de uso admitidos son:

* Recorte una representación en un rectángulo cuyos límites se definen como recorte.w, recorte.h, recorte.x y recorte.y. Se define mediante `instructions.crop` el objeto de representación.
* Cambie el tamaño de las imágenes con anchura, altura o ambas opciones. Se define mediante `instructions.width` y `instructions.height` en el objeto de representación. Para cambiar el tamaño utilizando solo la anchura o la altura, defina solo un valor. El servicio de cómputo conserva la proporción de aspecto.
* Defina la calidad de una imagen JPEG. Se define mediante `instructions.quality` el objeto de representación. La mejor calidad se denota por `100` y los valores más pequeños indican una calidad reducida.
* Cree imágenes entrelazadas. Se define mediante `instructions.interlace` el objeto de representación.
* Configure PPP para ajustar el tamaño procesado con fines de publicación de escritorio ajustando la escala aplicada a los píxeles. Se define mediante `instructions.dpi` el cambio de resolución de ppp en el objeto de representación. Sin embargo, para cambiar el tamaño de la imagen de forma que tenga el mismo tamaño y una resolución diferente, utilice las `convertToDpi` instrucciones.
* Cambie el tamaño de la imagen para que su anchura o altura representadas permanezcan iguales que el original en la resolución de destinatario especificada (PPP). Se define mediante `instructions.convertToDpi` el objeto de representación.

## Recursos de marca de agua {#add-watermark}

El SDK [de](https://github.com/adobe/asset-compute-sdk) cómputo de recursos admite la adición de una marca de agua a archivos de imagen PNG, JPEG, TIFF y GIF. La marca de agua se agrega siguiendo las instrucciones de representación del `watermark` objeto de la representación.

La marca de agua se realiza durante el posprocesamiento de la representación. Para marcar recursos, el trabajador personalizado [opta por el postprocesamiento](#opt-in-to-post-processing) estableciendo el campo `postProcess` del objeto de representación en `true`. Si el programa de trabajo no admite la opción de inclusión, no se aplica la marca de agua, incluso si el objeto de marca de agua está establecido en el objeto de representación en la solicitud.

## Instrucciones de representación {#rendition-instructions}

Estas son las opciones disponibles para el `renditions` arreglo de discos en [proceso](#process-request).

### Campos comunes {#common-fields}

| Nombre | Tipo | Descripción | Ejemplo |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | El formato de destinatario de las representaciones también puede ser `text` para la extracción de texto y `xmp` para extraer metadatos de XMP como xml. Consulte formatos [admitidos](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | Dirección URL de una aplicación [](develop-custom-application.md)personalizada. Debe ser una `https://` dirección URL. Si este campo está presente, la representación se crea mediante una aplicación personalizada. Cualquier otro campo de representación definido se utiliza en la aplicación personalizada. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Dirección URL a la que se debe cargar la representación generada mediante el PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Información de carga de URL prefirmada de varias partes para la representación generada. Esto es para la carga binaria directa de [AEM/Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con este comportamiento [de carga de](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)varias partes.<br>Fields:<ul><li>`urls`:: matriz de cadenas, una para cada URL de pieza prefirmada</li><li>`minPartSize`:: el tamaño mínimo que se debe usar para una parte = url</li><li>`maxPartSize`:: el tamaño máximo que se debe usar para una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Espacio reservado opcional controlado por el cliente y transferido como es a los eventos de representación. Permite a los clientes agregar información personalizada para identificar eventos de representación. No se debe modificar ni confiar en las aplicaciones personalizadas, ya que los clientes pueden cambiarlo en cualquier momento. | `{ ... }` |

### Campos específicos de representación {#rendition-specific-fields}

Para obtener una lista de los formatos de archivo admitidos actualmente, consulte Formatos [de archivo](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html)admitidos.

| Nombre | Tipo | Descripción | Ejemplo |
|-------------------|----------|-------------|---------|
| `*` | `*` | Se pueden agregar campos personalizados y avanzados que una aplicación [](develop-custom-application.md) personalizada comprenda. |  |
| `embedBinaryLimit` | `number` en bytes | Si este valor se establece y el tamaño del archivo de la representación es menor que este valor, la representación se incrusta en el evento que se envía una vez que se ha completado la generación de la representación. El tamaño máximo permitido para la incrustación es de 32 KB (32 x 1024 bytes). Si una representación tiene un tamaño mayor que el `embedBinaryLimit` límite, se coloca en una ubicación del almacenamiento de nube y no se incrusta en el evento. | `3276` |
| `width` | `number` | Anchura en píxeles. solo para representaciones de imágenes. | `200` |
| `height` | `number` | Altura en píxeles. solo para representaciones de imágenes. | `200` |
|  |  | La relación de aspecto siempre se mantiene si: <ul> <li> Se especifican `width` y `height` , y la imagen se ajusta al tamaño manteniendo la proporción de aspecto </li><li> Solo `width` o sólo `height` se especifica, la imagen resultante utiliza la dimensión correspondiente manteniendo la proporción de aspecto</li><li> Si no `width` se especifica ni `height` , se utiliza el tamaño de píxel de la imagen original. Depende del tipo de origen. Para algunos formatos, como los archivos PDF, se utiliza un tamaño predeterminado. Puede haber un límite de tamaño máximo.</li></ul> |  |
| `quality` | `number` | Especifique la calidad de jpeg en el rango de `1` a `100`. Solo se aplica a representaciones de imágenes. | `90` |
| `xmp` | `string` | Solo se utiliza en XMP reescritura de metadatos, es XMP codificado en base64 para volver a escribir en la representación especificada. |  |
| `interlace` | `bool` | Cree archivos PNG o GIF entrelazados o JPEG progresivo configurándolo en `true`. No afecta a otros formatos de archivo. |  |
| `jpegSize` | `number` | Tamaño aproximado del archivo JPEG en bytes. Anula cualquier `quality` configuración. No afecta a otros formatos. |  |
| `dpi` | `number` o `object` | Configure los ppp x e y. Para simplificar, también se puede establecer en un único número que se utilice tanto para x como para y. No tiene ningún efecto en la imagen misma. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` o `object` | los valores de los ppp x e y vuelven a muestrear mientras se mantiene el tamaño físico. Para simplificar, también se puede establecer en un único número que se utilice tanto para x como para y. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de archivos para incluir en el archivo ZIP (`fmt=zip`). Cada entrada puede ser una cadena URL o un objeto con los campos:<ul><li>`url`:: URL para descargar el archivo</li><li>`path`:: Almacenar archivo en esta ruta en el ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Gestión de duplicados para archivos ZIP (`fmt=zip`). De forma predeterminada, varios archivos almacenados en la misma ruta en el ZIP generan un error. Si se establece `duplicate` en `ignore` resultados, solo se almacena el primer recurso y el resto se ignora. | `ignore` |
| `watermark` | `object` | Contiene instrucciones sobre la [marca de agua](#watermark-specific-fields). |  |

### Campos específicos de marca de agua {#watermark-specific-fields}

El formato PNG se utiliza como marca de agua.

| Nombre | Tipo | Descripción | Ejemplo |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala de la marca de agua, entre `0.0` y `1.0`. `1.0` significa que la marca de agua tiene su escala original (1:1) y que los valores inferiores reducen el tamaño de la marca de agua. | Un valor de `0.5` significa la mitad del tamaño original. |
| `image` | `url` | Dirección URL del archivo PNG que se va a utilizar para marcar el agua. |  |

## Eventos asincrónicos {#asynchronous-events}

Una vez finalizado el procesamiento de una representación o cuando se produce un error, se envía un evento a un Historial [de Eventos de E/S de](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)Adobe. Los clientes deben escuchar la dirección URL del historial proporcionada a través [/registrarse](#register). La respuesta de historial incluye una `event` matriz compuesta por un objeto para cada evento, de la cual el `event` campo incluye la carga útil de evento real.

El Tipo de evento de E/S de Adobe para todos los eventos del [!DNL Asset Compute Service] es `asset_compute`. El historial se suscribe automáticamente solo a este tipo de evento y no hay ningún otro requisito para filtrar según el Tipo de evento de E/S de Adobe. Los tipos de evento específicos del servicio están disponibles en la `type` propiedad del evento.

### Event types {#event-types}

| Evento | Descripción |
|---------------------|-------------|
| `rendition_created` | Se envía para cada representación procesada y cargada correctamente. |
| `rendition_failed` | Se envía por cada representación que no se pudo procesar o cargar. |

### Atributos de evento {#event-attributes}

| Atributo | Tipo | Evento | Descripción |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Marca de hora cuando se envió evento en formato simplificado extendido [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) , tal como se define en JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | ID de solicitud de la solicitud original a `/process`, igual que el `X-Request-Id` encabezado. |
| `source` | `object` | `*` | El `source` de la `/process` solicitud. |
| `userData` | `object` | `*` | El nombre `userData` de la representación de la `/process` solicitud, si se establece. |
| `rendition` | `object` | `rendition_*` | El objeto de representación correspondiente transferido `/process`. |
| `metadata` | `object` | `rendition_created` | Propiedades de [metadatos](#metadata) de la representación. |
| `errorReason` | `string` | `rendition_failed` | El [motivo](#error-reasons) de error de representación, si existe. |
| `errorMessage` | `string` | `rendition_failed` | Texto que proporciona más detalles sobre el error de representación, si existe. |

### Metadatos {#metadata}

| Propiedad | Descripción |
|--------|-------------|
| `repo:size` | El tamaño de la representación en bytes. |
| `repo:sha1` | El compendio sha1 de la representación. |
| `dc:format` | Tipo MIME de la representación. |
| `repo:encoding` | La codificación charset de la representación en caso de que sea un formato basado en texto. |
| `tiff:ImageWidth` | Ancho de la representación en píxeles. Solo está presente para representaciones de imágenes. |
| `tiff:ImageLength` | Duración de la representación en píxeles. Solo está presente para representaciones de imágenes. |

### Motivos de error {#error-reasons}

| Motivo | Descripción |
|---------|-------------|
| `RenditionFormatUnsupported` | El formato de representación solicitado no es compatible con el origen determinado. |
| `SourceUnsupported` | El origen específico no es compatible aunque se admita el tipo. |
| `SourceCorrupt` | Los datos de origen están dañados. Incluye archivos vacíos. |
| `RenditionTooLarge` | No se pudo cargar la representación con las direcciones URL prefirmadas que se proporcionan en `target`. El tamaño real de la representación está disponible como metadatos en `repo:size` y el cliente puede utilizarla para volver a procesar esta representación con el número correcto de direcciones URL prefirmadas. |
| `GenericError` | Cualquier otro error inesperado. |