---
lab:
  title: 'Ejercicio 4: Exploración del código fuente del complemento'
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# Ejercicio 4: Exploración del código fuente del complemento

En este ejercicio, revisarás el código de la aplicación para que puedas comprender cómo funciona una **extensión de mensajes**.

## Tarea 1: Examen del manifiesto

La base de cualquier aplicación de Microsoft 365 es su manifiesto de aplicación. Aquí es donde proporcionas la información que Microsoft 365 necesita para acceder a la aplicación.

En el **directorio de trabajo**, abre el archivo **appPackackage/manifest.json**. Este archivo JSON se coloca en un archivo ZIP con dos archivos de icono para crear el paquete de aplicación. La propiedad **icons** incluye rutas de acceso a estos iconos.

```json
"icons": {
    "color": "Northwind-Logo3-192-${{TEAMSFX_ENV}}.png",
    "outline": "Northwind-Logo3-32.png"
},
```

Observa el token `${{TEAMSFX_ENV}}` en uno de los nombres de icono. El kit de herramientas de Teams reemplazará este token por el nombre de entorno, como **local** o **dev** (para una implementación de Azure en desarrollo). Por lo tanto, el color del icono cambiará en función del entorno.

### Descripción de la aplicación

Ahora mira el **nombre** y la **descripción**. Observa que la **descripción** es bastante larga. Esto es importante para que tanto los usuarios como Copilot puedan saber lo que hace la aplicación y cuándo usarla.

```json
    "name": {
        "short": "Northwind Inventory",
        "full": "Northwind Inventory App"
    },
    "description": {
        "short": "App allows you to find and update product inventory information",
        "full": "Northwind Inventory is the ultimate tool for managing your product inventory. With its intuitive interface and powerful features, you'll be able to easily find your products by name, category, inventory status, and supplier city. You can also update inventory information with the app. \n\n **Why Choose Northwind Inventory:** \n\n Northwind Inventory is the perfect solution for businesses of all sizes that need to keep track of their inventory. Whether you're a small business owner or a large corporation, Northwind Inventory can help you stay on top of your inventory management needs. \n\n **Features and Benefits:** \n\n - Easy Product Search through Microsoft 365 Copilot. Simply start by saying, 'Find northwind dairy products that are low on stock' \r - Real-Time Inventory Updates: Keep track of inventory levels in real-time and update them as needed \r  - User-Friendly Interface: Northwind Inventory's intuitive interface makes it easy to navigate and use \n\n **Availability:** \n\n To use Northwind Inventory, you'll need an active Microsoft 365 account . Ensure that your administrator enables the app for your Microsoft 365 account."
    },
```

### Definición del bot

Desplácate hacia abajo un poco a **composeExtensions**. La extensión Compose es el término histórico para la extensión de mensajes; aquí es donde se definen las extensiones de mensajes de la aplicación. Las extensiones de mensajes se comunican mediante Azure Bot Framework; esto proporciona un canal de comunicación rápido y seguro entre Microsoft 365 y la aplicación. Cuando ejecutaste el proyecto por primera vez, el kit de herramientas de Teams registró un bot y colocará su **botID** aquí.

```json
    "composeExtensions": [
        {
            "botId": "${{BOT_ID}}",
            "commands": [
                {
                    ...
```

### Definiciones de comando

Esta extensión de mensajes tiene dos comandos, que se definen en la matriz `commands`. Si completaste el ejercicio anterior, también habrá un tercer comando para buscar por nombre de empresa. Omitamos el primer comando de momento, ya que es el más complejo. El siguiente comando permite a Copilot (o a un usuario) buscar productos con descuento dentro de una categoría Northwind. Este comando acepta un único parámetro, **categoryName**.

```json
{
    "id": "discountSearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search for discounted products by category",
    "title": "Discounts",
    "type": "query",
    "parameters": [
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category to find discounted products",
            "inputType": "text"
        }
    ]
},
```

Ahora vamos a volver al primer comando, **inventorySearch**, que tiene 5 parámetros, lo que permite unas consultas mucho más sofisticadas.

```json
{
    "id": "inventorySearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search products by name, category, inventory status, supplier location, stock level",
    "title": "Product inventory",
    "type": "query",
    "parameters": [
        {
            "name": "productName",
            "title": "Product name",
            "description": "Enter a product name here",
            "inputType": "text"
        },
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category of the product",
            "inputType": "text"
        },
        {
            "name": "inventoryStatus",
            "title": "Inventory status",
            "description": "Enter what status of the product inventory. Possible values are 'in stock', 'low stock', 'on order', or 'out of stock'",
            "inputType": "text"
        },
        {
            "name": "supplierCity",
            "title": "Supplier city",
            "description": "Enter the supplier city of product",
            "inputType": "text"
        },
        {
            "name": "stockQuery",
            "title": "Stock level",
            "description": "Enter a range of integers such as 0-42 or 100- (for >100 items). Only use if you need an exact numeric range.",
            "inputType": "text"
        }
    ]
},
```

Copilot puede rellenarlos en función de las descripciones y la extensión de mensajes devolverá una lista de productos filtrados por todos los parámetros que no estén en blanco.

## Tarea 2: Examen del código del bot

Ahora abre el archivo **src/searchApp.ts**, que contiene el código del bot que usa [Bot Builder SDK](https://learn.microsoft.com/azure/bot-service/index-bf-sdk) para comunicarse con Azure Bot Framework. Observa que el bot extiende una clase SDK **TeamsActivityHandler**.

```typescript
export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  ...
```

### Consulta de extensión de mensajes

La aplicación puede controlar los mensajes (llamados **actividades**) procedentes de Microsoft 365 invalidando los métodos de **TeamsActivityHandler**.

El primero de ellos es una actividad de **consulta de extensión de mensajería**. A esta función se la llama cuando un usuario escribe en una extensión de mensajes o cuando Copilot la llama. El controlador envía la consulta en función del **commandID**. Estos son los mismos commandID que se usan en el manifiesto de la aplicación.

```typescript
  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }
  }
  ...
```

### Acciones de la tarjeta adaptable

El otro tipo de actividades que nuestra aplicación necesita controlar son las acciones de la tarjeta adaptable, como cuando un usuario selecciona **Actualizar existencias** o **Volver a pedir** en una tarjeta adaptable. Dado que no hay ningún método específico para una acción de tarjeta adaptable, el código invalida `onInvokeActivity()`, que es una clase de actividad mucho más amplia que incluye consultas de extensión de mensajes. Por ese motivo, el código comprueba manualmente el nombre de la actividad y la envía al controlador adecuado. Si el nombre de la actividad no es para una acción de tarjeta adaptable, la cláusula `else` ejecuta la implementación base de `onInvokeActivity()`, que, entre otras cosas, llamará a nuestro método `handleTeamsMessagingExtensionQuery()` si la actividad **Invocación** es una consulta.

```typescript
import {
  TeamsActivityHandler,
  TurnContext,
  MessagingExtensionQuery,
  MessagingExtensionResponse,
  InvokeResponse
} from "botbuilder";
import productSearchCommand from "./messageExtensions/productSearchCommand";
import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
import revenueSearchCommand from "./messageExtensions/revenueSearchCommand";
import actionHandler from "./adaptiveCards/cardHandler";

export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }

  }

  // Handle adaptive card actions
  public async onInvokeActivity(context: TurnContext): Promise<InvokeResponse> {
    let runEvents = true;
    // console.log (`🎬 Invoke activity received: ${context.activity.name}`);
    try {
      if(context.activity.name==='adaptiveCard/action'){
        switch (context.activity.value.action.verb) {
          case 'ok': {
            return actionHandler.handleTeamsCardActionUpdateStock(context);
          }
          case 'restock': {
            return actionHandler.handleTeamsCardActionRestock(context);
          }
          case 'cancel': {
            return actionHandler.handleTeamsCardActionCancelRestock(context);
          }
          default:
            runEvents = false;
            return super.onInvokeActivity(context);
        }
      } else {
          runEvents = false;
          return super.onInvokeActivity(context);
      }
    } 
```

## Tarea 3: Examen del código de comando de extensión de mensajes

En un esfuerzo por hacer que el código sea más modular, legible y reutilizable, cada comando de extensión de mensajes se coloca en su propio módulo TypeScript. Consulta **src/messageExtensions/discountSearchCommand.ts** para ver un ejemplo.

Primero, observa que el módulo exporta un `COMMAND_ID` constante, que contiene el mismo **commandID** que se encuentra en el manifiesto de la aplicación y permite que switch (Instrucción) de **searchApp.ts** funcione correctamente.

A continuación, proporciona una función, `handleTeamsMessagingExtensionQuery()` para controlar las consultas entrantes de **productos con descuento por categoría**.

```typescript
async function handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
): Promise<MessagingExtensionResponse> {

    // Seek the parameter by name, don't assume it's in element 0 of the array
    let categoryName = cleanupParam(query.parameters.find((element) => element.name === "categoryName")?.value);
    console.log(`💰 Discount query #${++queryCount}: Discounted products with categoryName=${categoryName}`);

    const products = await getDiscountedProductsByCategory(categoryName);

    console.log(`Found ${products.length} products in the Northwind database`)
    const attachments = [];
    products.forEach((product) => {
        const preview = CardFactory.heroCard(product.ProductName,
            `Avg discount ${product.AverageDiscount}%<br />Supplied by ${product.SupplierName} of ${product.SupplierCity}`,
            [product.ImageUrl]);

        const resultCard = cardHandler.getEditCard(product);
        const attachment = { ...resultCard, preview };
        attachments.push(attachment);
    });
    return {
        composeExtension: {
            type: "result",
            attachmentLayout: "list",
            attachments: attachments,
        },
    };
}
```

Observa que es posible que el índice de la matriz `query.parameters` no se corresponda con la posición del parámetro en el manifiesto. Aunque esto suele ser solo un problema para un comando de varios parámetros, el código seguirá recibiendo el valor en función del nombre del parámetro en lugar de codificar de forma rígida un índice.

Después de limpiar el parámetro (recortarlo y controlar el hecho de que a veces Copilot asume que "**\***" es un carácter comodín que coincide con todo), el código llama a la capa de acceso de datos de Northwind a `getDiscountedProductsByCategory()`.

A continuación, procesa una iteración en los productos y crea dos tarjetas para cada una:

- una tarjeta de _vista previa_, que se implementa como una tarjeta **prominente**. Esto se muestra en los resultados de la búsqueda en la interfaz de usuario y en algunas citas en Copilot.

- una tarjeta de _resultados_, que se implementa como una tarjeta **adaptable** que incluye todos los detalles.

En la siguiente tarea, revisaremos el código de tarjeta adaptable y también el diseñador de tarjetas adaptables.

## Tarea 4: Examen de las tarjetas adaptables y el código relacionado

Las tarjetas adaptables del proyecto se encuentran en la carpeta **src/adaptiveCards/**. Hay 3 tarjetas, cada una implementada como un archivo JSON.

- **editCard.json**: esta es la tarjeta inicial que muestra la extensión de mensajes o una referencia de Copilot.

- **successCard.json**: cuando un usuario realiza una acción, se muestra esta tarjeta para indicar que se ha realizado correctamente. Es principalmente lo mismo que la tarjeta de edición, excepto que incluye un mensaje para el usuario.

- **errorCard.json**: esta tarjeta se muestra si se produce un error en una acción.

Echemos un vistazo a la tarjeta de edición en el **Diseñador de tarjetas adaptables**. Abre el explorador web en [https://adaptivecards.io](https://adaptivecards.io) y selecciona la opción **Diseñador** en la parte superior.

![Captura de pantalla de las propiedades del Diseñador de tarjetas adaptables.](../media/5-01-adaptive-card-designer-01.png)

Observa las expresiones de enlace de datos como `"text": "📦 ${productName}",`. Esto enlaza la propiedad `productName` de los datos al texto de la tarjeta.

Ahora selecciona **Microsoft Teams** como aplicación host 1️⃣. Pega todo el contenido de **editCard.json** en el Editor de carga de tarjeta 2️⃣ y el contenido de **sampleData.json** en el Editor de datos de ejemplo 3️⃣. Los datos de ejemplo son idénticos a un producto tal como se proporciona en el código. Deberías ver la tarjeta como representada, excepto por un pequeño error que surge debido a la incapacidad del diseñador de mostrar uno de los formatos de tarjeta adaptable.

![Captura de pantalla de Copilot, representación de la tarjeta en función de json.](../media/5-01-adaptive-card-designer-02.png)

Cerca de la parte superior de la página, intenta cambiar el **tema** y el **dispositivo emulado** para ver qué aspecto tendría la tarjeta con un tema oscuro o en un dispositivo móvil. Esta es la herramienta que se usó para compilar las tarjetas adaptables para la aplicación de ejemplo.

Ahora, en Visual Studio Code, abre **cardHandler.ts**. Desde cada uno de los comandos de la extensión de mensajes se llama a la función `getEditCard()` para obtener una tarjeta de **resultados**. El código lee el JSON de la tarjeta adaptable ( que se considera una plantilla) y, a continuación, lo enlaza a los datos de producto. El resultado es más JSON; la misma tarjeta que la plantilla, con las expresiones de enlace de datos rellenadas. Por último, el módulo `CardFactory` se usa para convertir el JSON final en un objeto de tarjeta adaptable para la representación.

```typescript
function getEditCard(product: ProductEx): any {

    var template = new ACData.Template(editCard);
    var card = template.expand({
        $root: {
            productName: product.ProductName,
            unitsInStock: product.UnitsInStock,
            productId: product.ProductID,
            categoryId: product.CategoryID,
            imageUrl: product.ImageUrl,
            supplierName: product.SupplierName,
            supplierCity: product.SupplierCity,
            categoryName: product.CategoryName,
            inventoryStatus: product.InventoryStatus,
            unitPrice: product.UnitPrice,
            quantityPerUnit: product.QuantityPerUnit,
            unitsOnOrder: product.UnitsOnOrder,
            reorderLevel: product.ReorderLevel,
            unitSales: product.UnitSales,
            inventoryValue: product.InventoryValue,
            revenue: product.Revenue,
            averageDiscount: product.AverageDiscount
        }
    });
    return CardFactory.adaptiveCard(card);
}
```

Al desplazarte hacia abajo, verás el controlador para cada uno de los botones de acción de la tarjeta. La tarjeta envía datos cuando se hace clic en un botón de acción, específicamente en `data.txtStock`, que es el cuadro de entrada de **Cantidad** de la tarjeta y `data.productId`, que se envía en cada acción de tarjeta para que el código sepa qué producto actualizar.

```typescript
async function handleTeamsCardActionUpdateStock(context: TurnContext) {

    const request = context.activity.value;
    const data = request.action.data;
    console.log(`🎬 Handling update stock action, quantity=${data.txtStock}`);

    if (data.txtStock && data.productId) {

        const product = await getProductEx(data.productId);
        product.UnitsInStock = Number(data.txtStock);
        await updateProduct(product);

        var template = new ACData.Template(successCard);
        var card = template.expand({
            $root: {
                productName: product.ProductName,
                unitsInStock: product.UnitsInStock,
                productId: product.ProductID,
                categoryId: product.CategoryID,
                imageUrl: product.ImageUrl,
                ...
```

Como puedes ver, el código obtiene estos dos valores, actualiza la base de datos y después envía una nueva tarjeta que contiene un mensaje y los datos actualizados.

## Enhorabuena

Has completado el ejercicio 5 y el laboratorio de complementos para extensiones de mensajes de Microsoft 365 Copilot. ¡Muchas gracias por hacer estos laboratorios!

[Ir al resumen del laboratorio...](./7-summary.md)
