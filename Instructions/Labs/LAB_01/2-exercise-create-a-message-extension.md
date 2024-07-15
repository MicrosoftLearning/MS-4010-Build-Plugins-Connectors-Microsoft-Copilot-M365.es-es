---
lab:
  title: 'Ejercicio 1: Creación de una extensión de mensajes'
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 1: Creación de una extensión de mensajes

En este ejercicio, crearás una extensión de mensajes con un comando de búsqueda. Primero aplicarás scaffolding a un proyecto con una plantilla de proyecto del kit de herramientas de Teams y luego lo actualizarás para configurarlo mediante un recurso del Servicio de Bot de Azure AI para el desarrollo local. Crearás un túnel de desarrollo para habilitar la comunicación entre el servicio de bot y el servicio web que ejecutas localmente. Luego prepararás la aplicación para aprovisionar los recursos necesarios. Por último, ejecutarás y depurarás la extensión de mensajes y la pruebas en Microsoft Teams.

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams." lightbox="../media/2-search-results-nuget.png":::

## Tarea 1: Creación de un nuevo proyecto con el kit de herramientas de Teams para Visual Studio

Empieza creando un nuevo proyecto.

1. Abre **Visual Studio 2022**
1. Abre el menú **Archivo** y expande el menú **Nuevo**, selecciona **Nuevo proyecto**.
1. En la pantalla Crear un nuevo proyecto, expande el elemento desplegable **Todas las plataformas** y después selecciona **Microsoft Teams**. Selecciona **Siguiente** para continuar.
1. En la pantalla Configurar el nuevo proyecto. Especifica los valores siguientes:
    1. **Nombre de proyecto**: MsgExtProductSupport
    1. **Ubicación**: selecciona la ubicación de tu elección
    1. **Colocar la solución y el proyecto en el mismo directorio**: marcado
1. Selecciona **Crear** para aplicar scaffolding al proyecto
1. En el cuadro de diálogo Crear una nueva aplicación de Teams, expande el elemento desplegable **Todos los tipos de aplicaciones** y luego selecciona **Extensión de mensajes**
1. En la lista de plantillas, selecciona **Resultados de búsqueda personalizada**
1. Selecciona **Crear** para aplicar scaffolding a la aplicación

## Tarea 2: Configuración del Servicio de Bot de Azure AI

Se puede crear un recurso de servicio de bot en Azure como recurso o a través de dev.botframework.com. De forma predeterminada, la plantilla de resultados de búsqueda personalizada registra un bot mediante dev.botframework.com. En la actualidad, el registro del bot con dev.botframework.com no es compatible con Copilot para Microsoft 365.

Para que sea compatible con Copilot para Microsoft 365, actualiza el proyecto para aprovisionar un recurso del Servicio de Bot de Azure AI en Azure y usarlo para el desarrollo local.

Primero vamos a crear una variable de entorno para centralizar un nombre interno para la aplicación que podemos reutilizar en nuestros archivos y usar al aprovisionar recursos.

En Visual Studio:

1. En la carpeta **env**, abre **.env.local**
1. En el archivo agrega el código siguiente:

    ```text
    APP_INTERNAL_NAME=msgext-product-support
    ```

1. Guarda los cambios

Las expresiones de enlace de datos se usan, como `${{APP_INTERNAL_NAME}}`, que permite insertar valores de variables de entorno en archivos cuando se usa el kit de herramientas de Teams para aprovisionar recursos.

Para aprovisionar un recurso del Servicio de Bot de Azure AI, se requiere el registro de aplicación de Microsoft Entra. Crea un archivo de manifiesto de registro de aplicación que el kit de herramientas de Teams use para aprovisionar el registro de aplicación.

Continúa en Visual Studio:

1. En la carpeta **infra**, crea una nueva carpeta denominada **entra**.
1. En la carpeta crea un archivo denominado **entra.bot.manifest.json**
1. En el archivo agrega el código siguiente:

    ```json
    {
      "id": "${{BOT_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{BOT_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": []
    }
    ```

1. Guarda los cambios.

El kit de herramientas de Teams usa archivos Bicep para aprovisionar y configurar los recursos en Azure. Primero crea un archivo de parámetros. El archivo de parámetros se usa para pasar variables de entorno a una plantilla Bicep.

Continúa en Visual Studio:

1. En la carpeta **infra**, crea un nuevo archivo denominado **azure.parameters.local.json**
1. En el archivo agrega el código siguiente:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        }
      }
    }
    ```

1. Guarda los cambios.

Ahora, crea un archivo Bicep que se use con el archivo de parámetros.

1. En la carpeta **infra**, crea un nuevo archivo denominado **azure.local.bicep**
1. En el archivo agrega el código siguiente:

    ```bicep
    @maxLength(20)
    @minLength(4)
    @description('Used to generate names for all resources in this file')
    param resourceBaseName string
    
    @description('Required when create Azure Bot service')
    param botEntraAppClientId string
    @maxLength(42)
    param botDisplayName string
    param botAppDomain string
    
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
      }
    }
    ```

1. Guarda los cambios.

El último paso es actualizar el archivo del proyecto del kit de herramientas de Teams. Reemplaza los pasos que usan acciones de Bot Framework para aprovisionar el registro de aplicación de Microsoft Entra del bot mediante el archivo de manifiesto y el recurso del Servicio de Bot de Azure AI con el archivo Bicep.

Continúa en Visual Studio:

1. En la carpeta raíz del proyecto, abre **teamsapp.local.yml**
1. En el archivo, busca el paso que usa la acción **botAadApp/create** y remplázala por:

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: BOT_ID
          clientSecret: SECRET_BOT_PASSWORD
          objectId: BOT_ENTRA_APP_OBJECT_ID
          tenantId: BOT_ENTRA_APP_TENANT_ID
          authority: BOT_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: BOT_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.bot.manifest.json"
          outputFilePath : "./build/entra.bot.manifest.${{TEAMSFX_ENV}}.json"
    
      - uses: arm/deploy
        with:
          subscriptionId: ${{AZURE_SUBSCRIPTION_ID}}
          resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}}
          templates:
            - path: ./infra/azure.local.bicep
              parameters: ./infra/azure.parameters.local.json
              deploymentName: Create-resources-for-${{APP_INTERNAL_NAME}}-${{TEAMSFX_ENV}}
          bicepCliVersion: v0.9.1
    ```

1. En el archivo, quita el paso que usa la acción **botFramework/create**
1. Guarda los cambios.

El registro de aplicación se aprovisiona en dos pasos, primero la acción **aadApp/create** crea un nuevo registro de aplicación multiinquilino con un secreto de cliente, escribiendo sus salidas en el archivo **.env.local** como variables de entorno. Después la acción **aadApp/update** usa el archivo **entra.bot.manifest.json** para actualizar el registro de aplicación.

El último paso usa la acción **arm/deploy** para aprovisionar el recurso del Servicio de Bot de Azure AI en el grupo de recursos mediante el archivo **azure.parameters.local.json** y el archivo **azure.local.bicep**.

## Tarea 3: Creación de un túnel de desarrollo

Cuando el usuario interactúa con la extensión de mensajes, el servicio de bot envía solicitudes al servicio web. Durante el desarrollo, el servicio web se ejecuta localmente en el equipo. Para permitir que el servicio de bot llegue al servicio web, debes exponerlo más allá del equipo con un túnel de desarrollo.

:::image type="content" source="../media/18-select-dev-tunnel.png" alt-text="Captura de pantalla del menú expandido de túneles Dev en Visual Studio." lightbox="../media/18-select-dev-tunnel.png":::

Continúa en Visual Studio:

1. En la barra de herramientas, expande el menú del perfil de depuración **seleccionando el elemento desplegable situado junto al botón Microsoft Teams (browser)**
1. Expande el menú **Dev Tunnels (no active tunnel)** y selecciona **Create a Tunnel...**
1. En el cuadro de diálogo especifica los valores siguientes:
    1. **Cuenta**: selecciona la cuenta de tu elección
    1. **Nombre**: MsgExtProductSupport
    1. **Tipo de túnel**: temporal
    1. **Acceso**: público
1. Para crear el túnel, selecciona **Aceptar**, se muestra un mensaje que indica que el nuevo túnel es ahora el túnel activo actual
1. Cierra el mensaje seleccionado **Aceptar**

## Tarea 4: Actualización del manifiesto de la aplicación

El manifiesto de la aplicación describe las características y funcionalidades de la aplicación. Actualiza las propiedades del manifiesto de la aplicación para describir mejor la funcionalidad de la aplicación y sus características.

Descarga primero los iconos de la aplicación y agrégalos al proyecto.

:::image type="content" source="../media/app/color-local.png" alt-text="Icono de color usado para el desarrollo local." lightbox="../media/app/color-local.png":::

:::image type="content" source="../media/app/color-dev.png" alt-text="Icono de color usado para el desarrollo remoto." lightbox="../media/app/color-dev.png":::

1. Descarga **color-local.png** y **color-dev.png**
1. En la carpeta **appPackage**, agrega **color-local.png** y **color-dev.png**
1. En la carpeta elimina el archivo denominado **color.png**

A medida que el nombre de la aplicación se replica en diferentes lugares del proyecto, crea una nueva variable de entorno para almacenar este valor de forma centralizada.

Continúa en Visual Studio:

1. En la carpeta **env**, abre el archivo denominado **.env.local**
1. En el archivo agrega el código siguiente:

    ```text
    APP_DISPLAY_NAME=Contoso products
    ```

1. Guarda los cambios

Por último, actualiza los iconos, el nombre y los objetos de descripción en el archivo de manifiesto de la aplicación.

1. En la carpeta **appPackage**, abre el archivo denominado **manifest.json**
1. En el archivo, actualiza los **iconos**, **name y los objetos de **descripción** con:

    ```json
    {
        "icons": {
            "color": "color-${{TEAMSFX_ENV}}.png",
            "outline": "outline.png"
        },
        "name": {
            "short": "${{APP_DISPLAY_NAME}}",
            "full": "${{APP_DISPLAY_NAME}}"
        },
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation."
        }
    }
    ```

1. Guarda los cambios

## Tarea 6: Aprovisionamiento de recursos

Ahora que está todo listo, usa el kit de herramientas de Teams para ejecutar el proceso Preparar dependencias de la aplicación de Teams para aprovisionar los recursos necesarios.

:::image type="content" source="../media/19-prepare-teams-app-dependencies.png" alt-text="Captura de pantalla que muestra el menú expandido del kit de herramientas de Teams en Visual Studio Code." lightbox="../media/19-prepare-teams-app-dependencies.png":::

El proceso de preparar dependencias de la aplicación de Teams actualiza las variables de entorno **BOT_ENDPOINT** y **BOT_DOMAIN** en el archivo .env.local mediante la dirección URL activa del túnel de desarrollo y ejecuta las acciones descritas en el archivo **teamsapp.local.yml**.

Continúa en Visual Studio:

1. En el Explorador de soluciones, haz clic con el botón derecho en el proyecto **MsgExtProductSupport**
1. Expande el menú del **kit de herramientas de teams** y selecciona **Preparar dependencias de la aplicación de Teams**
1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona la cuenta del inquilino de desarrollador y después selecciona **Continuar**
1. En el cuadro de diálogo **Provision**, selecciona la cuenta que usarás para implementar recursos en Azure y especifica los valores siguientes:
    1. **Nombre de suscripción**: selecciona la suscripción que desea usar en el elemento desplegable
    1. **Grupo de recursos**: selecciona New... para abrir un cuadro de diálogo, escribe **rg-msgext-product-support-local y después selecciona **Aceptar**
    1. **Región**: selecciona la región que tengas más cerca en el elemento desplegable
1. Selecciona **Provision** para aprovisionar los recursos en Azure
1. En el mensaje de advertencia del kit de herramientas de Teams, selecciona **Aprovisionar**
1. En el mensaje de información del kit de herramientas de Teams, selecciona **View provisioned resources** para abrir una nueva ventana del explorador.

Dedica un momento a explorar los recursos creados en Azure.

## Tarea 7: Ejecución y depuración  

Ahora inicia el servicio web y prueba la extensión de mensajes. El kit de herramientas de Teams se usa para cargar el manifiesto de la aplicación y probar la extensión de mensajes en Microsoft Teams.

Continúa en Visual Studio:

1. Presiona F5 para iniciar una sesión de depuración y abrir una nueva ventana del explorador que navegue por el cliente web de Microsoft Teams.
1. Cuando se te solicite, escribe las credenciales de la cuenta de Microsoft 365.

  > [!IMPORTANT]
  > Si aparece un cuadro de diálogo en Microsoft Teams con el mensaje "This app cannot be found", sigue los pasos siguientes para cargar el paquete de la aplicación manualmente:
  >
  >  1. Cierra el cuadro de diálogo
  >  2. Ve a **Aplicaciones** en la barra lateral
  >  3. Selecciona **Administrar las aplicaciones** en el menú de la izquierda
  >  4. Selecciona **Cargar una aplicación** en la barra de comandos
  >  5. Selecciona **Cargar una aplicación personalizada** en el cuadro de diálogo
  >  6. En el Explorador de archivos, ve a la carpeta de la solución, abre la carpeta **appPackage\build** y selecciona **appPackage.local.zip**, y luego selecciona **Agregar**

Procede a instalar la aplicación:

1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**
1. Abre un chat de Microsoft Teams nuevo o existente
1. En el área de redacción del mensaje, selecciona **...** para abrir el control flotante de la aplicación
1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes
1. En el cuadro de texto, introduce **Bot Builder** para iniciar una búsqueda
1. En la lista de resultados, selecciona un resultado para insertar una tarjeta en el cuadro de redacción del mensaje

Cierra el explorador para detener la sesión de depuración.

[Ir al ejercicio siguiente...](./3-exercise-add-single-sign-on.md)