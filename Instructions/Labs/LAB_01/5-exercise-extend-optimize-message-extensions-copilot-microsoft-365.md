---
lab:
  title: 'Ejercicio 4: Ampliación y optimización de las extensiones de mensajes para su uso con Copilot para Microsoft 365'
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 4: Ampliación y optimización de las extensiones de mensajes para su uso con Copilot para Microsoft 365

En este ejercicio, ampliarás y optimizarás la extensión de mensajes para su uso con Copilot para Microsoft 365. Agregarás un nuevo parámetro denominado Target Audience y actualizarás la lógica de la extensión de mensajes para controlar varios parámetros. Por último, ejecutarás y depurarás la extensión de mensajes y la probarás en Copilot en Microsoft Teams.

## Tarea 1: Actualización del manifiesto de la aplicación

Especificar descripciones concisas y precisas en el manifiesto de la aplicación es crítico para garantizar que Copilot sepa cuándo y cómo invocar el complemento. Actualiza las descripciones de la aplicación, el comando y los parámetros en el manifiesto de la aplicación.

Abre Visual Studio:

1. En la carpeta **appPackage**, abre el archivo denominado **manifest.json**
1. Actualiza el objeto **description**

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    Como vamos a agregar otro parámetro al comando, actualiza también la descripción del comando para incluir el nuevo parámetro.

1. En la matriz de **comandos**, actualiza la **descripción** del comando.

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    Ahora agrega un nuevo parámetro que Copilot pueda usar. Este nuevo parámetro ayuda a los usuarios a buscar productos mediante Copilot que está dirigido a diferentes audiencias, como particulares y empresas.

1. En la matriz de **parámetros**, agrega el parámetro **TargetAudience** después del parámetro **ProductName**.

    ```json
    {    
        "parameters": [
            {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
            },
            {
                "name": "TargetAudience",
                "title": "Target audience",
                "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
                "inputType": "text"
            }
        ]
    }
    ```

1. Guarda los cambios

La descripción del parámetro **TargetAudience** describe lo que es y explica que el parámetro debe aceptar que **Consumer** o **Enterprise** son valores permitidos.

## Tarea 2: Actualización de la lógica de la extensión de mensajes

Para admitir el nuevo parámetro y las solicitudes complejas, actualiza el método OnTeamsMessagingExtensionQueryAsync en el controlador de actividad del bot para controlar varios parámetros.

Supongamos que un usuario escribe el mensaje "Find Contoso products aimed at individuals with the name Mark8". Dadas las descripciones del parámetro, "aimed at individuals" se traduce a **Consumer** y se pasa como valor del parámetro **TargetAudience**. "Mark8" se pasa como valor del parámetro **ProductName**.

Continúa en Visual Studio:

1. En la carpeta **Search**, abre el archivo denominado **SearchApp.cs**
1. En el método **OnTeamsMessagingExtensionQueryAsync**, busca el siguiente bloque de código:

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. Actualiza el bloque de código para obtener el valor del parámetro **TargetAudience** y crea una consulta de filtro que se usará al consultar la lista de SharePoint Online.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Guarda los cambios

## Tarea 3: Aprovisionamiento de recursos

Ejecuta el proceso de preparar dependencias de la aplicación de Teams para aprovisionar recursos.

Continúa en Visual Studio:

1. En **Explorador de soluciones**, haz clic con el botón derecho en el proyecto **MsgExtProductSupport**
1. Expande el menú del **kit de herramientas de teams** y selecciona **Preparar dependencias de la aplicación de Teams**
1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona **Continuar**
1. En el cuadro de diálogo **Provision**, selecciona **Provision**
1. En el cuadro de diálogo **Teams Toolkit warning**, selecciona **Provision**
1. En el cuadro de diálogo **Teams Toolkit information**, **cierra** la solicitud.

## Tarea 4: Ejecución y depuración

Ahora inicia el servicio web y prueba la extensión de mensajes en Copilot para Microsoft 365.

1. Presiona **F5** para iniciar una sesión de depuración y abrir una nueva ventana del explorador que vaya al cliente web de Microsoft Teams.
1. Escribe tus credenciales de la cuenta de Microsoft 365 y continúa con Microsoft Teams.
1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**
1. Abre la aplicación **Copilot** en Microsoft Teams
1. En el área de redacción de mensajes, abre el control flotante **Complementos**
1. En la lista de complementos, cambia al complemento **Contoso products** para habilitarlo.
1. Escribe **Find Contoso products aimed at individuals** como tu mensaje y envíalo
1. En la respuesta de Copilot, se devuelve un botón de inicio de sesión, selecciona el botón de **inicio de sesión** para autenticarte
1. Después de autenticarte correctamente, **escribe Find Contoso products aimed at individuals** como tu mensaje y envíalo
1. En la respuesta de Copilot, se muestran los datos devueltos en la respuesta del complemento y se hace referencia a éste en la respuesta.
1. Para ver la tarjeta adaptable relevante para el resultado, mantén el puntero sobre las referencias en la respuesta de Copilot.

Cierra el explorador para detener la sesión de depuración.

[Ir al resumen del laboratorio...](./6-summary.md)