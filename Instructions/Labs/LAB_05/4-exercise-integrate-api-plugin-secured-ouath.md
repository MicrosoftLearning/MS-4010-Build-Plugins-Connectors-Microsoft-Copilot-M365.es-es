---
lab:
  title: 'Ejercicio 3: Integración de un complemento de API con una API protegida con OAuth'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Ejercicio 3: Integración de un complemento de API con una API protegida con OAuth

Los complementos de API para Microsoft 365 Copilot permiten una integración con las API protegidas con OAuth. El Id. de cliente y el secreto de la aplicación que protege la API al registrarlos en el almacén de Teams. En tiempo de ejecución, Microsoft 365 Copilot ejecuta el complemento, recupera la información del almacén y la usa para obtener un token de acceso y llamar a la API. Al seguir este proceso, el Id. de cliente y el secreto permanecen seguros y nunca se exponen al cliente.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Abrir el proyecto de ejemplo y examinar la configuración de autenticación

Empiece descargando el proyecto de ejemplo:

1. En un explorador web, ve a [https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs). Recibirás un mensaje para descargar un archivo ZIP con el proyecto de ejemplo.
1. Guarda el archivo ZIP en el equipo.
1. Extrae el contenido del archivo ZIP.
1. Abre la carpeta en Visual Studio Code.

El proyecto de ejemplo es un proyecto de Kit de herramientas de Teams que incluye un agente declarativo, un complemento de API y una API segura con Microsoft Entra ID. La API se ejecuta en Azure Functions e implementa la seguridad mediante las funcionalidades de autenticación y autorización integradas de Azure Functions, a veces denominadas Easy Auth.

## Tarea 2: Examinar la configuración de autorización de OAuth2

Antes de continuar, examina la configuración de autorización de OAuth2 en el proyecto de ejemplo.

### Examen de la definición de la API

Para empezar, examina la configuración de seguridad de la definición de API incluida en el proyecto.

En Visual Studio Code:

1. Abre el archivo **appPackage/apiSpecificationFile/repair.yml**.
1. En la sección **components.securitySchemes**, observa la propiedad **oAuth2AuthCode**:

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  La propiedad define un esquema de seguridad de OAuth2 e incluye información sobre las direcciones URL a las que llamar para obtener un token de acceso y los ámbitos que usa la API.

  > **IMPORTANTE** Observa que el ámbito está completo con el URI del Id. de aplicación (**api://...**). Al trabajar con Microsoft Entra, debes calificar completamente los ámbitos personalizados. Cuando Microsoft Entra ve un ámbito no completo, supone que pertenece a Microsoft Graph, lo que conduce a errores de flujo de autorización.

1. Busca la propiedad **paths./repairs.get.security**. Observa que hace referencia al esquema de seguridad **oAuth2AuthCode** y al ámbito que el cliente necesita para realizar la operación.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **IMPORTANTE** Es meramente informativo enumerar los ámbitos necesarios en la especificación de API. Al implementar la API, eres responsable de validar el token y comprobar que contiene los ámbitos necesarios.

### Examen de la implementación de API

A continuación, consulta la implementación de la API.

En Visual Studio Code:

1. Abre el archivo **src/functions/repairs.ts**.
1. En la función de controlador de **reparaciones**, busca la siguiente línea que comprueba si la solicitud contiene un token de acceso con los ámbitos necesarios:

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. La función **hasRequiredScopes** se implementa aún más en el archivo **repairs.ts**:

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  La función comienza extrayendo el token de portador del encabezado de solicitud de autorización. A continuación, usa el paquete **jwt-decode** para descodificar el token y obtener la lista de ámbitos de la notificación **scp**. Por último, comprueba si la notificación **scp** contiene todos los ámbitos necesarios.

  Observa que la función no valida el token de acceso. En su lugar, solo comprueba si el token de acceso contiene los ámbitos necesarios. En esta plantilla, la API se ejecuta en Azure Functions e implementa la seguridad mediante Easy Auth, que es responsable de validar el token de acceso. Si la solicitud no contiene un token de acceso válido, el entorno de ejecución de Azure Functions lo rechaza antes de que llegue al código. Aunque Easy Auth valida el token, no comprueba los ámbitos necesarios, algo que debes hacer tú.

### Examen de la configuración de la tarea del almacén

En este proyecto, usarás el kit de herramientas de Teams para agregar la información de OAuth al almacén. El kit de herramientas de Teams registra la información de OAuth en el almacén mediante una tarea especial en la configuración del proyecto.

En Visual Studio Code:

1. Abre el archivo **./teampsapp.local.yml**.
1. En la sección **provision**, busca la tarea **oauth/register**.

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  La tarea toma los valores de las variables de proyecto **TEAMS_APP_ID**, **AAD_APP_CLIENT_ID** y **SECRET_AAD_APP_CLIENT_SECRET** almacenadas en los archivos **env/.env.local** y **env/.env.local.user** y los registra en el almacén. También habilita la clave de prueba para el intercambio de códigos (PKCE) para obtener seguridad adicional. A continuación, toma el Id. de entrada del almacén y lo escribe en el archivo de entorno **env/.env.local**. El resultado de esta tarea es una variable de entorno denominada **OAUTH2AUTHCODE_CONFIGURATION_ID**. El kit de herramientas escribe el valor de esta variable en el archivo **appPackages/ai-plugin.json** que contiene la definición del complemento. En tiempo de ejecución, el agente declarativo que carga el complemento de API usa este Id. para recuperar la información de OAuth del almacén y empezar un flujo de autorización para obtener un token de acceso.

  > **IMPORTANTE** La tarea **oauth/register** solo es responsable de registrar la información de OAuth en el almacén si aún no existe. Si la información ya existe, el kit de herramientas de Teams omitirá la ejecución de esta tarea.

1. A continuación, busca la tarea **oauth/update**.

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  La tarea mantiene la información de OAuth en el almacén sincronizado con el proyecto. Es necesario que el proyecto funcione correctamente. Una de las propiedades clave es la dirección URL en la que está disponible el complemento de API. Cada vez que inicies el proyecto, el kit de herramientas de Teams abre un túnel de desarrollo en una nueva dirección URL. La información de OAuth del almacén debe hacer referencia a esta dirección URL para que Copilot pueda acceder a la API.

### Examen de la configuración de autenticación y autorización

La siguiente parte que se va a explorar es la configuración de la autenticación y la autorización de Azure Functions. La API de este ejercicio usa las funcionalidades integradas de autenticación y autorización de Azure Functions. El kit de herramientas de Teams configura estas funcionalidades al aprovisionar Azure Functions en Azure.

En Visual Studio Code:

1. Abre el archivo **infra/azure.bicep**.
1. Busca el recurso **authSettings**:

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  Este recurso habilita las funcionalidades integradas de autenticación y autorización en la aplicación de Azure Functions. Para empezar, en la sección **globalValidation**, define que la aplicación solo permite solicitudes autenticadas. Si la aplicación recibe una solicitud no autenticada, la rechaza con un error HTTP 401. A continuación, en la sección **identityProviders**, la configuración define que usa Microsoft Entra ID (anteriormente conocido como Azure Active Directory) para autorizar las solicitudes. Define el registro de aplicaciones de Microsoft Entra que usa para proteger la API y especifica las audiencias permitidas para llamar a la API.

### Examen del registro de la aplicación de Microsoft Entra

La parte final que se va a examinar es el registro de aplicaciones de Microsoft Entra que el proyecto usa para proteger la API. Al usar OAuth, se protege el acceso a los recursos mediante una aplicación. Normalmente, la aplicación define las credenciales necesarias para obtener un token de acceso, como un secreto de cliente o un certificado. También especifica los distintos permisos (también conocidos como ámbitos) que el cliente puede solicitar al llamar a la API. El registro de aplicaciones de Microsoft Entra representa una aplicación en la nube de Microsoft y define una aplicación para su uso con los flujos de autorización de OAuth.

En Visual Studio Code:

1. Abre el archivo **./aad.manifest.json**.
1. Busca la propiedad **oauth2Permissions**.

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  La propiedad define un ámbito personalizado denominado **repairs_read** para conceder al cliente permiso para leer las reparaciones de la API de reparaciones.

1. Busca la propiedad **identifierUris**.

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  La propiedad **identifierUris** define un identificador que se usa para calificar completamente el ámbito.
