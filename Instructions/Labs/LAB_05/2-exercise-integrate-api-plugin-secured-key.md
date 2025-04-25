---
lab:
  title: 'Ejercicio 1: Integración de un complemento de API con una API protegida con una clave'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Ejercicio 1: Integración de un complemento de API con una API protegida con una clave

Los complementos de API para Microsoft 365 Copilot te permiten integrarlo con las API protegidas con una clave. Para proteger la clave de API, regístrala en el almacén de Teams. En tiempo de ejecución, Microsoft 365 Copilot ejecuta el complemento, recupera la clave de API del almacén y la usa para llamar a la API. Al seguir este proceso, la clave de API permanece segura y nunca se expone al cliente.

En este ejercicio, crearás un nuevo agente declarativo con un complemento de API que se autentica con una clave de API que generes.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Crear un nuevo proyecto

Empieza por crear un nuevo complemento de API para Microsoft 365 Copilot. Abra Visual Studio Code.

En Visual Studio Code:

1. En la **barra Activity**, abre la extensión kit de herramientas de Teams.
1. En el panel de extensión **Teams Toolkit**, elige **Create a New App**.
1. En la lista de plantillas de proyecto, elige **Copilot Agent**.
1. En la lista de características de la aplicación, elige **Declarative Agent**.
1. Elige la opción **Add plugin**.
1. Elige la opción **Start with a new API**.
1. En la lista de tipos de autenticación, elige **API Key (Bearer Token Auth)**.
1. Como lenguaje de programación, elige **TypeScript**.
1. Elige una carpeta para almacenar el proyecto.
1. Asigna al proyecto el nombre **da-repairs-key**.

El kit de herramientas de Teams crea un nuevo proyecto que incluye un agente declarativo, un complemento de API y una API protegida con una clave.

## Tarea 2: Examinar la configuración de autenticación de clave de API

Antes de continuar, examina la configuración de autenticación de clave de API en el proyecto generado.

### Examen de la definición de la API

Para empezar, mira cómo se define la autenticación de clave de API en la definición de API.

En Visual Studio Code:

1. Abre el archivo **appPackage/apiSpecificationFile/repair.yml**. Este archivo contiene la definición de OpenAPI para la API.
1. En la sección **components.securitySchemes**, observa la propiedad **apiKey**:

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  La propiedad define un esquema de seguridad que usa la clave de API como token de portador en el encabezado de solicitud de autorización.

1. Busca la propiedad **paths./repairs.get.security**. Observa que hace referencia al esquema de seguridad **apiKey**.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### Examen de la implementación de API

A continuación, mora cómo valida la API la clave de API en cada solicitud.

En Visual Studio Code:

1. Abre el archivo **src/functions/repairs.ts**.
1. En la función de controlador de **repairs**, busca la siguiente línea que rechaza todas las solicitudes no autorizadas:

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. La función **isApiKeyValid** se implementa aún más en el archivo repairs.ts:

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  La función comprueba si el encabezado de autorización contiene un token de portador y lo compara con la clave de API definida en la variable de entorno **API_KEY**.

Este código muestra una implementación simplista de la seguridad de clave de API, pero ilustra cómo funciona la seguridad de las claves de API en la práctica.

### Examen de la configuración de la tarea del almacén

En este proyecto, usarás el kit de herramientas de Teams para agregar la clave de API al almacén. El kit de herramientas de Teams registra la clave de API en el almacén mediante una tarea especial en la configuración del proyecto.

En Visual Studio Code:

1. Abre el archivo **./teampsapp.local.yml**.
1. En la sección **Provision**, busca la tarea **apiKey/register**.

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  La tarea toma el valor de la variable de proyecto **SECRET_API_KEY**, almacenada en el archivo **env/.env.local.user**, y la registra en el almacén. A continuación, toma el Id. de entrada del almacén y lo escribe en el archivo de entorno **env/.env.local**. El resultado de esta tarea es una variable de entorno denominada **APIKEY_REGISTRATION_ID**. El kit de herramientas escribe el valor de esta variable en el archivo **appPackages/ai-plugin.json** que contiene la definición del complemento. En tiempo de ejecución, el agente declarativo que carga el complemento de API usa este Id. para recuperar la clave de API del almacén y llamar a la API de forma segura.

## Tarea 3: Configurar la clave de API para el desarrollo local

Para poder probar el proyecto, debes definir una clave de API para la API. A continuación, almacena la clave de API en el almacén y registra el id. de entrada del almacén en el complemento de API. Para el desarrollo local, almacena la clave de API en el proyecto y usa el kit de herramientas de Teams para registrarla en el almacén automáticamente.

En Visual Studio Code:

1. Abre el panel **Terminal** (Ctrl + ').
1. En una línea de comandos:
  1. Restaura las dependencias del proyecto, mediante la ejecución de `npm install`.
  1. Genera una nueva clave de API mediante la ejecución de: `npm run keygen`.
  1. Copia la clave generada en el portapapeles.
1. Abre el archivo **env/.env.local.user**.
1. Actualiza la propiedad **SECRET_API_KEY** a la clave de API recién generada. La propiedad actualizada tiene el siguiente aspecto:

  ```text
  SECRET_API_KEY='your_key'
  ```

1. Guarda los cambios.

Cada vez que compiles el proyecto, el kit de herramientas de Teams actualiza automáticamente la clave de API en el almacén y actualiza el proyecto con el id. de entrada del almacén.