---
lab:
  title: 'Ejercicio 3: Recuperación de la información del producto desde SharePoint Online'
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 3: Recuperación de la información del producto desde SharePoint Online

En este ejercicio, aprovisionarás y configurarás un sitio de SharePoint Online, que almacena información del producto como elementos de una lista. Actualizarás el código de extensión de mensajes para recuperar los elementos de la lista de SharePoint Online mediante el SDK de Microsoft Graph y devolverás los datos del elemento de lista en los resultados de búsqueda. Por último, ejecutarás y depurarás la extensión de mensajes y la probarás en Microsoft Teams.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams. Los resultados de la búsqueda se devuelven desde SharePoint Online. Cada resultado de búsqueda muestra el nombre, la categoría y la imagen del producto." lightbox="../media/4-search-results-sharepoint-online.png":::

## Tarea 1: Aprovisionamiento y configuración del sitio de SharePoint de marketing de productos

Empieza creando un sitio de SharePoint Online con el servicio de look book de SharePoint.

En un explorador web:

1. Ve a **SharePoint look book** en [https://lookbook.microsoft.com](https://lookbook.microsoft.com)
1. En el panel de navegación superior, expande **View the designs**
1. En el menú **View the designs**, expande **Team** y selecciona **Product Support**
1. Selecciona **Add you your tenant**
1. Cuando se te solicite, inicia sesión en el inquilino.
1. En la pantalla de consentimiento de permisos, revisa los permisos necesarios y selecciona **Aceptar** para volver al servicio de look book de SharePoint.
1. En el formulario, acepta los valores predeterminados y selecciona **Provision**

Se envía un correo electrónico a tu dirección de correo electrónico para notificarte cuando se complete el aprovisionamiento del sitio. Este proceso puede tardar unos minutos en completarse.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="Captura de pantalla de la página principal del sitio de equipo de SharePoint Online de soporte de productos. Se muestra una lista de productos lanzados recientemente." lightbox="../media/1-sharepoint-online-product-support-site.png":::

Crea índices en la lista para habilitar el filtrado en las columnas Title y Retail Category al consultar la lista mediante Microsoft Graph API.

Continúa en el explorador web:

1. Ve al sitio de **Product support** en **<https://tenant.sharepoint.com/sites/productmarketing>**, reemplaza el **inquilino** por el nombre de la instancia de SharePoint Online.
1. En la **barra del conjunto de aplicaciones de Microsoft 365**, selecciona el **engranaje de configuración** para abrir el panel lateral de Configuración.
1. En el encabezado de **SharePoint**, selecciona **Contenido del sitio**
1. En la lista de listas y bibliotecas, mantén el puntero sobre la lista de **Productos**, selecciona el **icono vertical de tres puntos** para expandir el menú de  **Mostrar acciones** y luego selecciona **Configuración**.
1. En la sección **Columnas**, en la lista de columnas, selecciona **Indexed columns**
1. Selecciona **Create a new index**

## Tarea 2: Agregar el nombre de host de SharePoint y las variables de entorno de la dirección URL del sitio

A continuación, centralizaremos el nombre de host de la instancia de SharePoint Online y la dirección URL del sitio de soporte de productos como variables de entorno. Después expondrás los valores como una variable de entorno para usarla en tiempo de ejecución y actualizarás el código para leer el valor.

Abre Visual Studio:

1. En la carpeta **env**, abre el archivo denominado **.env.local**
1. En el archivo, agrega las variables de entorno **SPO_HOSTNAME** y **SPO_SITE_URL**, reemplaza **tenant** por el nombre de la instancia de SharePoint Online:

    ```text
    SPO_HOSTNAME=tenant.sharepoint.com
    SPO_SITE_URL=sites/productmarketing
    ```

1. Guarda los cambios

Luego actualiza la acción para escribir las variables de entorno en el archivo de configuración de la aplicación.

1. En la carpeta raíz del proyecto, abre el archivo denominado **teamsapp.local.yml**
1. Busca el paso que usa la acción **file/createOrUpdateJsonFile**, que tiene como destino el archivo **./appsettings.Development.json**
1. En el archivo, actualiza la matriz de **contenido** y agrega las variables **SPO_HOSTNAME** y **SPO_SITE_URL**:

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
            SPO_HOSTNAME: ${{SPO_HOSTNAME}}
            SPO_SITE_URL: ${{SPO_SITE_URL}}
    ```

1. Guarda los cambios

Ahora, actualiza la clase ConfigOptions para incluir las nuevas variables de entorno

1. En la carpeta raíz del proyecto, abre Config.cs
1. En la clase ConfigOptions, agrega nuevas propiedades de cadena con los nombres SPO_HOSTNAME y SPO_SITE_URL

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
      public string SPO_HOSTNAME { get; set; }
      public string SPO_SITE_URL { get; set; }
    }
    ```

1. Guarda los cambios

Luego actualiza la configuración de la aplicación con las dos variables de entorno.

1. En la carpeta raíz del proyecto abre Program.cs
1. Agrega nuevas líneas para agregar las variables de entorno SPO_HOSTNAME y SPO_SITE_URL como opciones de configuración de la aplicación.

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    builder.Configuration["SPO_HOSTNAME"] = config.SPO_HOSTNAME;
    builder.Configuration["SPO_SITE_URL"] = config.SPO_SITE_URL;
    ```

1. Guarda los cambios

El último paso es actualizar el controlador de actividad del bot para leer los valores de la configuración de la aplicación y almacenar el valor en propiedades de solo lectura.

1. En la carpeta Search, abre el archivo denominado SearchApp.cs
1. En la clase SearchApp, crea propiedades de cadena de solo lectura con los nombres spoHostname y spoSiteUrl

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
      private readonly string spoHostname;
      private readonly string spoSiteUrl;
    }
    ```

1. Actualiza el constructor para establecer los valores de propiedad mediante la App Configuration insertada:

    ```csharp
    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    } 
    ```

1. Guarda los cambios.

## Tarea 3: Actualización del comando de búsqueda

A medida que la extensión de mensajes devuelva información de los productos, actualizarás el título y la descripción del comando de búsqueda, actualizarás también el nombre del parámetro y su descripción.

Continúa en Visual Studio:

1. En la carpeta **appPackage**, abre el archivo denominado **manifest.json**
1. En la matriz **composeExtensions**, actualiza el objeto de comando con:

    ```json
    "composeExtensions": [
      {
        "botId": "${{BOT_ID}}",
        "commands": [
          {
            "id": "Search",
            "type": "query",
            "title": "Products",
            "description": "Find products by name",
            "initialRun": false,
            "fetchTask": false,
            "context": [
              "commandBox",
              "compose",
              "message"
            ],
            "parameters": [
              {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
              }
            ]
          }
        ]
      }
    ],
    ```

1. Guarda los cambios

## Tarea 4: Obtención del valor de la consulta de usuario

Cuando se ejecuta el método OnTeamsMessagingExtensionQueryAsync, lo primero que queremos hacer es comprender lo que escribió el usuario en el cuadro de búsqueda.

Primero vamos a eliminar el código existente.

Continúa en Visual Studio:

1. En la carpeta **Search**, abre el archivo denominado **SearchApp.cs**
1. En el método **OnTeamsMessagingExtensionQueryAsync**, elimina todo el código **después** de la instrucción **if** que comprueba si hay un token de acceso.
1. En la clase **SearchApp**, elimina el método **FindPackages** y la propiedad **_adaptiveCardFilePath**.

Después de eliminar el código existente, la clase **SearchApp** debe ser parecido al siguiente fragmento de código:

```csharp
public class SearchApp : TeamsActivityHandler
{
    private readonly string connectionName;
    private readonly string spoHostname;
    private readonly string spoSiteUrl;

    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    }

    protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
    {
        var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
        var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);

        if (!HasToken(tokenResponse))
        {
            return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
        }
    }
}
```

Ahora escribiremos el código para obtener el valor del parámetro **ProductName**.

1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega código para recuperar el valor del parámetro **ProductName** de la matriz **Parameters** en el objeto **MessagingExtensionQuery**

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    ```

1. En la clase **SearchApp**, implementa el método **GetQueryData**.

    ```csharp
    private static string GetQueryData(IList<MessagingExtensionParameter> parameters, string key)
    {
      if (parameters.Any() != true)
      {
        return string.Empty;
      }
    
      var foundPair = parameters.FirstOrDefault(pair => pair.Name == key);
      return foundPair?.Value?.ToString() ?? string.Empty;
    }
    ```

1. Guarda los cambios

El método **GetQueryData** se usa para recuperar el valor asociado a una clave específica de una lista de objetos **MessagingExtensionParameter**. Proporciona una manera cómoda de extraer datos de la matriz de parámetros en el objeto **MessagingExtensionQuery**.

## Tarea 5: Creación del filtro de consulta de lista en OData de SharePoint

Ahora que tenemos el valor pasado por el usuario, usa este valor para crear un filtro de consulta en OData. El filtro se usa para consultar la lista de SharePoint Online por la columna Title, que contiene el nombre del producto.

Continúa en Visual Studio:

1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega código para crear la variable **filterQuery**.

    ```csharp
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Guarda los cambios

El código construye una consulta de filtro basada en el parámetro de **nombre**. Si se proporciona el parámetro de nombre, se crea una expresión de filtro para buscar elementos con un campo **Title** a partir del nombre proporcionado. Si no se proporciona el parámetro de nombre, se asigna una cadena vacía como consulta de filtro. La consulta de filtro resultante se usa más adelante en el código para recuperar elementos filtrados de un sitio de SharePoint.

## Tarea 6: Instalación y configuración del SDK de Microsoft Graph

Para ejecutar solicitudes autenticadas en Microsoft Graph, usa el **SDK de Microsoft Graph**.

Instala el paquete del SDK de Microsoft Graph desde NuGet, luego crea una clase **TokenProvider**, que te permite usar el token de acceso obtenido del servicio de token y después inicializa un nuevo **GraphServiceClient**.

Continúa en Visual Studio:

1. En el Explorador de soluciones, haz clic con el botón derecho en el proyecto **MsgExtProductSupport**
1. Selecciona **Administrar paquetes NuGet…**
1. Selecciona la pestaña **Examinar** y busca **Microsoft.Graph**
1. En la lista de resultados, selecciona **Microsoft.Graph**
1. En el elemento desplegable **Versión**, selecciona **5.42.0**
1. Selecciona **Instalar**
1. En el cuadro de diálogo **Aceptación de licencia**, selecciona **Acepto** para instalar el SDK

Después de instalar el paquete, crea un proveedor de tokens para el SDK de Microsoft Graph.

1. En la carpeta **Search**, crea un nuevo archivo denominado **TokenProvider.cs**
1. En el archivo agrega el código siguiente:

    ```csharp
    using Microsoft.Kiota.Abstractions.Authentication;
    
    namespace MsgExtProductSupport.Search
    {
       public class TokenProvider : IAccessTokenProvider
        {
            public string Token { get; set; }
            public AllowedHostsValidator AllowedHostsValidator => throw new NotImplementedException();
    
            public Task<string> GetAuthorizationTokenAsync(Uri uri, Dictionary<string, object>? additionalAuthenticationContext = null, CancellationToken cancellationToken = default)
            {
                return Task.FromResult(Token);
            }
        }
    }
    ```

1. Guarda los cambios

Ahora crea un método para crear una nueva instancia de **GraphServiceClient**.

1. En la carpeta **Search**, abre el archivo denominado **SearchApp.cs**
1. En el archivo, importa los espacios de nombres que necesitas:

    ```csharp
    using Microsoft.Graph;
    using Microsoft.Kiota.Abstractions.Authentication;
    ```

1. En el **método OnTeamsMessagingExtensionQueryAsync**, agregue código para crear un nuevo cliente de Graph, que usarás para enviar solicitudes a Microsoft Graph.

    ```csharp
    var graphClient = CreateGraphClient(tokenResponse);
    ```

1. En la clase **SearchApp**, implementa el método **CreateGraphClient**.

    ```csharp
    private static GraphServiceClient CreateGraphClient(TokenResponse tokenResponse)
    {
      TokenProvider provider = new() { Token = tokenResponse.Token };
      var authenticationProvider = new BaseBearerTokenAuthenticationProvider(provider);
      var graphClient = new GraphServiceClient(authenticationProvider);
      return graphClient;
    }
    ```

1. Guarda los cambios

Este código configura el proveedor de autenticación y crea un objeto de cliente que se puede usar para interactuar con Microsoft Graph API mediante el token de acceso proporcionado.

## Tarea 7: Consultar la lista de productos

Para consultar los elementos de la lista productos y versiones posteriores, crearás los resultados de búsqueda, usarás GraphServiceClient para enviar solicitudes para obtener datos de productos de SharePoint Online.

Continúa en Visual Studio:

En el método **OnTeamsMessagingExtensionQueryAsync**, agrega código para obtener los datos de SharePoint:

  ```csharp
  var site = await GetSharePointSite(graphClient, spoHostname, spoSiteUrl, cancellationToken);
  var drive = await GetSharePointDrive(graphClient, site.SharepointIds.SiteId, "Product Imagery", cancellationToken);
  var items = await GetProducts(graphClient, site.SharepointIds.SiteId, filterQuery, cancellationToken);
  ```

Este código hará lo siguiente:

- **Obtener el sitio de marketing de productos**, el objeto de sitio contiene el id. del sitio de SharePoint, que se usa para devolver y consultar objetos en el sitio.
- **Obtener la unidad de imágenes de producto**, la unidad representa la biblioteca de documentos que contiene las imágenes de producto. La unidad se usa más adelante para obtener las imágenes de producto que se muestran en los resultados de la búsqueda
- **Obtener los productos**, que se usa para consultar la lista de productos en función de la consulta del usuario.

Implementa los tres métodos en la clase **SearchApp**.

- Implementa el método **GetSharePointSite**

    ```csharp
    private static async Task<Site> GetSharePointSite(GraphServiceClient graphClient, string hostName, string siteUrl, CancellationToken cancellationToken)
    {
        return await graphClient.Sites[$"{hostName}:/{siteUrl}"].GetAsync(r => r.QueryParameters.Select = new string[] { "sharePointIds" }, cancellationToken);
    }
    ```

Este método usa GraphServiceClient para enviar una solicitud a Microsoft Graph para devolver el objeto de sitio mediante una ruta de acceso. La ruta de acceso se crea a partir de la combinación del nombre de host y la dirección URL del sitio de SharePoint Online. Como solo necesitas los valores de propiedad de sharePointIds, el parámetro de seleccionar consulta está configurado para devolver solo esta propiedad en la respuesta.

- Implementa el método **GetSharePointDrive**

    ```csharp
    private static async Task<Drive> GetSharePointDrive(GraphServiceClient graphClient, string siteId, string name, CancellationToken cancellationToken)
    {
        var drives = await graphClient.Sites[siteId].Drives.GetAsync(r => r.QueryParameters.Select = new string[] { "id", "name" }, cancellationToken);
        var drive = drives.Value.Find(d => d.Name == name);
        return drive;
    }
    ```

Este método usa GraphServiceClient y el id. de sitio para devolver una colección de bibliotecas de documentos del sitio, devolviendo las propiedades de id. y nombre de cada biblioteca de documentos. A continuación, se filtra la colección de bibliotecas para devolver la unidad, que tiene el mismo nombre que el parámetro de método de nombre.

- Implementa el método **GetProducts**

    ```csharp
    private static async Task<SiteCollectionResponse> GetProducts(GraphServiceClient graphClient, string siteId, string filterQuery, CancellationToken cancellationToken)
    {
        var fields = new string[]
        {
            "fields/Id",
            "fields/Title",
            "fields/RetailCategory",
            "fields/PhotoSubmission",
            "fields/CustomerRating",
            "fields/ReleaseDate"
        };
    
        var request = graphClient.Sites.WithUrl($"https://graph.microsoft.com/v1.0/sites/{siteId}/lists/Products/items?expand={string.Join(",", fields)}&$filter={filterQuery}");
        return await request.GetAsync(null, cancellationToken);
    }
    ```

Este método usa GraphServiceClient para devolver elementos de lista filtrados de la lista de productos mediante la consulta de filtro pasada y devuelve los datos del elemento de lista definidos en la matriz de campos.

## Tarea 8: Creación de resultados de búsqueda

Después de obtener los productos de SharePoint, crearás los resultados de búsqueda, que se devolverán al usuario.

La creación de los resultados de la búsqueda consiste en iterar en la matriz de elementos, para cada elemento, se crea un MessagingExtensionAttachment, que contiene la vista previa y las tarjetas de contenido.

Antes de iterar los elementos, crearás una plantilla de tarjeta adaptable que usarás en el bucle.

Continúa en Visual Studio:

1. En la carpeta **Resources**, crea un nuevo archivo denominado **Product.json**

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.6",
      "body": [
        {
          "type": "TextBlock",
          "text": "${Product.Title}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${Product.RetailCategory}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${ProductImage}",
              "altText": "${Product.Title}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(Product.CustomerRating,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(Product.ReleaseDate,'dd/MM/yyyy')}"
            }
          ]
        },
        {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View",
              "url": "https://${SPOHostname}/${SPOSiteUrl}/Lists/Products/DispForm.aspx?ID=${Product.Id}"
            }
          ]
        }
      ]
    }
    ```

La plantilla usa expresiones de enlace de datos, que se reemplazan por valores reales al representar la tarjeta adaptable. Cuando se representa la tarjeta, contiene información del producto, una imagen del producto y un botón de acción. El botón de acción abre un explorador que navega al formulario para mostrar la lista de SharePoint para el producto.

Luego agrega el código para transformar el archivo JSON en una plantilla de tarjeta adaptable.

1. En la carpeta **Search**, abre **SearchApp.cs**
1. En el archivo importa los espacios de nombres que necesitas:

    ```csharp
    using AdaptiveCards.Templating;
    ```

1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega el código para leer el contenido del archivo JSON y crea un nuevo objeto **AdaptiveCardTemplate**.

    ```csharp
    var card = File.ReadAllText(@"Resources\Product.json");
    var template = new AdaptiveCardTemplate(card);
    ```

1. Guarda los cambios

Después crea un bucle para iterar los elementos de lista. Cada iteración hará lo siguiente:

- Deserializar los datos del elemento actual en un objeto de producto
- Obtener la miniatura de la imagen del producto
- Crear una tarjeta de contenido
- Crear una tarjeta de vista previa
- Crear un messagingExtensionAttachment combinando el contenido y las tarjetas de vista previa
- Agregar el MessagingExtensionAttachment a la lista

Una vez finalizado el bucle; tendrás una lista de datos adjuntos que se pueden devolver al usuario.

1. En la carpeta **Search**, abre **SearchApp.cs**
1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega código para crear una nueva lista, para almacenar objetos **MessagingExtensionAttachment** en

    ```csharp
    var attachments = new List<MessagingExtensionAttachment>();
    ```

1. Crea un bucle foreach para recorrer en iteración los elementos de lista.

    ```csharp
    foreach (var item in items.Value) { 
            
    }
    ```

1. Agrega el código siguiente al bucle foreach:

    ```csharp
    var product = JsonConvert.DeserializeObject<Product>(item.AdditionalData["fields"].ToString());
    product.Id = item.Id;
    
    var thumbnails = await GetThumbnails(graphClient, drive.Id, product.PhotoSubmission, cancellationToken);
    
    var resultCard = template.Expand(new
    {
      Product = product,
      ProductImage = thumbnails.Large.Url,
      SPOHostname = spoHostname,
      SPOSiteUrl = spoSiteUrl,
    });
    
    var previewcard = new ThumbnailCard
    {
      Title = product.Title,
      Subtitle = product.RetailCategory,
      Images = new List<CardImage> { new() { Url = thumbnails.Small.Url } }
    }.ToAttachment();
    
    var attachment = new MessagingExtensionAttachment
    {
      Content = JsonConvert.DeserializeObject(resultCard),
      ContentType = AdaptiveCard.ContentType,
      Preview = previewcard
    };
    
    attachments.Add(attachment);
    ```

1. Guarda los cambios

Para los datos del elemento de lista, estén fuertemente tipados, crea un modelo que represente el **producto**.

1. Crea una carpeta denominada **Models** en la carpeta raíz del proyecto
1. Crea un nuevo archivo denominado **Product.cs** en la carpeta **Models**

    ```csharp
    namespace MsgExtProductSupport.Models
    {
        public class Product
        {
            public string Title { get; set; }
            public string RetailCategory { get; set; }
            public Link Specguide { get; set; }
            public string PhotoSubmission { get; set; }
            public double CustomerRating { get; set; }
            public DateTime ReleaseDate { get; set; }
            public string Id { get; set; }
            public string ContentType { get; set; }
            public DateTime Modified { get; set; }
            public DateTime Created { get; set; }
        }
    
        public class Link
        {
            public string Description { get; set; }
            public string Url { get; set; }
        }
    }
    ```

1. Guarda los cambios

Luego implementa el método **GetThumbnails** para recuperar imágenes en miniatura de Microsoft Graph para un producto.

1. En la carpeta **Search**, abre el archivo denominado **SearchApp.cs**
1. Crea el método **GetThumbnails** en la clase **SearchApp**

    ```csharp
    private static async Task<ThumbnailSet> GetThumbnails(GraphServiceClient graphClient, string driveId, string photoUrl, CancellationToken cancellationToken)
    {
        var fileName = photoUrl.Split('/').Last();
        var driveItem = await graphClient.Drives[driveId].Root.ItemWithPath(fileName).GetAsync(null, cancellationToken);
        var thumbnails = await graphClient.Drives[driveId].Items[driveItem.Id].Thumbnails["0"].GetAsync(r => r.QueryParameters.Select = new string[] { "small", "large" }, cancellationToken);
        return thumbnails;
    }
    ```

1. Guarda los cambios

El método **GetThumbnails** usa el punto de conexión de miniaturas de Microsoft Graph API para devolver miniaturas pequeñas y grandes de la imagen del producto almacenada en SharePoint.

## Tarea 9: Devolución de resultados de búsqueda

Ahora que tenemos una colección de objetos MessagingExtensionResult, podemos devolverlos al usuario como resultados de búsqueda.

- En el método **OnTeamsMessagingExtensionQueryAsync**, agrega código para devolver los resultados de búsqueda como respuesta de extensión de mensajes.

    ```csharp
    return new MessagingExtensionResponse
    {
      ComposeExtension = new MessagingExtensionResult
      {
        Type = "result",
        AttachmentLayout = "list",
        Attachments = attachments
      }
    };
    ```

## Tarea 10: Aprovisionamiento de recursos

Ejecuta el proceso de preparar dependencias de la aplicación de Teams para aprovisionar recursos.

Continúa en Visual Studio:

1. En el **Explorador de soluciones**, haz clic con el botón derecho en el proyecto **MsgExtProductSupport**
1. Expande el menú del **kit de herramientas de teams** y selecciona **Preparar dependencias de la aplicación de Teams**
1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona **Continuar**
1. En el cuadro de diálogo **Provision**, selecciona **Provision**
1. En el cuadro de diálogo **Teams Toolkit warning**, selecciona **Provision**
1. En el cuadro de diálogo **Teams Toolkit information**, **cierra** la solicitud.

## Tarea 11: Ejecución y depuración

Ahora inicia el servicio web y prueba la extensión de mensajes en Microsoft Teams.

Continúa en Visual Studio:

1. Presiona **F5** para iniciar una sesión de depuración y abrir una nueva ventana del explorador que vaya al cliente web de Microsoft Teams.
1. Escribe tus credenciales de la cuenta de Microsoft 365 y continúa con Microsoft Teams.
1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**
1. Abre un chat de Microsoft Teams nuevo o existente
1. En el área de redacción del mensaje, selecciona **...** para abrir el control flotante de la aplicación
1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes
1. Escribe **Mark8** en el cuadro de texto. Se muestran dos resultados, **Mark8** y el **controlador de Mark8**
1. Selecciona **Mark8** para insertar una tarjeta en el cuadro de redacción del mensaje
1. **Envía el mensaje** que contiene la tarjeta
1. En la tarjeta enviada, selecciona el botón **Vista** para ver el elemento de lista de SharePoint del producto en la lista de productos en una nueva pestaña.

Cierra el explorador para detener la sesión de depuración.

[Ir al ejercicio siguiente...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)