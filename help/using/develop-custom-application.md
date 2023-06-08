---
title: Desarrollar para [!DNL Asset Compute Service]
description: Creación de aplicaciones personalizadas con [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '1618'
ht-degree: 0%

---

# Desarrollar una aplicación personalizada {#develop}

Antes de empezar a desarrollar una aplicación personalizada:

* Asegúrese de que todas las [requisitos previos](/help/using/understand-extensibility.md#prerequisites-and-provisioning) se cumplen.
* Instale el [herramientas de software necesarias](/help/using/setup-environment.md#create-dev-environment).
* Consulte [configurar su entorno](setup-environment.md) para asegurarse de que está listo para crear una aplicación personalizada.

## Creación de una aplicación personalizada {#create-custom-application}

Asegúrese de tener el [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) instalado localmente.

1. Para crear una aplicación personalizada, [crear un proyecto de App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Para ello, ejecute `aio app init <app-name>` en su terminal.

   Si aún no ha iniciado sesión, este comando solicitará al explorador que inicie sesión en [Consola de Adobe Developer](https://console.adobe.io/) con su Adobe ID. Consulte [aquí](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) para obtener más información sobre cómo iniciar sesión desde cli.

   El Adobe recomienda iniciar sesión. Si tiene problemas, siga las instrucciones [para crear una aplicación sin iniciar sesión](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Después de iniciar sesión, siga las indicaciones de la CLI y seleccione la `Organization`, `Project`, y `Workspace` para utilizar en la aplicación. Elija el proyecto y el espacio de trabajo que creó al [configurar su entorno](setup-environment.md). Cuando se le solicite `Which extension point(s) do you wish to implement ?`, asegúrese de seleccionar `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Cuando se le solicite con `Which Adobe I/O App features do you want to enable for this project?`, seleccione `Actions`. Asegúrese de anular la selección `Web Assets` como recursos web utilizan diferentes comprobaciones de autenticación y autorización.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Cuando se le solicite `Which type of sample actions do you want to create?`, asegúrese de seleccionar `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Siga el resto de las indicaciones y abra la nueva aplicación en Visual Studio Code (o su editor de código favorito). Contiene el andamiaje y el código de ejemplo de una aplicación personalizada.

   Lea aquí acerca de [componentes principales de una aplicación de App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   La aplicación de plantilla aprovecha nuestras [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) para cargar, descargar y organizar representaciones de aplicaciones, de modo que los desarrolladores solo necesiten implementar la lógica de aplicación personalizada. Dentro de `actions/<worker-name>` carpeta, la `index.js` es donde se agrega el código de aplicación personalizado.

Consulte [aplicaciones personalizadas de ejemplo](#try-sample) para ver ejemplos e ideas de aplicaciones personalizadas.

### Añadir credenciales {#add-credentials}

Al iniciar sesión al crear la aplicación, la mayoría de las credenciales del Generador de aplicaciones se recopilan en el archivo ENV. Sin embargo, el uso de la herramienta para desarrolladores requiere credenciales adicionales.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenciales de almacenamiento de herramientas para desarrolladores {#developer-tool-credentials}

Herramienta para desarrolladores utilizada para probar aplicaciones personalizadas con el [!DNL Asset Compute service] requiere un contenedor de almacenamiento en la nube para alojar archivos de prueba y para recibir y mostrar representaciones generadas por aplicaciones.

>[!NOTE]
>
>Esto es independiente del almacenamiento en la nube de [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. Solo se aplica al desarrollo y las pruebas con la herramienta para desarrolladores de Assets computes.

Asegúrese de tener acceso a un [contenedor de almacenamiento en la nube compatible](https://github.com/adobe/asset-compute-devtool#prerequisites). Varios desarrolladores pueden compartir este contenedor en distintos proyectos según sea necesario.

#### Agregar credenciales al archivo ENV {#add-credentials-env-file}

Agregue las siguientes credenciales para la herramienta de desarrollo al archivo ENV en la raíz de su proyecto de App Builder:

1. Agregue la ruta absoluta al archivo de clave privada creado al agregar servicios a su proyecto de App Builder:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Descargue el archivo desde la consola de Adobe Developer. Vaya a la raíz del proyecto y haga clic en &quot;Descargar todo&quot; en la esquina superior derecha. El archivo se descarga con `<namespace>-<workspace>.json` como nombre de archivo. Realice una de las siguientes acciones:

   * Cambie el nombre del archivo como `console.json` y muévalo a la raíz de su proyecto.
   * De forma opcional, puede añadir la ruta absoluta al archivo JSON de integración de la consola de Adobe Developer. Esto es lo mismo [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) que se descarga en el espacio de trabajo del proyecto.

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. Agregue las credenciales de S3 o Azure Storage. Solo necesita acceder a una solución de almacenamiento en la nube.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>El `config.json` el archivo contiene credenciales. Desde el proyecto, agregue el archivo JSON a su `.gitignore` para evitar que se comparta. Lo mismo se aplica a los archivos .env y .aio.

## Ejecución de la aplicación {#run-custom-application}

Antes de ejecutar la aplicación con la Herramienta de desarrollo de Asset compute, configure correctamente [credenciales](#developer-tool-credentials).

Para ejecutar la aplicación en la herramienta para desarrolladores, utilice `aio app run` comando. Implementa la acción en [!DNL Adobe I/O] Ejecute e inicie la herramienta de desarrollo en el equipo local. Esta herramienta se utiliza para probar las solicitudes de aplicación durante el desarrollo. Este es un ejemplo de solicitud de representación:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>No use el `--local` Indicador con el `run` comando. No funciona con [!DNL Asset Compute] aplicaciones personalizadas y la herramienta para desarrolladores de Asset compute. Las aplicaciones personalizadas son invocadas por [!DNL Asset Compute Service] que no puede acceder a las acciones que se ejecutan en los equipos locales del desarrollador.

Consulte [aquí](test-custom-application.md) Obtenga información sobre cómo probar y depurar la aplicación. Cuando haya terminado de desarrollar la aplicación personalizada, [implementar la aplicación personalizada](deploy-custom-application.md).

## Pruebe la aplicación de ejemplo proporcionada por Adobe. {#try-sample}

A continuación se muestran ejemplos de aplicaciones personalizadas:

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-picture](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aplicación personalizada de plantilla {#template-custom-application}

El [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) es una aplicación de plantilla. Genera una representación simplemente copiando el archivo de origen. El contenido de esta aplicación es la plantilla recibida al elegir `Adobe Asset Compute` en la creación de la aplicación aio.

El archivo de la aplicación, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utiliza el [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) para descargar el archivo de origen, organice cada procesamiento de representación y cargue las representaciones resultantes de nuevo en el almacenamiento en la nube.

El [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definido dentro del código de la aplicación, es donde se debe realizar toda la lógica de procesamiento de la aplicación. La llamada de retorno de representación en `worker-basic` simplemente copia el contenido del archivo de origen en el archivo de representación.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Llamar a una API externa {#call-external-api}

En el código de la aplicación, puede realizar llamadas de API externas para ayudar con el procesamiento de la aplicación. A continuación se muestra un ejemplo de archivo de aplicación que invoca una API externa.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Por ejemplo, la variable [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) realiza una solicitud de captura a una URL estática desde Wikimedia usando el [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) biblioteca.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Pasar parámetros personalizados {#pass-custom-parameters}

Puede pasar parámetros definidos personalizados a través de los objetos de representación. Se puede hacer referencia a ellos dentro de la aplicación en [`rendition` instrucciones](https://github.com/adobe/asset-compute-sdk#rendition). Un ejemplo de objeto de representación es el siguiente:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Un ejemplo de archivo de aplicación que accede a un parámetro personalizado es:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

El `example-worker-animal-pictures` pasa un parámetro personalizado [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) para determinar qué archivo obtener de Wikimedia.

## Compatibilidad con autenticación y autorización {#authentication-authorization-support}

De forma predeterminada, las aplicaciones personalizadas de Asset compute incluyen comprobaciones de autorización y autenticación para el proyecto de App Builder. Esto se habilita configurando la variable `require-adobe-auth` anotación a `true` en el `manifest.yml`.

### Acceso a otras API de Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Agregue los servicios de API a [!DNL Asset Compute] Espacio de trabajo de consola creado en la configuración. Estos servicios forman parte del token de acceso JWT generado por [!DNL Asset Compute Service]. Se puede acceder al token y a otras credenciales dentro de la acción de la aplicación `params` objeto.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Pasar credenciales para sistemas de terceros {#pass-credentials-for-tp}

Para gestionar las credenciales de otros servicios externos, páselas como parámetros predeterminados en las acciones. Se cifran automáticamente en tránsito. Para obtener más información, consulte [creación de acciones en la guía para desarrolladores de Runtime](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). A continuación, configúrelos mediante variables de entorno durante la implementación. Se puede acceder a estos parámetros en la variable `params` objeto dentro de la acción.

Establezca los parámetros predeterminados dentro de `inputs` en el `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

El `$VAR` la expresión lee el valor de una variable de entorno denominada `VAR`.

Durante el desarrollo, el valor se puede establecer en el archivo ENV local como `aio` lee automáticamente variables de entorno de archivos ENV además de las variables establecidas desde el shell que realiza la invocación. En este ejemplo, el archivo ENV tiene el siguiente aspecto:

```CONF
#...
SECRET_KEY=secret-value
```

Para la implementación de producción se pueden establecer las variables de entorno en el sistema de CI, por ejemplo, usando secretos en las acciones de GitHub. Por último, acceda a los parámetros predeterminados dentro de la aplicación como tal:

```javascript
const key = params.secretKey;
```

## Tamaño de aplicaciones {#sizing-workers}

Una aplicación se ejecuta en un contenedor en [!DNL Adobe I/O] Runtime con [límites](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) que se puede configurar mediante la variable `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Debido al procesamiento más extenso que suelen realizar las aplicaciones de Asset compute, es más probable que haya que ajustar estos límites para obtener un rendimiento óptimo (lo suficientemente grande como para gestionar recursos binarios) y una mayor eficacia (sin desperdiciar recursos debido a la memoria contenedora no utilizada).

El tiempo de espera predeterminado para las acciones en tiempo de ejecución es de un minuto, pero se puede aumentar configurando la variable `timeout` límite (en milisegundos). Si espera procesar archivos más grandes, aumente este tiempo. Considere el tiempo total que se tarda en descargar el origen, procesar el archivo y cargar la representación. Si se agota el tiempo de espera de una acción, es decir, no devuelve la activación antes del límite de tiempo de espera especificado, Runtime descarta el contenedor y no lo vuelve a utilizar.

Las aplicaciones de asset compute, por naturaleza, tienden a estar enlazadas a la red y a la entrada o salida del disco. El archivo de origen debe descargarse primero, el procesamiento suele consumir muchos recursos y las representaciones resultantes se cargan de nuevo.

La memoria disponible para un contenedor de acciones se especifica mediante `memorySize` en MB. Actualmente, esto también define cuánto acceso de CPU obtiene el contenedor y, lo que es más importante, es un elemento clave del coste de utilizar Runtime (los contenedores más grandes cuestan más). Utilice un valor mayor aquí cuando el procesamiento requiera más memoria o CPU, pero tenga cuidado de no desperdiciar recursos, ya que cuanto más grandes sean los contenedores, menor será el rendimiento general.

Además, es posible controlar la concurrencia de acciones dentro de un contenedor mediante `concurrency` configuración. Número de activaciones simultáneas que obtiene un solo contenedor (de la misma acción). En este modelo, el contenedor de acciones es como un servidor Node.js que recibe varias solicitudes simultáneas, hasta ese límite. Si no se establece, el valor predeterminado en Tiempo de ejecución es 200, que es bueno para acciones más pequeñas del Generador de aplicaciones, pero normalmente es demasiado grande para las aplicaciones de Asset compute, dado que su procesamiento local y actividad de disco son más intensos. Según su implementación, es posible que algunas aplicaciones no funcionen bien con la actividad simultánea. El SDK de Asset compute garantiza que las activaciones estén separadas por la escritura de archivos en diferentes carpetas únicas.

Probar aplicaciones para encontrar los números óptimos de `concurrency` y `memorySize`. Contenedores más grandes = un límite de memoria más alto podría permitir una mayor concurrencia, pero también podría ser un derroche para un tráfico más bajo.
