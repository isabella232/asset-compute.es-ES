---
title: Desarrollar para [!DNL Asset Compute Service].
description: Cree aplicaciones personalizadas mediante [!DNL Asset Compute Service].
translation-type: tm+mt
source-git-commit: 127895cf1bab59546f9ba0be2b3b7a935627effb
workflow-type: tm+mt
source-wordcount: '1496'
ht-degree: 0%

---


# Desarrollar una aplicación personalizada {#develop}

Antes de comenzar a desarrollar una aplicación personalizada:

* Asegúrese de que se cumplen todos los [requisitos previos](/help/understand-extensibility.md#prerequisites-and-provisioning) .
* Instale las herramientas [de software](/help/setup-environment.md#create-dev-environment)necesarias.
* Consulte [Configuración del entorno](setup-environment.md) para asegurarse de que está preparado para crear una aplicación personalizada.

## Creación de una aplicación personalizada {#create-custom-application}

Asegúrese de tener la CLI [de E/S de](https://github.com/adobe/aio-cli) Adobe instalada localmente.

1. Para crear una aplicación personalizada, [cree una aplicación](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)de Firefly. Para hacerlo, ejecute `aio app init <app-name>` en su terminal.

   Si aún no ha iniciado sesión, este comando solicita a un navegador que le pida que inicie sesión en la consola [de desarrollador de](https://console.adobe.io/) Adobe con su Adobe ID. Consulte [aquí](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli) para obtener más información sobre el inicio de sesión desde el cli.

   Adobe recomienda iniciar sesión. Si tiene problemas, siga las instrucciones [para crear una aplicación sin iniciar sesión](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Después de iniciar sesión, siga las indicaciones de la CLI y seleccione la opción `Organization`, `Project`y `Workspace` utilizar para la aplicación. Elija el proyecto y el espacio de trabajo que creó al [configurar el entorno](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from Adobe I/O Console..
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. Cuando se le pregunte con `Which Adobe I/O App features do you want to enable for this project?`, seleccione al menos `Actions`:

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Cuando se le pregunte `Which type of sample actions do you want to create?`, asegúrese de seleccionar `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Siga el resto de las indicaciones y abra la nueva aplicación en el código de Visual Studio (o en su editor de código favorito). Contiene el andamiaje y el código de muestra de una aplicación personalizada.

   Lea aquí los componentes [principales de una aplicación](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)de Firefly.

   La aplicación de plantilla aprovecha el SDK [de cómputo de](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) recursos para cargar, descargar y orquestar representaciones de aplicaciones, de modo que los desarrolladores sólo necesitan implementar la lógica de aplicación personalizada. Dentro de la `actions/<worker-name>` carpeta, el `index.js` archivo es donde se agrega el código de aplicación personalizado.

Consulte [ejemplo de aplicaciones](#try-sample) personalizadas para ver ejemplos e ideas de aplicaciones personalizadas.

### Añadir credenciales {#add-credentials}

Al iniciar sesión al crear la aplicación, la mayoría de las credenciales de Firefly se recopilan en el archivo ENV. Sin embargo, el uso de la herramienta para desarrolladores requiere credenciales adicionales.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenciales del almacenamiento de herramientas para desarrolladores {#developer-tool-credentials}

La herramienta para desarrolladores que se utiliza para probar aplicaciones personalizadas con el almacenamiento real [!DNL Asset Compute service] requiere un contenedor de nube para alojar archivos de prueba y para recibir y mostrar representaciones generadas por las aplicaciones.

>[!NOTE]
>
>Esto es independiente del almacenamiento de nube de [!DNL Adobe Experience Manager] como Cloud Service. Solo se aplica para desarrollar y probar con la herramienta para desarrolladores de Asset Compute.

Asegúrese de tener acceso a un contenedor [de almacenamiento en la nube](https://github.com/adobe/asset-compute-devtool#prerequisites)compatible. Este contenedor puede ser compartido por varios desarrolladores en diferentes proyectos según sea necesario.

#### Añadir credenciales al archivo ENV {#add-credentials-env-file}

Añada las siguientes credenciales para la herramienta para desarrolladores en el archivo ENV en la raíz del proyecto de Firefly:

1. Añada la ruta absoluta al archivo de clave privada creado al agregar servicios al proyecto de luciérnagas:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Añada las credenciales de Almacenamiento de S3 o Azure. Solo necesita acceder a una solución de almacenamiento en la nube.

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

## Ejecutar la aplicación {#run-custom-application}

Antes de ejecutar la aplicación con Asset Compute Developer Tool, configure correctamente las [credenciales](#developer-tool-credentials).

Para ejecutar la aplicación en la herramienta para desarrolladores, utilice `aio app run` command. Implementa la acción en Adobe I/O Runtime y inicio la herramienta de desarrollo en su equipo local. Esta herramienta se utiliza para probar las solicitudes de aplicación durante el desarrollo. A continuación se muestra un ejemplo de solicitud de representación:

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
>No utilice el `--local` indicador con el `run` comando. No funciona con las aplicaciones personalizadas ni con [!DNL Asset Compute] la herramienta para desarrolladores de Asset Compute. Las aplicaciones personalizadas son llamadas por los [!DNL Asset Compute Service] que no pueden acceder a las acciones que se ejecutan en los equipos locales del desarrollador.

Consulte [aquí](test-custom-application.md) cómo probar y depurar la aplicación. Cuando haya terminado de desarrollar la aplicación personalizada, [implemente la aplicación](deploy-custom-application.md)personalizada.

## Pruebe la aplicación de ejemplo proporcionada por Adobe {#try-sample}

Las siguientes son aplicaciones personalizadas de ejemplo:

* [trabajador básico](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-images](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aplicación personalizada de plantilla {#template-custom-application}

El [programa básico](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) de trabajo es una aplicación de plantilla. Genera una representación simplemente copiando el archivo de origen. El contenido de esta aplicación es la plantilla que se recibe al elegir `Adobe Asset Compute` en la creación de la aplicación de AIO.

El archivo de la aplicación [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utiliza el [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) para descargar el archivo de origen, orquestar cada procesamiento de representación y cargar las representaciones resultantes de nuevo en el almacenamiento de la nube.

El [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) valor definido dentro del código de la aplicación es dónde realizar toda la lógica de procesamiento de la aplicación. La rellamada de representación `worker-basic` simplemente copia el contenido del archivo de origen en el archivo de representación.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Llamar a una API externa {#call-external-api}

En el código de la aplicación, puede realizar llamadas de API externas para ayudar en el procesamiento de la aplicación. A continuación se muestra un archivo de aplicación de ejemplo que invoca una API externa.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Por ejemplo, [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) realiza una solicitud de captura a una URL estática de Wikimedia mediante la [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) biblioteca.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Pasar parámetros personalizados {#pass-custom-parameters}

Puede pasar parámetros definidos personalizados a través de los objetos de representación. Se puede hacer referencia a ellos dentro de la aplicación en [`rendition` instrucciones](https://github.com/adobe/asset-compute-sdk#rendition). Un ejemplo de objeto de representación es:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Un ejemplo de un archivo de aplicación que accede a un parámetro personalizado es:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

El `example-worker-animal-pictures` pasa un parámetro personalizado [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) para determinar qué archivo se va a recuperar de Wikimedia.

## Compatibilidad con autenticación y autorización {#authentication-authorization-support}

De forma predeterminada, las aplicaciones personalizadas de Asset Compute incluyen comprobaciones de autorización y autenticación para aplicaciones de Firefly. Esto se habilita estableciendo la `require-adobe-auth` anotación en `true` la `manifest.yml`.

### Acceso a otras API de Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Añada los servicios de API al espacio de trabajo de la [!DNL Asset Compute] consola creado en la configuración. Estos servicios forman parte del token de acceso JWT generado por [!DNL Asset Compute Service]. Se puede acceder al token y a otras credenciales dentro del objeto de acción de la aplicación `params` .

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Pasar credenciales para sistemas de terceros {#pass-credentials-for-tp}

Para gestionar las credenciales de otros servicios externos, pase estas credenciales como parámetros predeterminados en las acciones. Estos se cifran automáticamente en tránsito. Para obtener más información, consulte [Creación de acciones en la guía](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)para desarrolladores de Runtime. A continuación, configúrelos mediante variables de entorno durante la implementación. Se puede acceder a estos parámetros en el `params` objeto dentro de la acción.

Defina los parámetros predeterminados dentro del `inputs` en la `manifest.yml`:

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

La `$VAR` expresión lee el valor de una variable de entorno denominada `VAR`.

Durante el desarrollo, el valor puede configurarse en el archivo ENV local como `aio` lee automáticamente las variables de entorno de los archivos ENV además de las variables configuradas desde el shell invocante. En este ejemplo, el archivo ENV presenta el siguiente aspecto:

```CONF
#...
SECRET_KEY=secret-value
```

Para la implementación de producción, se pueden establecer las variables de entorno en el sistema CI, por ejemplo, usando secretos en Acciones de GitHub. Por último, acceda a los parámetros predeterminados dentro de la aplicación como tales:

```javascript
const key = params.secretKey;
```

## Cambio de tamaño de aplicaciones {#sizing-workers}

Una aplicación se ejecuta en un contenedor de Adobe I/O Runtime con [límites](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) que se pueden configurar mediante el `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Debido al procesamiento más extenso que suelen realizar las aplicaciones de Asset Compute, es más probable que uno tenga que ajustar estos límites para obtener un rendimiento óptimo (lo suficientemente grande como para gestionar recursos binarios) y eficacia (no desperdiciar recursos debido a la memoria contenedor no utilizada).

El tiempo de espera predeterminado para acciones en tiempo de ejecución es de un minuto, pero se puede aumentar estableciendo el `timeout` límite (en milisegundos). Si espera procesar archivos más grandes, aumente esta vez. Considere el tiempo total que se tarda en descargar el origen, procesar el archivo y cargar la representación. Si se agota el tiempo de espera de una acción, es decir, no devuelve la activación antes del límite de tiempo de espera especificado, Runtime descarta el contenedor y no lo reutiliza.

Las aplicaciones de cálculo de recursos por naturaleza tienden a estar enlazadas a E/S de red y de disco. El archivo de origen debe descargarse primero, el procesamiento suele ser de alta E/S y, a continuación, las representaciones resultantes se cargan de nuevo.

La memoria disponible para un contenedor de acción se especifica `memorySize` en MB. Actualmente, esto también define cuánto acceso de CPU obtiene el contenedor y, lo que es más importante, es un elemento clave del costo de usar Runtime (los contenedores más grandes cuestan más). Utilice aquí un valor mayor cuando el procesamiento requiera más memoria o CPU, pero tenga cuidado de no desperdiciar recursos, ya que cuanto más grandes sean los contenedores, menor será el rendimiento general.

Además, es posible controlar la concurrencia de acciones dentro de un contenedor mediante la `concurrency` configuración. Es el número de activaciones simultáneas que obtiene un solo contenedor (de la misma acción). En este modelo, el contenedor de acción es como un servidor Node.js que recibe varias solicitudes simultáneas, hasta ese límite. Si no se establece, el valor predeterminado en Tiempo de ejecución es 200, que es bueno para acciones de Firefly más pequeñas, pero generalmente es demasiado grande para las aplicaciones de Asset Compute debido a que su procesamiento local y actividad de disco son más intensos. Algunas aplicaciones, dependiendo de su implementación, también podrían no funcionar bien con la actividad simultánea. El SDK de cómputo de recursos garantiza que las activaciones se separan escribiendo archivos en distintas carpetas únicas.

Pruebe las aplicaciones para encontrar los números óptimos para `concurrency` y `memorySize`. Contenedores más grandes = límite de memoria más alto podría permitir más concurrencia pero también podría ser derrochador para tráfico más bajo.
