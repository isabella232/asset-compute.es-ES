---
title: Probar y depurar [!DNL Asset Compute Service] aplicación personalizada.
description: Probar y depurar [!DNL Asset Compute Service] aplicación personalizada.
translation-type: tm+mt
source-git-commit: 7e520921ebb459c963d61d70c66497b8e62521cf
workflow-type: tm+mt
source-wordcount: '787'
ht-degree: 0%

---


# Probar y depurar una aplicación personalizada {#test-debug-custom-worker}

## Ejecutar pruebas unitarias para una aplicación personalizada {#test-custom-worker}

Instale [Docker Desktop](https://www.docker.com/get-started) en su equipo. Para probar un programa de trabajo personalizado, ejecute el siguiente comando en la raíz de la aplicación:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Esto ejecuta una estructura de prueba de unidad personalizada para las acciones de aplicación de Asset compute en el proyecto como se describe a continuación. Se conecta mediante una configuración en el archivo `package.json`. También es posible realizar pruebas unitarias de JavaScript como Jest. `aio app test` ejecuta ambos.

El complemento [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) está incrustado como dependencia de desarrollo en la aplicación de aplicación personalizada, de modo que no es necesario instalarlo en sistemas de compilación/prueba.

### Marco de pruebas de la unidad de aplicación {#unit-test-framework}

El módulo de prueba de la unidad de aplicación de Asset compute permite probar las aplicaciones sin escribir ningún código. Se basa en el principio de las aplicaciones del archivo de representación de origen. Se debe configurar una determinada estructura de archivos y carpetas para definir casos de prueba con archivos de origen de prueba, parámetros opcionales, representaciones esperadas y secuencias de comandos de validación personalizadas. De forma predeterminada, las representaciones se comparan para la igualdad de bytes. Además, los servicios HTTP externos se pueden burlar fácilmente con archivos JSON sencillos.

### Añadir pruebas {#add-tests}

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

Consulte [ejemplos de aplicaciones personalizadas](https://github.com/adobe/asset-compute-example-workers/) para ver algunos ejemplos. A continuación se presenta una referencia detallada.

### Resultados de la prueba {#test-output}

El resultado detallado de la prueba, incluidos los registros de la aplicación personalizada, está disponible en la carpeta `build` en la raíz de la aplicación Firefly, como se muestra en la salida `aio app test`.

### Conversión de servicios externos {#mock-external-services}

Es posible burlarse de las llamadas de servicio externas en sus acciones mediante la definición de archivos `mock-<HOST_NAME>.json` en los casos de prueba, donde HOST_NAME es el host que desea burlar. Un caso de uso de ejemplo es una aplicación que realiza una llamada por separado a S3. La nueva estructura de prueba tendría este aspecto:

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

El archivo de prueba es una respuesta http con formato JSON. Para obtener más información, consulte [esta documentación](https://www.mock-server.com/mock_server/creating_expectations.html). Si hay varios nombres de host para burlarse, defina varios `mock-<mocked-host>.json` archivos. A continuación se muestra un archivo de muestra para `google.com` denominado `mock-google.com.json`:

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

#### Compartir archivos en los casos de prueba {#share-files-across-test-cases}

Se recomienda utilizar enlaces simbólicos relativos si comparte `file.*`, `params.json` o `validate` secuencias de comandos en varias pruebas. Son compatibles con git. Asegúrese de asignar un nombre único a los archivos compartidos, ya que podría tener otros. En el ejemplo que se muestra a continuación, las pruebas combinan y coinciden con algunos archivos compartidos, y con los suyos propios:

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

### Probar errores esperados {#test-unexpected-errors}

Los casos de pruebas de error no deben contener un archivo `rendition.*` esperado y deben definir el `errorReason` esperado dentro del archivo `params.json`.

Estructura de caso de prueba de error:

```json
<error_test_case>/
    file.jpg
    params.json
```

Archivo de parámetros con motivo de error:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Consulte la lista completa y la descripción de [motivos de error de Asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depurar una aplicación personalizada {#debug-custom-worker}

Los siguientes pasos muestran cómo puede depurar la aplicación personalizada con código de Visual Studio. Permite ver los registros en vivo, los puntos de interrupción de visitas y el código de paso a paso, así como la recarga en vivo de los cambios de código local en cada activación.

Muchos de estos pasos generalmente se automatizan `aio` de forma predeterminada. Consulte la sección Depuración de la aplicación en la [documentación de la búsqueda](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md). Por ahora, los siguientes pasos incluyen una solución alternativa.

1. Instale el [wskdebug](https://github.com/apache/openwhisk-wskdebug) más reciente desde GitHub y el [ngrok](https://www.npmjs.com/package/ngrok) opcional.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Añada al archivo JSON de configuración de usuario. Sigue usando el antiguo depurador de códigos VS, el nuevo tiene [algunos problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Cierre todas las instancias de aplicaciones abiertas mediante `aio app run`.
1. Implemente el código más reciente mediante `aio app deploy`.
1. Ejecute sólo la herramienta Devolución de Asset compute mediante `npx adobe-asset-compute devtool`. Manténgalo abierto.
1. En el editor de código VS, agregue la configuración de depuración siguiente a su `launch.json`:

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

   Obtenga el NOMBRE DE ACCIÓN del resultado de `aio app deploy`. Parece `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`.

1. Seleccione `wskdebug worker` en la configuración de ejecución/depuración y pulse el icono de reproducción. Espere hasta que se muestre **[!UICONTROL Listo para activaciones]** en la ventana **[!UICONTROL Consola de depuración]**.

1. Haga clic en **[!UICONTROL ejecutar]** en Devtool. Puede ver las acciones que se ejecutan en el editor de código VS y el inicio de registros que se muestra.

1. Configure un punto de interrupción en el código, vuelva a ejecutarlo y debería activarlo.

Cualquier cambio en el código se carga en tiempo real y es efectivo en cuanto se produce la siguiente activación.

>[!NOTE]
>
>Existen dos activaciones para cada solicitud en las aplicaciones personalizadas. La primera solicitud es una acción web que se llama asincrónicamente en el código del SDK. La segunda activación es la que entra en el código.
