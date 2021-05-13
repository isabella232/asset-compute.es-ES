---
title: Probar y depurar [!DNL Asset Compute Service] aplicación personalizada
description: Probar y depurar [!DNL Asset Compute Service] aplicación personalizada.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: ebc0d717b3f6fc4518f4a79cd44ebe8fdcf9ec6a
workflow-type: tm+mt
source-wordcount: '811'
ht-degree: 0%

---

# Probar y depurar una aplicación personalizada {#test-debug-custom-worker}

## Ejecutar pruebas unitarias para una aplicación personalizada {#test-custom-worker}

Instale [Docker Desktop](https://www.docker.com/get-started) en su equipo. Para probar un trabajador personalizado, ejecute el siguiente comando en la raíz de la aplicación:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Esto ejecuta una estructura de prueba de unidad personalizada para las acciones de aplicación de Asset compute en el proyecto como se describe a continuación. Se vincula a través de una configuración en el archivo `package.json`. También es posible realizar pruebas unitarias de JavaScript, como Jest. `aio app test` ejecuta ambos.

El complemento [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) está incrustado como dependencia de desarrollo en la aplicación de aplicación personalizada, de modo que no sea necesario instalarlo en sistemas de compilación/prueba.

### Marco de pruebas de la unidad de aplicación {#unit-test-framework}

El marco de prueba de la unidad de aplicación de Asset compute permite probar las aplicaciones sin escribir ningún código. Se basa en el principio de las aplicaciones de origen a archivo de representación. Se debe configurar una estructura de archivos y carpetas determinada para definir casos de prueba con archivos de origen de prueba, parámetros opcionales, representaciones esperadas y secuencias de comandos de validación personalizadas. De forma predeterminada, las representaciones se comparan con la igualdad de bytes. Además, los servicios HTTP externos se pueden burlar fácilmente usando archivos JSON simples.

### Agregar pruebas {#add-tests}

Se esperan pruebas dentro de la carpeta `test` en el nivel raíz del proyecto [!DNL Adobe I/O]. Los casos de prueba de cada aplicación deben estar en la ruta `test/asset-compute/<worker-name>`, con una carpeta para cada caso de prueba:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Eche un vistazo a [ejemplo de aplicaciones personalizadas](https://github.com/adobe/asset-compute-example-workers/) para ver algunos ejemplos. A continuación se muestra una referencia detallada.

### Prueba de salida {#test-output}

Los resultados detallados de la prueba, incluidos los registros de la aplicación personalizada, están disponibles en la carpeta `build` en la raíz de la aplicación Firefly, como se muestra en la salida `aio app test` .

### Maquetación de servicios externos {#mock-external-services}

Es posible burlarse de las llamadas de servicio externas en sus acciones definiendo archivos `mock-<HOST_NAME>.json` en sus casos de prueba, donde HOST_NAME es el host que desea burlar. Un caso de uso de ejemplo es una aplicación que realiza una llamada separada a S3. La nueva estructura de prueba tendría este aspecto:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

El archivo de prueba es una respuesta http con formato JSON. Para obtener más información, consulte [esta documentación](https://www.mock-server.com/mock_server/creating_expectations.html). Si hay varios nombres de host para burlarse, defina varios `mock-<mocked-host>.json` archivos. A continuación, se muestra un ejemplo de archivo de prueba para `google.com` llamado `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

El ejemplo `worker-animal-pictures` contiene un [archivo de prueba](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) para el servicio de Wikimedia con el que interactúa.

#### Compartir archivos en casos de prueba {#share-files-across-test-cases}

Se recomienda utilizar enlaces simbólicos relativos si se comparten scripts `file.*`, `params.json` o `validate` en varias pruebas. Son compatibles con Git. Asegúrese de asignar un nombre único a los archivos compartidos, ya que es posible que tenga otros diferentes. En el ejemplo siguiente, las pruebas están mezclando y coincidiendo con algunos archivos compartidos, y los suyos propios:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Prueba de errores esperados {#test-unexpected-errors}

Los casos de pruebas de error no deben contener un archivo `rendition.*` esperado y deben definir el `errorReason` esperado dentro del archivo `params.json`.

>[!NOTE]
>
>Si un caso de prueba no contiene un archivo `rendition.*` esperado y no define el `errorReason` esperado dentro del archivo `params.json`, se supone que es un caso de error con cualquier `errorReason`.

Estructura del caso de prueba de error:

```json
<error_test_case>/
    file.jpg
    params.json
```

Archivo de parámetro con motivo de error:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Consulte la lista y descripción completa de [motivos de error de Asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depurar una aplicación personalizada {#debug-custom-worker}

Los siguientes pasos muestran cómo puede depurar la aplicación personalizada con código de Visual Studio. Permite ver registros en vivo, puntos de interrupción de visitas y pasos a través del código, así como la recarga en vivo de cambios de código local con cada activación.

Muchos de estos pasos suelen automatizarse con `aio` fuera de la caja, consulte la sección Depuración de la aplicación en la [Documentación de Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Por ahora, los pasos siguientes incluyen una solución alternativa.

1. Instale el [wskdebug](https://github.com/apache/openwhisk-wskdebug) más reciente desde GitHub y el [ngrok](https://www.npmjs.com/package/ngrok) opcional.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Agregue al archivo JSON de configuración de usuario. Sigue usando el antiguo depurador de código VS, el nuevo tiene [algunos problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Cierre todas las instancias de aplicaciones abiertas mediante `aio app run`.
1. Implemente el código más reciente mediante `aio app deploy`.
1. Ejecute sólo la herramienta de desarrollo de Asset compute utilizando `aio asset-compute devtool`. Manténgalo abierto.
1. En el Editor de código VS, agregue la siguiente configuración de depuración a su `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Obtenga el `ACTION NAME` de la salida de `aio app deploy`.

1. Seleccione `wskdebug worker` en la configuración de ejecución/depuración y pulse el icono de reproducción. Espere a que se inicie hasta que aparezca **[!UICONTROL Ready for activations]** en la ventana **[!UICONTROL Debug Console]**.

1. Haga clic en **[!UICONTROL run]** en la herramienta de desarrollo. Puede ver las acciones que se ejecutan en el editor de código VS y los registros empiezan a mostrarse.

1. Establezca un punto de interrupción en el código, vuelva a ejecutarse y debe activarse.

Cualquier cambio en el código se carga en tiempo real y es efectivo en cuanto se produce la siguiente activación.

>[!NOTE]
>
>Hay dos activaciones para cada solicitud en aplicaciones personalizadas. La primera solicitud es una acción web que se llama a sí misma asincrónicamente en el código SDK. La segunda activación es la que entra en el código.
