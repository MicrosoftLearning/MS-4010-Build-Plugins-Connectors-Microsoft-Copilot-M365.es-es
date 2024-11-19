---
lab:
  title: 'Ejercicio 2: Agregar un inicio de sesión único'
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 2: Agregar un inicio de sesión único

En este ejercicio, agregarás el inicio de sesión único a la extensión de mensajes para autenticar las consultas de usuario.

![Captura de pantalla de un desafío de autenticación en una extensión de mensajes basada en búsqueda. Se muestra un vínculo al inicio de sesión.](../media/2-sign-in.png)

### Duración del ejercicio

  - **Tiempo estimado para completarla:** 40 minutos

## Tarea 1: Configuración del registro de aplicaciones de API de back-end

En primer lugar, crea un registro de aplicación de Microsoft Entra para la API de back-end. Para los fines de este ejercicio, crearás uno nuevo, sin embargo, en un entorno de producción, usarías un registro de aplicación existente.

En una ventana del explorador:

1. Accede a [Azure Portal](https://portal.azure.com).

1. Abre el menú del portal y selecciona **Microsoft Entra ID**.

1. Selecciona **Administrar > Registros de aplicaciones** y luego **Nuevo registro**.

1. En el formulario Registrar una aplicación, especifica los valores siguientes:

    1. **Nombre**: Products API

    1. **Tipos de cuenta admitidos**: cuentas en cualquier directorio organizacional (cualquier inquilino - multiinquilino de Microsoft Entra ID).

1. Selecciona **Registrar** para crear el registro de aplicación.

1. En el menú de la izquierda del registro de aplicación, selecciona **Administrar > Exponer una API**.

1. Junto a **URI de id. de aplicación**, selecciona **Agregar** y **Guardar** para crear un URI de id. de aplicación.

1. En la sección Ámbitos definidos con esta API, selecciona **Agregar un ámbito**.

1. En el formulario Agregar un ámbito, especifica los valores siguientes:

    1. **Nombre de ámbito**: Product.Read

    1. **¿Quién puede dar el consentimiento?**: Administradores y usuarios

    1. **Nombre para mostrar del consentimiento del administrador**: Leer products

    1. **Descripción del consentimiento del administrador**: Permite que la aplicación lea los datos del producto

    1. **Nombre para mostrar del consentimiento del usuario**: Leer productos

    1. **Descripción del consentimiento del usuario**: Permite que la aplicación lea los datos del producto

    1. **Estado**: Habilitado

1. Selecciona **Agregar ámbito** para crear el ámbito.

A continuación, anota el id. de registro de aplicación y el id. de ámbito. Necesitas estos valores para configurar el registro de aplicación usado para obtener un token de acceso para la API de back-end.

1. En el menú de la izquierda del registro de aplicación, selecciona **Manifiesto**.

1. Copie el valor de propiedad **appId** y guárdalo para usarlo más tarde.

1. Copia el valor de la propiedad **oauth2Permissions.id** y guárdalo para usarlo más tarde.

Como necesitamos estos valores en el proyecto, agrégalos al archivo de entorno.

En Visual Studio y el **proyecto TeamsApp**:

1. En la carpeta **env**, abre **.env.local**.

1. En el archivo, agrega las siguientes variables de entorno y establece los valores en el **id. de registro de aplicación** y el **id. de ámbito** que guardaste anteriormente:

    ```text
    BACKEND_API_ENTRA_APP_ID=<app-registration-id>
    BACKEND_API_ENTRA_APP_SCOPE_ID=<scope-id>
    ```

1. Guarda los cambios.

## Tarea 2: Creación de un archivo de manifiesto de registro de aplicación para la autenticación con la API de back-end

Para autenticarse con la API de back-end, necesitas un registro de aplicación para obtener un token de acceso con el que llamar a la API.

A continuación, crea un archivo de manifiesto de registro de aplicación. El manifiesto define los ámbitos de permisos de API y el URI de redirección en el registro de aplicación.

En Visual Studio y el **proyecto TeamsApp**:

1. En la carpeta **infra\entra**, crea un nuevo archivo (<kbd>Ctrl+Mayús+A</kbd>) denominado **entra.products.api.manifest.json**.

1. En el archivo agrega el código siguiente:

    ```json
    {
      "id": "${{PRODUCTS_API_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{PRODUCTS_API_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-product-api-${{TEAMSFX_ENV}}",
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
      "requiredResourceAccess": [
        {
          "resourceAppId": "${{BACKEND_API_ENTRA_APP_ID}}",
          "resourceAccess": [
            {
              "id": "${{BACKEND_API_ENTRA_APP_SCOPE_ID}}",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. Guarda los cambios.

La propiedad **requiredResourceAccess** especifica el id de registro de aplicación y el id. de ámbito de la API de back-end.

La propiedad **replyUrlsWithType** especifica el URI de redirección usado por el servicio de token de Bot Framework para devolver el token de acceso al servicio de token después de que el usuario se autentique.

A continuación, actualice el flujo de trabajo automatizado para crear y actualizar el registro de aplicación.

En el proyecto **TeamsApp**:

1. Abre **teamsapp.local.yml**.

1. En el archivo, busca el paso que usa la acción **aadApp/update**.

1. Después de la acción, agrega las acciones **aadApp/create** y **aadApp/update** para crear y actualizar el registro de la aplicación (a partir de la **línea 31**):

    ```yml
      - uses: aadApp/create
        with:
            name: ${{APP_INTERNAL_NAME}}-products-api-${{TEAMSFX_ENV}}
            generateClientSecret: true
            signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
            clientId: PRODUCTS_API_ENTRA_APP_ID
            clientSecret: SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET
            objectId: PRODUCTS_API_ENTRA_APP_OBJECT_ID
            tenantId: PRODUCTS_API_ENTRA_APP_TENANT_ID
            authority: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY
            authorityHost: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
            manifestPath: "./infra/entra/entra.products.api.manifest.json"
            outputFilePath : "./infra/entra/build/entra.products.api.${{TEAMSFX_ENV}}.json"
    ```

1. Guarda los cambios

La acción **aadApp/create** crea un nuevo registro de aplicación con el nombre, la audiencia y genera un secreto de cliente especificado. La propiedad **writeToEnvironmentFile** escribe el id. de registro de aplicación, el secreto de cliente, el id. de objeto, el id. de inquilino, la autoridad y el host de autoridad en los archivos de entorno. El secreto de cliente se cifra y almacena de forma segura en el archivo **env.local.user**. El nombre de la variable de entorno del secreto de cliente tiene el prefijo **SECRET_**, indica al Kit de herramientas de Teams que no escriba el valor en los registros.

La acción **aadApp/update** actualiza el registro de aplicación con el archivo de manifiesto especificado.

## Tarea 3: Centralización del nombre de la configuración de conexión

A continuación, centraliza el nombre de la configuración de conexión en el archivo de entorno y actualiza la configuración de la aplicación para acceder al valor de variable de entorno en runtime.

Continúa en Visual Studio y en el proyecto **TeamsApp**:

1. En la carpeta **env**, abre **.env.local**.

1. En el archivo agrega el código siguiente:

    ```text
    CONNECTION_NAME=ProductsAPI
    ```

1. Abre **teamsapp.local.yml**.

1. En el archivo, busca el paso que usa la acción **file/createOrUpdateJsonFile** destinada al archivo **/appsettings.Development.json**. Actualiza la matriz de contenido para incluir la variable de entorno **CONNECTION_NAME** y escribe el valor en el archivo **appsettings.Development.json**:

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../ProductsPlugin/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Guarda los cambios.

A continuación, actualiza la configuración de la aplicación para acceder a la variable de entorno **CONNECTION_NAME**.

En el proyecto **ProductsPlugin**:

1. Abre **Config.cs**.

1. En la clase **ConfigOptions**, agrega una nueva propiedad con el nombre **CONNECTION_NAME**:

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Guarda los cambios.

1. Abre **Program.cs**.

1. En el archivo, actualiza el código que lee la configuración de la aplicación para incluir la propiedad **CONNECTION_NAME**:

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["ConnectionName"] = config.CONNECTION_NAME;
    ```

1. Guarda los cambios.

A continuación, actualiza el código del bot para usar el nombre de configuración de conexión en tiempo de ejecución.

1. En la carpeta **Search**, abre **SearchApp.cs**.

1. Al principio de la clase **SearchApp** (aproximadamente en la línea 14), crea un constructor que acepte un objeto **IConfiguration** y asigne el valor de la propiedad **CONNECTION_NAME** a un campo privado llamado **connectionName**:

    ```csharp
    private readonly string connectionName;
    public SearchApp(IConfiguration configuration)
    {
      connectionName = configuration["CONNECTION_NAME"];
    }  
    ```

1. Guarde los cambios.

## Tarea 4: Configuración de la configuración de conexión de la API de productos

Para autenticarte con la API de back-end, debes configurar una configuración de conexión en el recurso de Azure Bot.

Continúa en Visual Studio y en el proyecto **TeamsApp**:

1. En la carpeta **infra**, abre el archivo denominado **azure.parameters.local.json**.

1. En el archivo, agrega los parámetros **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** y **connectionName**:

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
        },
        "backendApiEntraAppClientId": {
          "value": "${{BACKEND_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientId": {
          "value": "${{PRODUCTS_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientSecret": {
          "value": "${{SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Guarda los cambios.

A continuación, actualiza el archivo Bicep para incluir los nuevos parámetros y pasarlos al recurso de Azure Bot.

1. En la carpeta **infra**, abre el archivo denominado **azure.local.bicep**.

1. En el archivo, después de la declaración del parámetro **botAppDomain**, agrega las declaraciones del parámetro **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** y **connectionName**:

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. En la declaración del módulo **azureBotRegistration**, agrega los nuevos parámetros:

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botEntraAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        backendApiEntraAppClientId: backendApiEntraAppClientId
        productsApiEntraAppClientId: productsApiEntraAppClientId
        productsApiEntraAppClientSecret: productsApiEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Guarda los cambios.

Por último, actualice el archivo Bicep de registro del bot para incluir la nueva configuración de conexión.

1. En la carpeta **infra/botRegistration**, abre **azurebot.bicep**.

1. En el archivo, después de la declaración de parámetro **botAppDomain**, agrega las declaraciones de parámetro **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret**, and **connectionName**:

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. En el archivo, agrega un nuevo recurso denominado **botServicesProductsApiConnection** al final del archivo:

    ```bicep
    resource botServicesProductsApiConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: productsApiEntraAppClientId
        clientSecret: productsApiEntraAppClientSecret
        scopes: 'api://${backendApiEntraAppClientId}/Product.Read'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${botEntraAppClientId}'
          }
        ]
      }
    }
    ```

1. Guarda los cambios.

## Tarea 5: Configuración de la autenticación en la extensión de mensajes

Para autenticar las consultas de usuario en la extensión de mensajes, usa el SDK de Bot Framework para obtener un token de acceso para el usuario desde servicio de token de Bot Framework. El token de acceso se puede usar después para acceder a los datos desde un servicio externo.

Para simplificar el código, crea una clase auxiliar que controle la autenticación de usuario.

Continúa en Visual Studio y el proyecto **ProductsPlugin**:

1. Crea una nueva carpeta denominada **Helpers**.

1. En la carpeta **Helpers**, crea un nuevo archivo de clase denominado **AuthHelpers.cs**

1. En el archivo agrega el código siguiente:

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    using Microsoft.Bot.Schema.Teams;
    internal static class AuthHelpers
    {
        internal static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
        {
            var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);
            return new MessagingExtensionResponse
            {
                ComposeExtension = new MessagingExtensionResult
                {
                    Type = "auth",
                    SuggestedActions = new MessagingExtensionSuggestedAction
                    {
                        Actions = [
                            new() {
                                Type = ActionTypes.OpenUrl,
                                Value = resource.SignInLink,
                                Title = "Sign In",
                            },
                        ],
                    },
                },
            };
        }
        internal static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
        {
            var magicCode = string.Empty;
            if (!string.IsNullOrEmpty(state))
            {
                if (int.TryParse(state, out var parsed))
                {
                    magicCode = parsed.ToString();
                }
            }
            return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
        }
        internal static bool HasToken(TokenResponse tokenResponse) => tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
    }
    ```

1. Guarda los cambios.

Los tres métodos auxiliares de la clase **AuthHelpers** controlan la autenticación de usuario en la extensión de mensajes.

- El método **CreateAuthResponse** crea una respuesta que representa un vínculo de inicio de sesión en la interfaz de usuario. Usa el método **GetSignInResourceAsync** para recuperar el vínculo de inicio de sesión del servicio de token.

- El método **GetToken** usa el cliente de servicio de token para obtener un token de acceso para el usuario actual El método usa un código mágico para comprobar la autenticidad de la solicitud.

- **HasToken** comprueba si la respuesta del servicio de token contiene un token de acceso Si el token no es nulo o está vacío, el método devuelve true.

A continuación, actualiza el código de extensión de mensajes para usar los métodos auxiliares para autenticar consultas de usuario.

1. En la carpeta **Search**, abre **SearchApp.cs**.

1. En la parte superior del archivo, agrega las siguientes instrucciones using:

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega el código siguiente al principio del método:

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

1. Guarda los cambios.

A continuación, agrega el dominio del servicio de token al archivo de manifiesto de la aplicación para asegurarte de que el cliente puede confiar en el dominio al iniciar un flujo de inicio de sesión único.

En el proyecto **TeamsApp**:

1. En la carpeta **appPackage**, abre **manifest.json**

1. En el archivo, actualiza la matriz **validDomains** y agrega el dominio del servicio de token:

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]
    ```

1. Guarda los cambios.

## Tarea 6: Creación y actualización de recursos

Ahora que está todo preparado, ejecuta el proceso **Preparar dependencias de la aplicación de Teams** para crear nuevos recursos y actualizar los existentes.

> [!NOTE]
> Si el aprovisionamiento no puede preparar las dependencias, asegúrate de que tienes los valores adecuados para **BACKEND_API_ENTRA_APP_ID** y **BACKEND_API_ENTRA_APP_SCOPE_ID** en **env.local**.

Continúa en Visual Studio:

1. En **Explorador de soluciones**, haz clic con el botón derecho en el proyecto **TeamsApp**

1. Expande el menú del **kit de herramientas de teams** y selecciona **Preparar dependencias de la aplicación de Teams**.

1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona **Continuar**.

1. En el cuadro de diálogo **Aprovisionar**, selecciona **Aprovisionar**.

1. En el cuadro de diálogo **Advertencia del kit de herramientas de Teams**, selecciona **Aprovisionar**.

1. En el cuadro de diálogo **Información del kit de herramientas de Teams**, selecciona el icono de la cruz para cerrar el cuadro de diálogo.

## Tarea 7: Ejecución y depuración

Con los recursos aprovisionados, inicia una sesión de depuración para probar la extensión de mensajes.

1. Para iniciar una nueva sesión de depuración, presiona <kbd>F5</kbd> o selecciona **Iniciar** en la barra de herramientas.

1. Espera hasta que se abra una ventana del explorador y aparezca el cuadro de diálogo de instalación de la aplicación en el cliente web de Microsoft Teams. Si se te solicita, escribe las credenciales de tu cuenta de Microsoft 365.

1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**.

1. Abre un chat de Microsoft Teams nuevo o existente

1. En el área de redacción de mensajes, empieza a escribir **/apps** para abrir el selector de aplicaciones.

1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes

1. En el cuadro de texto, escribe **hello**. Es posible que tengas que escribir la búsqueda varias veces.

1. Se muestra el mensaje: **You'll need to sign in to use this app**:

    ![Captura de pantalla de un desafío de autenticación en una extensión de mensajes basada en búsqueda. Se muestra un vínculo al inicio de sesión.](../media/2-sign-in.png)

1. Sigue el vínculo de **inicio de sesión** para iniciar el flujo de autenticación.

1. Da tu consentimiento a los permisos solicitados y vuelve a Microsoft Teams:

    ![Captura de pantalla del cuadro de diálogo de consentimiento de permisos de la API de Microsoft Entra.](../media/18-api-permission-consent.png)

1. Espera a que se complete la búsqueda y se muestren los resultados.

1. En la lista de resultados, selecciona **hello** para insertar una tarjeta en el cuadro de redacción del mensaje

Vuelve a Visual Studio y selecciona **Detener** en la barra de herramientas o presiona <kbd>Mayús</kbd> + <kbd>F5</kbd> para detener la sesión de depuración.

[Ir al ejercicio siguiente...](./4-exercise-return-data-api.md)