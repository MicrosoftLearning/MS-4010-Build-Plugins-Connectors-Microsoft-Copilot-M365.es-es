---
lab:
  title: 'Ejercicio 1: Configuración de una conexión externa e implementación del esquema'
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Ejercicio: Configuración de una conexión externa e implementación del esquema

En este ejercicio, crearás un conector personalizado de Microsoft Graph como una aplicación de consola. Registrarás un nuevo registro de aplicaciones de Microsoft Entra y agregarás el código para crear una conexión externa e implementar su esquema.

## Antes de comenzar

Este ejercicio tarda unos **XX minutos** en completarse.

## Tarea 1: Registro de un nuevo registro de aplicaciones de Microsoft Entra

Para empezar, registra un nuevo registro de aplicación Entra, que el conector de Graph personalizado usa para autenticarse con Microsoft 365.

En un explorador web:

1. Ve a **Azure Portal** en [https://portal.azure.com](https://portal.azure.com).
1. Selecciona **Microsoft Entra ID** en el panel de navegación.
1. Selecciona **Registros de aplicaciones** en el panel de navegación lateral.
1. En el panel de navegación superior, selecciona **Nuevo registro**.
1. Especifica los valores siguientes:
   1. **Nombre:** conector de Graph de documentos de MSGraph
   1. **Tipos de cuenta admitidos:** cuentas en este directorio organizativo únicamente (inquilino único)
1. Selecciona **Registro** para confirmar la entrada
1. En la pantalla de información general, copia los valores de las propiedades **Id. de aplicación** e **Id. de directorio (inquilino)**. Los vas a necesitar más adelante.

## Tarea 2: Creación de una credencial

Dado que este conector de Graph personalizado se ejecuta sin la interacción del usuario, debes configurarlo para autenticarse automáticamente. Para simplificar, crea un secreto.

Continúa en el explorador web:

1. Selecciona **Certificados y secretos** en el panel de navegación lateral.
1. En la pestaña **Secretos de cliente**, selecciona el botón **Nuevo secreto de cliente**.
1. Crea el secreto seleccionando **Agregar**.
1. Copia el **valor** del secreto recién creado. Lo necesitarás más adelante.

## Paso 3: Concesión de permisos de API

El último paso para configurar el registro de la aplicación Entra es concederle permisos de API para que pueda crear la conexión externa y el esquema.

Continúa en el explorador web:

1. En el panel de navegación lateral, selecciona **Permisos de API**.
1. Selecciona **Agregar permiso**.
1. Selecciona **Microsoft Graph** en la lista de API.
1. Luego selecciona **Permisos de aplicación**.
1. En el cuadro de texto del filtro, escribe **external**.
1. Expande la sección **ExternalConnection** y selecciona el permiso **ExternalConnection.ReadWrite.OwnedBy**.
1. Expande la sección **ExternalItem** y selecciona el permiso **ExternalItem.ReadWrite.OwnedBy**.
1. Para confirmar tu elección, selecciona el botón **Agregar permisos**.
1. Para completar la configuración, concede el consentimiento del administrador seleccionando el botón **Grant admin consent for (tenant)**.
1. Confirma el cuadro de diálogo seleccionando **Sí**.

## Tarea 4: Creación de una nueva aplicación de consola e instalación de las dependencias

Después de configurar el registro de la aplicación Entra, el siguiente paso será crear una aplicación de consola, donde implementarás el código del conector de Graph.

En un terminal:

1. Crea una carpeta nueva y cambia el directorio de trabajo.
1. Crea una nueva aplicación de consola ejecutando `dotnet new console`.
1. Agrega las dependencias, que necesitas para compilar el conector:
   1. Para agregar la biblioteca necesaria para autenticarse con Microsoft 365, ejecuta `dotnet add package Azure.Identity`.
   1. Para agregar la biblioteca cliente para comunicarse con las API de Graph, ejecuta `dotnet add package Microsoft.Graph`.
   1. Para agregar la biblioteca necesaria para trabajar con los secretos de usuario, que configurarás en el paso siguiente, ejecuta `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`.
   1. Implementa el conector de Graph como una aplicación de consola con dos comandos: uno para crear la conexión externa e implementar el esquema y otro para importar el contenido. Ejecuta `dotnet add package System.CommandLine --prerelease` para admitir la definición de comandos en la aplicación.

## Tarea 5: Almacenamiento seguro de la información de registro de las aplicaciones Entra

Después de crear el registro de la aplicación Entra, anotaste su información, como el id. de la aplicación y del inquilino, así como el secreto. Esta información se usará en el conector para autenticarse con Microsoft 365. Para que esté disponible en el código, almacénelo de forma segura con el proyecto como secretos de usuario.

En un terminal:

1. Asegúrate de que el directorio de trabajo esté establecido en la aplicación de consola recién creada.
1. Para iniciar secretos de usuario, ejecuta `dotnet user-secrets init`.
1. Para almacenar de forma segura la información sobre el registro de la aplicación, reemplaza los tokens por los valores reales que copiaste anteriormente y ejecuta lo siguiente:

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## Tarea 6: Creación de un cliente de Microsoft Graph

Los conectores de Graph personalizados usan la API de Microsoft Graph para administrar su conexión y elementos externos. Para empezar, crea una instancia de la clase `GraphServiceClient` desde el paquete NuGet de **Microsoft.Graph** que instalaste en el proyecto.

1. Abre el proyecto en el editor de código
1. En el proyecto, agrega un nuevo archivo de código denominado **GraphService.cs**.
1. En el archivo, empieza agregando referencias a los espacios de nombres que usarás, agregando:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. A continuación, define una nueva clase denominada GraphService:

   ```csharp
   class GraphService
   {
   }
   ```

1. En la clase `GraphService`, define un singleton para almacenar una instancia de `GraphServiceClient` para que se comunique con las API de Microsoft Graph:

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. En el singleton `Client`, implementa el captador para que cree una nueva instancia de `GraphServiceClient` si aún no existe.

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. Crea una nueva instancia de `GraphServiceClient` utilizando como credencial la información sobre el registro de la aplicación Entra que almacenaste previamente:

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   Empieza con la creación de un generador de configuración para acceder a la información sobre el registro de la aplicación Entra almacenada en secretos de usuario. A continuación, usarás el generador para recuperar la información de registro de la aplicación. A continuación, crea una nueva credencial de secreto de cliente pasando el Id. de cliente y de inquilino y el secreto de cliente. Por último, crea una instancia de `GraphServiceClient` pasando la credencial recién creada.

1. Este es el aspecto del código completo:

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().   AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. Guarde los cambios

## Tarea 7: Definición de la conexión externa y configuración del esquema

El siguiente paso es definir la conexión externa y el esquema que debe usar el conector de Graph. Dado que el código del conector necesita acceso al id. de la conexión externa en varios lugares, almacénalo en un lugar central en el código.

En el editor de código:

1. Crea un archivo de texto denominado **ConnectionConfiguration.cs**.
1. Agrega una referencia al espacio de nombres con modelos de Microsoft Graph:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. A continuación, en el mismo archivo, define una nueva clase estática denominada `ConnectionConfiguration`:

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1. En la clase `ConnectionConfiguration`, agrega una nueva propiedad llamada `ExternalConnection`. Impleméntala para devolver una instancia del modelo de `ExternalConnection` de Microsoft Graph:

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. A continuación, agrega otra propiedad denominada `Schema`. Impleméntala para devolver una instancia del modelo de `Schema` de Graph:

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. En el esquema, define las propiedades que supervisas para cada elemento externo que ingieras utilizando el conector:

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   Comienza con la definición de la propiedad **title**, que almacena el título del elemento externo importado a Microsoft 365. El título del elemento forma parte del índice de texto completo (`IsSearchable = true`). Los usuarios también pueden consultar explícitamente su contenido en consultas de palabras clave (`IsQueryable = true`). El título también se puede recuperar y mostrar en los resultados de la búsqueda (`IsRetrievable = true`). La propiedad **title** representa el título del elemento, que se indica mediante la etiqueta semántica `Title`.

   A continuación, define la propiedad **description**, que almacena el resumen del contenido del elemento externo. Su definición es similar a la del título. Sin embargo, no hay ninguna etiqueta semántica para la descripción, por eso no se define.

   A continuación, define una propiedad para almacenar la dirección URL del icono para cada elemento. Copilot para Microsoft 365 requiere esta propiedad y debe asignarse mediante la etiqueta semántica `IconUrl`.

   Por último, define la propiedad **url**, que almacena la dirección URL original del elemento externo. Los usuarios usan esta dirección URL para navegar al elemento externo desde los resultados de la búsqueda o Copilot desde Microsoft 365. La URL es una de las propiedades que requiere Copilot para Microsoft 365, por lo que se asigna mediante la etiqueta semántica de la `Url`:

1. Este es el aspecto del código completo:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. Guarde los cambios

## Tarea 8: Creación de una conexión externa

Continúa con la adición de código que usa la información sobre la conexión externa que definiste en la sección anterior para crear la conexión externa en Microsoft 365.

En el editor de código:

1. Crea un nuevo archivo llamado **ConnectionService.cs**.
1. En el archivo, empieza por agregar referencias a los espacios de nombres:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. A continuación, en el mismo archivo, define una nueva clase estática denominada `ConnectionService`:

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. Agrega un método llamado `CreateConnection` a la clase `ConnectionService`:

   ```csharp
   async static Task CreateConnection()
   {
   }
   ```

1. En el método `CreateConnection`, usa la instancia de cliente de Microsoft Graph para llamar a las API de Microsoft Graph y crear la conexión externa mediante la información de conexión definida anteriormente:

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. Este es el aspecto del código completo:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. Guarde los cambios.

Antes de probar este código, agrega el código para crear el esquema. De este modo, puedes probar el flujo completo de creación y configuración de la conexión externa.

## Tarea 9: Creación de un esquema de conexión externa

La última parte de la creación de una conexión externa es crear su esquema.

En el editor de código:

1. Abre el archivo **ConnectionService.cs**.
1. En la clase ConnectionService, agrega un nuevo método denominado `CreateSchema`:

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. En el método `CreateSchema`, usa la instancia de cliente de Microsoft Graph para llamar a la API de Microsoft Graph para crear el esquema. A continuación, espera a que se cree.

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. En el mismo archivo, agrega un nuevo método denominado `ProvisionConnection`. En su código, llama a los métodos `CreateConnection` y `CreateSchema` que definiste anteriormente:

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. Este es el aspecto del código completo:

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. Guarde los cambios.

## Tarea 10: Probar el código

El último paso que queda es crear un punto de entrada en la aplicación, que creará la conexión y su esquema. Para ello, crea un comando que se invoca al iniciar la aplicación desde la línea de comandos.

En el editor de código:

1. Abra el archivo **Program.cs**.
1. Reemplace el contenido por el código siguiente:

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   Para empezar, debes definir un comando denominado `provision-connection.`. Este comando invoca el método `ConnectionService.ProvisionConnection` que definiste anteriormente. Por último, registra el comando con el procesador de línea de comandos e inicia los argumentos de supervisión de aplicaciones pasados en la línea de comandos.

1. Guarde los cambios

Para probar la aplicación:

1. Abra un terminal.
1. Cambie el directorio de trabajo a la carpeta del proyecto.
1. Ejecute `dotnet build` para compilar el proyecto.
1. Ejecute `dotnet run -- provision-connection` para iniciar la aplicación.
1. Espera varios minutos para que se cree la conexión y el esquema.

[Ir al ejercicio siguiente...](./3-exercise-import-external-content.md)