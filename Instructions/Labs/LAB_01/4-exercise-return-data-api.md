---
lab:
  title: 'Ejercicio 3: Devolución de datos de productos de la API protegida por Microsoft Entra'
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 3: Devolución de datos de productos de la API protegida por Microsoft Entra

En este ejercicio, actualizarás la extensión de mensajes para recuperar datos de una API personalizada. Obtienes datos de la API personalizada en función de la consulta de usuario y devuelves datos en los resultados de búsqueda al usuario.

![Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams.](../media/3-search-results-api.png)

### Duración del ejercicio

  - **Tiempo estimado para completarla:** 50 minutos

## Tarea 1: Instalación y configuración de Dev Proxy

En este ejercicio, usarás Dev Proxy, una herramienta de línea de comandos que puede simular API. Resulta útil cuando quieres probar la aplicación sin tener que crear una API real.

Para completar este ejercicio, debes instalar la versión más reciente del [proxy de desarrollo](/microsoft-cloud/dev/dev-proxy/get-started) y descargar el valor preestablecido del proxy de desarrollo para este módulo.

El valor preestablecido simula una API CRUD (Crear, Leer, Actualizar, Eliminar) con un almacén de datos en memoria, que está protegido por Microsoft Entra. Esto significa que puedes probar la aplicación como si llamaras a una API real que requiere autenticación.

1. Para instalar el proxy de desarrollo, abre una nueva **ventana del símbolo del sistema como administrador**:

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. Ejecuta el siguiente texto de comando para descargar el valor preestablecido:

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. Mantén abierta la ventana de símbolo del sistema para usarla más adelante.

## Tarea 2: Obtención del valor de la consulta de usuario

Crea un método que obtenga el valor de consulta de usuario por el nombre del parámetro.

En Visual Studio, abre el proyecto **ProductsPlugin**.

1. En la carpeta **Helpers**, crea un nuevo archivo denominado **MessageExtensionHelpers.cs**.

1. En el archivo, reemplaza el código con lo siguiente:

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. Guarda los cambios.

A continuación, actualiza el método **OnTeamsMessagingExtensionQueryAsync** en la clase SearchApp para usar el nuevo método auxiliar.

1. En la carpeta **Search**, abre **SearchApp.cs**.

1. En el método **OnTeamsMessagingExtensionQueryAsync**, reemplaza el siguiente código:

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   con

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. Mueve el cursor a la variable **text**, usa `Ctrl + R`, `Ctrl + R` y cambia el nombre de la variable por **name**.

1. Presiona **Entrar** para cambiar el nombre de la variable en 3 archivos.

1. Guarda los cambios.

Ahora, el método **OnTeamsMessagingExtensionQueryAsync** debería ser similar a lo que se ve en esta imagen:

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## Tarea 3: Obtención de datos de la API personalizada

Para obtener datos de la API personalizada, debes enviar el token de acceso en el encabezado Autorización de la solicitud y deserializar la respuesta en un modelo que represente los datos del producto.

Primero, crea un modelo que represente los datos del producto que se devuelven desde la API personalizada.

En Visual Studio, abre el proyecto **ProductsPlugin**.

1. Crea una carpeta con el nombre **Models**.

1. Crea un nuevo archivo denominado **Product.cs** en la carpeta **Models**.

1. En el archivo, reemplaza todo el código existente por lo siguiente:

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. Guarda los cambios.

A continuación, crea una clase de servicio que recupere los datos del producto de la API personalizada.

En Visual Studio, abre el proyecto **ProductsPlugin**.

1. Crea una carpeta denominada **Services**.

1. En la carpeta **Services**, crea un nuevo archivo denominado **ProductService.cs**.

1. En el archivo, reemplaza todo el código existente por lo siguiente:

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. Guarda los cambios.

La clase **ProductsService** contiene métodos para obtener datos de producto de la API personalizada. El constructor de clase toma un token de acceso como parámetro y configura una instancia **httpClient** con el token de acceso en el encabezado de autorización.

A continuación, actualiza el método **OnTeamsMessagingExtensionQueryAsync** para usar la clase **ProductsService** para obtener datos de producto de la API personalizada.

1. En la carpeta **Search**, abre **SearchApp.cs**.

1. En el método **OnTeamsMessagingExtensionQueryAsync**, agrega el código siguiente después de la declaración de variable **name** para obtener datos de producto de la API personalizada:

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. Guarda los cambios.

## Tarea 4: Creación de resultados de búsqueda

Ahora que tienes los datos del producto, puedes incluirlos en los resultados de búsqueda que se devuelven al usuario.

Primero, vamos a actualizar la plantilla de tarjeta adaptable existente para mostrar la información del producto.

Continúa en Visual Studio y en el proyecto **ProductsPlugin**:

1. En la carpeta **Resources**, cambia el nombre de **card.json** a **Product.json**.

1. En la carpeta **Resources**, crea un nuevo archivo denominado **Product.data.json**. Este archivo contiene datos de ejemplo que Visual Studio usa para generar una vista previa de la plantilla de tarjeta adaptable.

1. En el archivo, agrega el siguiente JSON:

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. Guarda los cambios.

1. En la carpeta **Resources**, abre **Product.json**.

1. En el archivo, reemplaza el contenido por el JSON siguiente:

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
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
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. Guarda los cambios.

La plantilla de tarjeta adaptable usa expresiones de enlace para mostrar la información del producto. Las expresiones**\$\{name\}**, **\$\{category\}**, **\$\{imageUrl\}**, **\$\{callVolume\}** y **\$\{releaseDate\}** se reemplazan por los valores correspondientes de los datos de producto. Las funciones de plantilla **formatNumber** y **formatDateTime** se usan para dar formato a los valores **callVolume** y **releaseDate**, en un número y una fecha respectivamente.

Dedica un momento a explorar la vista previa de la tarjeta adaptable en Visual Studio. En la vista previa se muestra cómo se ve la plantilla tarjeta adaptable cuando los datos del producto están enlazados a la plantilla. Usa los datos de ejemplo del archivo **Product.data.json** para generar la vista previa.

A continuación, actualiza la propiedad **validDomains** en el manifiesto de la aplicación para incluir el dominio **raw.githubusercontent.com**, de modo que las imágenes de la plantilla de tarjeta adaptable se puedan mostrar en Microsoft Teams.

En el proyecto **TeamsApp**:

1. En la carpeta **appPackage**, abre **manifest.json**

1. En el archivo, agrega el dominio de GitHub a la propiedad **validDomains**:

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. Guarda los cambios.

A continuación, actualiza el método **OnTeamsMessagingExtensionQueryAsync** para crear una lista de datos adjuntos que contengan la información del producto.

En el proyecto **ProductsPlugin**:

1. En la carpeta **Search**, abre **SearchApp.cs**.

1. Actualiza **card.json** a **Product.json** para reflejar el cambio en el nombre de archivo. En la línea 34, reemplace el siguiente código:

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   con

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. Agrega el código siguiente después de la declaración de variable **template** para crear una lista de datos adjuntos:

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. Actualiza la instrucción **return** para incluir la variable **attachments**:

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

1. Guardar los cambios

## Tarea 5: Creación y actualización de recursos

Ahora que está todo preparado, ejecuta el proceso **Preparar dependencias de la aplicación de Teams** para crear nuevos recursos y actualizar los existentes.

Continúa en Visual Studio:

1. En **Explorador de soluciones**, haz clic con el botón derecho en el proyecto **TeamsApp**

1. Expande el menú del **kit de herramientas de teams** y selecciona **Preparar dependencias de la aplicación de Teams**.

1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona **Continuar**.

1. En el cuadro de diálogo **Aprovisionar**, selecciona **Aprovisionar**.

1. En el cuadro de diálogo **Advertencia del kit de herramientas de Teams**, selecciona **Aprovisionar**.

1. En el cuadro de diálogo **Información del kit de herramientas de Teams**, selecciona el icono de la cruz para cerrar el cuadro de diálogo.

## Tarea 6: Ejecución y depuración

Con los recursos aprovisionados, inicia una sesión de depuración para probar la extensión de mensajes.

Primero, inicia el proxy de desarrollo para simular la API personalizada.

1. En la **ventana del símbolo del sistema** que todavía tienes abierta, ejecuta el siguiente comando para iniciar el proxy de desarrollo:

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. Si se te solicita, acepta las advertencias del certificado.

> [!NOTE]
> Cuando se ejecuta el proxy de desarrollo, actúa como proxy de todo el sistema.

A continuación, inicia una sesión de depuración de Visual Studio:

1. Para iniciar una nueva sesión de depuración, presiona <kbd>F5</kbd> o selecciona **Iniciar** en la barra de herramientas.

1. Espera hasta que se abra una ventana del explorador y aparezca el cuadro de diálogo de instalación de la aplicación en el cliente web de Microsoft Teams. Si se te solicita, escribe las credenciales de tu cuenta de Microsoft 365.

1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**.

1. Abre un chat de Microsoft Teams nuevo o existente

1. En el área de redacción de mensajes, escribe **/apps** para abrir el selector de aplicaciones.

1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes

1. Escribe **mark8** en el cuadro de texto. Es posible que tengas que escribir la consulta de búsqueda varias veces.

1. Espera a que se complete la búsqueda y se muestren los resultados.

    ![Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams.](../media/3-search-results-api.png)

1. En la lista de resultados, selecciona un resultado para insertar una tarjeta en el cuadro de redacción de mensajes

Vuelve a Visual Studio y selecciona **Detener** en la barra de herramientas o presiona <kbd>Mayús</kbd> + <kbd>F5</kbd> para detener la sesión de depuración. Además, apaga el proxy de desarrollo mediante <kbd>Ctrl</kbd> + <kbd>C</kbd>.

[Ir al ejercicio siguiente...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)