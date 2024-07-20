---
lab:
  title: 'Ejercicio 2: Agregar un inicio de sesión único'
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 2: Agregar un inicio de sesión único

En este ejercicio, actualizarás la extensión de mensajes para que se pida a los usuarios que inicien sesión y autentiquen. Configurarás el registro de aplicaciones de Bot Microsoft Entra y el manifiesto de la aplicación para habilitar el inicio de sesión único. Configurarás un registro de aplicaciones de Microsoft Entra para autenticarse con Microsoft Graph y actualizarás la lógica de la extensión de mensajes para obtener un token de acceso mediante el servicio de token de Bot Framework. Después ejecutarás y depurarás la extensión de mensajes para probarla en Microsoft Teams.

## Tarea 1: Configuración del inicio de sesión único

Configura primero un registro de la aplicación en Bot Microsoft Entra.

En Visual Studio:

1. En la carpeta **infra\entra**, abre el archivo denominado **entra.bot.manifest.json**
1. En el archivo, actualiza la matriz **identifierUris** para establecer el URI del id. de aplicación

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ]
    ```

1. En el archivo, actualiza la matriz **oauth2Permissions** para crear un ámbito para permitir que Teams llame a las API web como administrador o usuario:

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. En el archivo, actualiza la matriz **preAuthorizedApplications** para agregar clientes de Microsoft Teams, Microsoft Outlook y Copilot para Microsoft 365 a la lista de clientes autorizados:

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ]
    ```

1. Guarda los cambios

Luego actualiza el archivo de manifiesto de la aplicación para definir el recurso que el cliente debe usar al iniciar un flujo de inicio de sesión único en la aplicación.

Continúa en Visual Studio:

1. En la carpeta **appPackage**, abre el archivo denominado **manifest.json**
1. En el archivo agrega el código siguiente:

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    }
    ```

1. Guarda los cambios

## Tarea 2: Creación de un archivo de manifiesto de registro de aplicaciones de Microsoft Entra para Microsoft Graph

Para autenticarse con Microsoft Graph, crea un nuevo archivo de manifiesto de registro de aplicaciones.

Continúa en Visual Studio:

1. En la carpeta **infra\entra**, crea un archivo denominado **entra.graph.manifest.json**
2. En el archivo agrega el código siguiente:

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
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
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
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

1. Guarda los cambios

La matriz **requiredResourceAccess** define los ámbitos de permisos de API y la matriz **replyUrlsWithType** define los URI de redirección en el registro de la aplicación.

Ahora, actualiza el archivo del proyecto con acciones para crear el registro de la aplicación.

1. En la carpeta raíz del proyecto, abre **teamsapp.local.yml**
1. En el archivo, busca el paso que usa la acción **arm/deploy**
1. Antes del paso, agrega el código siguiente:

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. Guarda los cambios

## Tarea 3: Creación de la configuración de la conexión de OAuth

La configuración de la conexión del Servicio de Bot de Azure AI se usa para administrar la autenticación de usuarios en bots y extensiones de mensajes.

Primero, centraliza el nombre de la configuración de la conexión de OAuth que se usa para crear la configuración de conexión como una variable de entorno y agrega código para usar el valor de la variable de entorno en el tiempo de ejecución.

Continúa en Visual Studio:  

1. En la carpeta **env**, abre **.env.local**
1. En el archivo agrega el código siguiente:

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. En la carpeta raíz del proyecto, abre el archivo denominado **teamsapp.local.yml**
1. En el archivo, busca el paso que usa la acción **file/createOrUpdateJsonFile** destinada al archivo **/appsettings.Development.json**
1. Actualiza la matriz de contenido agregando la variable **CONNECTION_NAME**

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Guarda los cambios
1. En la carpeta raíz del proyecto, abre **Config.cs**
1. En la clase **ConfigOptions**, agrega una nueva propiedad de cadena con el nombre **CONNECTION_NAME**

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Guarda los cambios
1. En la carpeta raíz del proyecto, abre el archivo denominado **Program.cs**
1. En el archivo, agrega una nueva línea para agregar la variable de entorno **CONNECTION_NAME** como una configuración de App Configuration:

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

Después, actualiza el controlador de actividad del bot para acceder a App Configuration.

1. En la carpeta **Search**, abre el archivo denominado **SearchApp.cs**
1. En la clase **SearchApp**, crea una propiedad de cadena de solo lectura con el nombre **connectionName**.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. Crea un constructor, que inserta la App Configuration y establece el valor de la propiedad connectionName.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. Guarda los cambios

Luego actualiza los archivos Bicep para aprovisionar la configuración de la conexión de OAuth.

Primero, actualiza el archivo de parámetros para pasar las credenciales del registro de la aplicación de Microsoft Graph Microsoft Entra y el nombre de la configuración de la conexión.

1. En la **carpeta** infra, abre el archivo denominado **azure.parameters.local.json**
1. En el **objeto** parameters, agrega los parámetros **graphEntraAppClientId**, **graphEntraAppClientSecret** y **connectionName**

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
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Guarda los cambios

Luego actualiza el archivo Bicep.

1. En la carpeta **infra**, abre el archivo denominado **azure.local.bicep**
1. En el archivo, después de la declaración del parámetro **botAppDomain**, agrega las declaraciones del parámetro **graphEntraAppClientId**, **graphEntraAppClientSecret** y **connectionName**

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. En el módulo **azureBotRegistration** , agrega los parámetros **graphEntraAppClientId**, **graphEntraAppClientSecret** y **connectionName**

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Guarda los cambios.

Por último, actualiza el archivo Bicep de registro del bot.

1. En la carpeta **infra/botRegistration**, abre el archivo denominado **azurebot.bicep**
1. En el archivo, después de la declaración de parámetro **botAppDomain**, agrega las declaraciones de parámetro **graphEntraAppClientId**, **graphEntraAppClientSecret** y **connectionName**

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. Después del recurso **botServiceM365ExtensionsChannel**, agrega un nuevo recurso para la conexión de Microsoft Graph.

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. Guarda los cambios

## Tarea 4: Autenticación de las consultas de usuario

Luego agrega código para autenticar a los usuarios cuando inicien una búsqueda mediante la extensión de mensajes.

Continúa en Visual Studio:

1. En la carpeta **Search**, abre el archivo denominado **SearchApp.cs**
1. En el archivo, empieza agregando el espacio de nombres **authentication** desde Bot Framework SDK.

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega el código siguiente al principio del método:

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

El bloque de código anterior usa tres métodos que controlan la autenticación de usuarios.

- **GetToken** usa el cliente de servicio de token para obtener un token de acceso para el usuario actual
- **HasToken** comprueba si la respuesta del servicio de token contiene un token de acceso
- Se llama a **CreateAuthResponse** si no se devuelve ningún token y se devuelve una respuesta, que muestra un vínculo de inicio de sesión en la interfaz de usuario

Ahora, crea los métodos en la clase **SearchApp**.

- Crea el método **GetToken** con el código siguiente:

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
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
```

Comprueba primero si el parámetro de estado no es nulo o está vacío, el código intentará analizarlo como un entero mediante el método int.TryParse. Si el análisis se realiza correctamente, el valor analizado se asigna a la variable magicCode como una cadena. Luego magicCode se pasa como un argumento al método GetUserTokenAsync junto con otros parámetros. El método GetUserTokenAsync usa magicCode para comprobar la autenticidad de la solicitud. Este se asegura de que la misma entidad que solicita el token de usuario es la que inició el proceso de autenticación.

- Crea el método HasToken con el código siguiente:

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

Para comprobar si se obtiene un token válido del servicio de token, comprueba si la respuesta del token y la propiedad Token de la respuesta no están vacías o nulas.

- Crea el método **CreateAuthResponse** con el código siguiente:

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

Primero usa el método **GetSignInResourceAsync** para recuperar el vínculo de inicio de sesión del servicio de token. El vínculo de inicio de sesión se usa para construir un objeto **MessagingExtensionResponse**. Crea un nuevo objeto y establece la propiedad **ComposeExtension** de la respuesta en un nuevo objeto **MessagingExtensionResult**. La propiedad type del resultado se establece en "auth" para indicar que el resultado es una respuesta de autenticación. La propiedad **SuggestedActions** del resultado se establece en un nuevo objeto **MessagingExtensionSuggestedAction**. La propiedad Actions de las acciones sugeridas se establece en una lista que contiene un único objeto **CardAction**. Este objeto **CardAction** representa una acción que el usuario puede realizar. La propiedad Type de **CardAction** se establece en **ActionTypes.OpenUrl**, lo que indica que es una acción que abre una dirección URL. La propiedad Value se establece en el vínculo de inicio de sesión recuperado del recurso. La propiedad Title se establece en "Iniciar sesión", que especifica el título de la acción. Por último, la respuesta construida se devuelve del método .

Cuando un usuario sigue el vínculo de inicio de sesión se le lleva a un recurso hospedado en un dominio externo. El dominio debe incluirse en el archivo de manifiesto de la aplicación. Agrega el dominio del servicio de token de Bot Framework al manifiesto de la aplicación.

Continúa en Visual Studio:

1. En la carpeta **appPackage**, abre **manifest.json**
1. En el archivo, actualiza la matriz **validDomains** y agrega el dominio del servicio de token:

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. Guarda los cambios.

## Tarea 5: Aprovisionamiento de recursos

Ahora que ya está todo listo, ejecuta el proceso de preparar las dependencias de la aplicación de Teams para aprovisionar los recursos necesarios.

Continúa en Visual Studio:

1. En **Explorador de soluciones**, haz clic con el botón derecho en el proyecto **MsgExtProductSupport**
1. Expande el menú del **kit de herramientas de teams** y selecciona **Preparar dependencias de la aplicación de Teams**
1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona **Continuar**
1. En el cuadro de diálogo **Provision**, selecciona **Provision**
1. En el cuadro de diálogo **Teams Toolkit warning**, selecciona **Provision**
1. En el cuadro de diálogo **Teams Toolkit information**, selecciona **View provisioned resources** para abrir una nueva ventana del explorador.

Dedica un momento a explorar los recursos que se crean y actualizan en Azure.

## Tarea 6: Ejecución y depuración

Ahora inicia el servicio web y prueba la extensión de mensajes en Microsoft Teams.

Continúa en Visual Studio:

1. Presiona **F5** para iniciar una sesión de depuración y abrir una nueva ventana del explorador que vaya al cliente web de Microsoft Teams.
1. En el explorador y, si es necesario, introduce tus credenciales de cuenta de Microsoft 365 y continúe con Microsoft Teams.
1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**
1. Abre un chat de Microsoft Teams nuevo o existente
1. En el área de redacción del mensaje, selecciona **...** para abrir el control flotante de la aplicación
1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes
1. En el cuadro de texto, introduce **Bot Builder** para iniciar una búsqueda
1. En la lista de resultados, **selecciona un resultado** para insertar una tarjeta en el cuadro de redacción del mensaje
1. Se muestra el mensaje: **You'll need to sign in to use this app**
1. Selecciona el **vínculo de inicio de sesión** para abrir una nueva pestaña e iniciar el flujo de autenticación
1. En la página de consentimiento de permisos, revisa los permisos que se solicitan
1. Selecciona *Aceptar* para cerrar la pestaña y volver a Microsoft Teams
1. En el área de redacción del mensaje, selecciona **...** para abrir el control flotante de la aplicación
1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes
1. En el cuadro de texto, introduce **Bot Builder** para iniciar una búsqueda
1. Se te pedirá que vuelvas a iniciar sesión. Sigue el **vínculo de inicio de sesión** de nuevo para iniciar la búsqueda.
1. En la lista de resultados, **selecciona un resultado** para insertar una tarjeta en el cuadro de redacción del mensaje

Cierra el explorador para detener la sesión de depuración.

[Ir al ejercicio siguiente...](./4-exercise-retrieve-product-information-from-sharepoint-online.md)