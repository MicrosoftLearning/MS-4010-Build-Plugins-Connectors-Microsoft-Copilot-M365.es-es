---
lab:
  title: 'Ejercicio 1: Ejecución del ejemplo como una extensión de mensajes'
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# Ejercicio 1: Ejecución del ejemplo como una extensión de mensajes

En este ejercicio, ejecutarás la aplicación como complemento para Teams y Outlook. Experimentarás con varios mensajes y observarás cómo se invoca el complemento mediante distintos parámetros.

## Tarea 1: Configuración del proyecto para usarlo por primera vez

En este proyecto, la base de datos de Northwind se almacena en Azure Table Storage; cuando se depura localmente, se usa el emulador de almacenamiento de [Azurite](https://learn.microsoft.com/azure/storage/common/storage-use-azurite?tabs=visual-studio). Esto se basa principalmente en el proyecto, pero el proyecto no se compilará a menos que proporciones la cadena de conexión.

El kit de herramientas de Teams almacena variables de entorno en la carpeta **env** y rellenará todos los valores automáticamente cuando inicies el proyecto la primera vez. Sin embargo, hay un valor específico de la aplicación de ejemplo y es la cadena de conexión para acceder a la base de datos de Northwind. La configuración necesaria se proporciona en el archivo **env/.env.local.user.sample**. 

Realiza una copia de este archivo en la carpeta **env** y denomínalo **.env.local.user**. Aquí es donde se almacenan las configuraciones secretas o confidenciales.

Si no estás seguro de cómo hacerlo en Visual Studio Code: 

1. Expande la carpeta **env** y haz clic con el botón derecho en **.env.local.user.sample**. Selecciona **Copy**. 

1. Después haz clic con el botón derecho en cualquier parte de la carpeta **env** y selecciona **Paste**. Tendrás un nuevo archivo denominado **.env.local.user copy.sample**. 

1. Use el mismo menú contextual para cambiar el nombre del archivo a **.env.local.user** y has terminado.

    ![Captura de pantalla que muestra la copia de .env.local.user.sample en .env.local.user.](../media/2-01-setup-project-01.png)

El archivo **.env.local.user** resultante debe contener esta línea:

```console
SECRET_BOT_PASSWORD=
SECRET_STORAGE_ACCOUNT_CONNECTION_STRING=UseDevelopmentStorage=true
```

## Tarea 2: Ejecución de la aplicación de forma local

1. En Visual Studio Code con la **carpeta de trabajo** abierta, presiona **F5** para iniciar la depuración o selecciona el botón Inicio 1️⃣.

1. Selecciona **Debug in Teams (Edge)** 2️⃣.

    ![Captura de pantalla que muestra la ejecución de la aplicación localmente.](../media/2-02-run-project-01.png)

    La primera vez que ejecutes la aplicación, es posible que se te pida que permitas que NodeJS pase por el firewall; esto es necesario para permitir que la aplicación se comunique.

    Puede tardar un tiempo mientras se cargan todos los paquetes npm. Por último, se abrirá una ventana del explorador e se te invitará a iniciar sesión.

    ![Captura de pantalla que muestra la ventana del explorador abierta con un formulario de inicio de sesión.](../media/2-02-run-project-03.png)

    Una vez que hayas iniciado sesión, Microsoft Teams deberá abrir y mostrar una oferta de diálogo para instalar la aplicación. Observa la información mostrada; que se deriva del **manifiesto de la aplicación**.

1. Selecciona **Add** para agregar Northwind Inventory como aplicación personal.

    ![Captura de pantalla de la pantalla de instalación de la aplicación con un botón Agregar grande.](../media/2-02-run-project-04.png)

> [!NOTE]
> Si ves esta pantalla, debe corregir el archivo **env/.env.local.user**; esto se explicó en la tarea anterior.
>
> ![Captura de pantalla del error que se muestra debido a que falta una variable de entorno.](../media/2-01-setup-project-06.png)

Se te debería dirigir a un chat dentro de la aplicación, pero puedes usar la aplicación en cualquier chat.

## Tarea 3: Prueba en Microsoft Teams

1. En el **chat de Northwind Inventory**: comienza a escribir un mensaje 1️⃣ que haga referencia a un producto. Luego selecciona  **+** 2️⃣ para insertar una tarjeta adaptable para el producto. 

1. En el panel flotante, selecciona la aplicación **Northwind Inventory** que acabas de instalar 3️⃣.

    ![Captura de pantalla que muestra la selección de "+" para abrir el panel de extensión de mensajes.](../media/2-03-test-message-extension-teams-take-2-01.png)

  Verás un cuadro de diálogo de búsqueda con 2 pestañas 1️⃣. La pestaña **Product Inventory** permite buscar productos por nombre.

1. Escribe un nombre de producto o el principio de un nombre de producto como **chai** en el cuadro de búsqueda 2️⃣. Si te detienes mientras escribes las primeras letras, se te darán más opciones de productos que comienzan con los mismos caracteres.

1. Selecciona **Chai** 3️⃣ para insertar una tarjeta adaptable en la conversación junto con tu comentario.

    ![Captura de pantalla que muestra la selección de Chai en los resultados.](../media/2-03-test-message-extension-teams-take-2-02.png)

1. Puedes ver la tarjeta, pero no puedes usarla hasta que la envíes. Realiza las modificaciones finales en el mensaje y selecciona **Send**. Observa que no hay ningún chai en pedido 1️⃣. Debemos tener algunos bebedores de chai frecuentes y que se podrían pasar así que es mejor que pidas más. 

    ![Captura de pantalla que muestra el envío de la tarjeta adaptable.](../media/2-03-test-message-extension-teams-take-2-03.png)

    > [!NOTE]
    > Las acciones de la tarjeta adaptable no funcionarán hasta que envíes la tarjeta. Si recibes un error, comprueba y asegúrate de que has enviado el mensaje y estás trabajando con la tarjeta después de que se haya enviado.

1. Selecciona el botón **Realizar acción** 2️⃣ para abrir una tarjeta secundaria. Escribe una cantidad 3️⃣ y selecciona el botón **Restock** 4️⃣. La tarjeta se actualizará con un mensaje de confirmación y un número actualizado de unidades en pedido.

    ![Captura de pantalla que muestra la actualización de la cantidad de chai en la tarjeta adaptable.](../media/2-03-test-message-extension-teams-take-2-04.png)

Puedes cancelar el pedido o modificar los niveles de existencias con los otros dos botones.

## Tarea 4: Consultas avanzadas

De nuevo en Visual Studio Code, abre el archivo de manifiesto de la aplicación denominado **manifest.json** en el directorio **appPackage**. Observarás que la información de la aplicación que se mostró cuando instalaste la aplicación está aquí. Desplázate hacia abajo y busca `composeExtensions:`. 

```json
"composeExtensions": [
    {
        "botId": "${{BOT_ID}}",
        "commands": [
            {
                "id": "inventorySearch",
                ...
                "description": "Search products by name, category, inventory status, supplier location, stock level",
                "title": "Product inventory",
                "type": "query",
                "parameters": [ ... ]
            },
            {
                "id": "discountSearch",
                ...
                "description": "Search for discounted products by category",
                "title": "Discounts",
                "type": "query",
                "parameters": [ ...]
            }
        ]
    }
],
```

> [!NOTE]
> Las extensiones de redacción son el nombre histórico de la extensión de mensajes; la extensión de mensajes de Northwind Inventory se define aquí.

Primero observa el **id. de bot** aprovisionado por Microsoft Teams, que usa el canal del bot de Azure para intercambiar mensajes seguros y en tiempo real con la aplicación. El kit de herramientas de Teams registrará el bot y rellenará el id. automáticamente.

Luego observa la colección de comandos. Estos corresponden a las pestañas del cuadro de diálogo de búsqueda en Teams. En esta aplicación, los comandos están diseñados principalmente para Copilot más que para los usuarios normales.

Ya has ejecutado el primer comando al buscar un producto por nombre. Para probar los otros comandos, escribe **Beverages**, **Dairy** o **Produce** en la pestaña **Discounts** y verás los productos con descuento en esas categorías. Copilot puede usar la consulta para responder a preguntas sobre productos con descuento.

![Captura de pantalla que muestra la búsqueda de bebidas en la pestaña Discount.](../media/2-03-test-multi-02.png)

Ahora vuelve a examinar el primer comando. Observarás que tiene 5 parámetros.

```json
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
```

Aunque Teams solo puede mostrar el primer parámetro; Copilot puede usar los 5, lo que le permite realizar consultas más avanzadas de los datos de Northwind Inventory. Como solución alternativa a la limitación de la interfaz de usuario de Teams, la pestaña **Northwind Inventory** aceptará hasta 5 parámetros separados por comas con el formato:

```console
name,category,inventoryStatus,supplierCity,supplierName
```

![Captura de pantalla que muestra la entrada de varios campos separados por comas en la pestaña Northwind Inventory.](../media/2-03-test-multi-04.png)

Lee detenidamente las descripciones del archivo JSON anterior al escribir una consulta. Intenta escribir los términos que se enumeran a continuación y, al escribirlos, ten en cuenta la pestaña de la consola de depuración en Visual Studio Code, donde verá cada consulta a medida que se ejecuta.

- **_chai_**: encontrar productos con nombres que comiencen por **chai**.

- **_c,bev_**: buscar productos en categorías a partir de **bev** y nombres que comienzan por **c**.

- **_,,out,_**: encontrar productos que están agotados.

- **_,,in,London_**: encontrar productos que están pedidos a proveedores en Londres.

- **_tofu,produce,,Osaka_**: encontrar productos en la categoría **produce** con proveedores en **Osaka** y nombres que comienzan por **tofu**.

Cada término de consulta filtra la lista de productos hacia abajo. El formato de cada término de consulta es arbitrario; asegúrate de explicárselos a Copilot en la descripción de cada parámetro.

## Tarea 5: Prueba en Microsoft Outlook

Vamos a realizar un breve desvío para que puedas ver cómo funcionan las extensiones de mensajes en Microsoft Outlook.

1. Abre primero el menú de la aplicación de Microsoft 365 1️⃣ y selecciona **Outlook** 2️⃣.

    ![Captura de pantalla que muestra la apertura Outlook para Microsoft 365.](../media/2-04-test-message-extension-outlook-01.png)

1. Selecciona **Nuevo correo** para empezar a redactar un correo electrónico.

    ![Captura de pantalla de la creación de un nuevo correo electrónico en Outlook.](../media/2-04-test-message-extension-outlook-02.png)

1. Agrega un **destinatario** 1️⃣ y un **asunto** 2️⃣ y después coloca el cursor en el cuerpo del mensaje 3️⃣. Incluso puedes escribir algo. Cuando estés listo, elige **Insertar** en la barra de herramientas y selecciona **Aplicaciones** en la barra de herramientas 4️⃣.

    ![Captura de pantalla de la selección del botón Aplicación al redactar un mensaje de Outlook.](../media/2-04-test-message-extension-outlook-03.png)

1. Selecciona la aplicación **Northwind Inventory** y búscala si es necesario para encontrarla.

    ![Captura de pantalla de la selección de la aplicación Northwind Inventory.](../media/2-04-test-message-extension-outlook-04.png)

1. Busca **Chai** 1️⃣ como antes y selecciona el resultado para insertar la tarjeta adaptable 2️⃣.

    ![Captura de pantalla de la entrada de una búsqueda de Chai.](../media/2-04-test-message-extension-outlook-05.png)

    ![Captura de pantalla de la realización de acciones en un mensaje en Outlook.](../media/2-04-test-message-extension-outlook-07-a.png)

> [!NOTE]
> La tarjeta adaptable no funcionará hasta que envíes el mensaje. El destinatario no podrá ver la tarjeta si no usa Microsoft Outlook y no podrá realizar ninguna acción en ella si no tiene instalada la aplicación Northwind Inventory.

## Tarea 6: Visualización de la base de datos de Northwind en el Explorador de Azure Storage

La base de datos de Northwind no es elegante, ¡pero es real! Si deseas ver o incluso modificar los datos:

1. Abre el [Explorador de Azure Storage](https://azure.microsoft.com/products/storage/storage-explorer/) mientras se ejecuta Azurite (la ejecución de la aplicación inicia Azurite automáticamente).

1. Abre **Emulator & Attached**, **Storage Accounts**, **Emulator - Default Ports** y **Tables** para ver los datos de Northwind.

    ![Captura de pantalla del Explorador de Azure Storage que muestra las tablas de la base de datos de Northwind.](../media/2-06-azure-storage-explorer-01.png)

El código lee la tabla **Products** en cada consulta, pero solo se accede a las demás tablas cuando se inicia la aplicación. Por lo tanto, si deseas agregar una nueva categoría, deberás reiniciar la aplicación para que aparezca.

## Comprobar el trabajo

Después de seguir todas las tareas de este ejercicio, debes tener una aplicación de extensión de mensajes operativa que se pueda usar como complemento de Microsoft 365 para Teams o Outlook.

Cuando todo funcione, estarás listo para ejecutar la aplicación de ejemplo en **Microsoft 365 Copilot**. 

[Ir el ejercicio siguiente...](./4-exercise-2-run-copilot-plugin.md)
