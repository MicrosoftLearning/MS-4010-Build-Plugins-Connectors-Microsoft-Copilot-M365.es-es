---
lab:
  title: 'Ejercicio 1: Descarga del proyecto y examen de archivos'
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Ejercicio 1: Descarga del proyecto y examen de archivos

La extensión de un agente declarativo con acciones permite recuperar y actualizar los datos almacenados en sistemas externos en tiempo real. Mediante complementos de API, puedes conectarte a sistemas externos a través de sus API para recuperar y actualizar información.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Descarga del proyecto de inicio

Empieza descargando el proyecto de ejemplo. En un explorador web:

1. Vaya a [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript).
    1. Sigue los pasos para [descargar el código fuente del repositorio](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) en el equipo.
    1. Extrae el contenido del archivo ZIP descargado en tu **carpeta de Documentos**.
    1. Abre la carpeta en Visual Studio Code.

El proyecto de ejemplo es un proyecto del Teams Toolkit que incluye un agente declarativo y una API anónima que se ejecuta en Azure Functions. El agente declarativo es idéntico al agente declarativo recién creado mediante el Teams Toolkit. La API pertenece a un restaurante italiano ficticio y te permite navegar por el menú del día y hacer pedidos.

## Tarea 2: Examen de la definición de API

Primero examina la definición de API de la API del restaurante italiano.

En Visual Studio Code:

1. En la vista **Explorer**, abre el archivo **appPackage/apiSpecificationFile/ristorante.yml**. El archivo es una especificación de OpenAPI que describe la API del restaurante italiano.
1. Busca la propiedad **servers.url**.

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    Observa que apunta a una dirección URL local que coincide con la dirección URL estándar al ejecutar Azure Functions localmente.

1. Busca la propiedad **paths**, que contiene dos operaciones: **/dishes** para recuperar el menú del día y **/orders** para realizar un pedido.

    > [!IMPORTANT]
    > Ten en cuenta que cada operación contiene la propiedad **operationId** que identifica de forma única la operación en la especificación de API. Copilot requiere que cada operación tenga un Id. exclusivo para saber a qué API debe llamar para las indicaciones de usuario específicas.

## Tarea 3: Examen de la implementación de la API

A continuación, examina la API de ejemplo que usas en este ejercicio.

En Visual Studio Code:

1. En la vista **Explorer**, abre el archivo **src/data.json**. El archivo contiene un elemento de menú ficticio de nuestro restaurante italiano. Cada plato consta de lo siguiente:

    - nombre,
    - descripción,
    - vínculo a una imagen,
    - precio,
    - en qué plato se sirve,
    - tipo (plato o bebida),
    - opcionalmente una lista de alergenos

    En este ejercicio, las API usan este archivo como origen de datos.
1. A continuación, amplía la carpeta **src/functions**. Observa los dos archivos denominados **dishes.ts** y **placeOrder.ts**. Estos archivos contienen la implementación de las dos operaciones definidas en la especificación de API.
1. Abre el archivo **src/functions/dishes.ts**. Dedica un momento a revisar cómo funciona la API. Comienza cargando los datos de ejemplo desde el archivo **src/functions/data.json**.

    ```typescript
    import data from "../data.json";
    ```

    A continuación, busca en los distintos parámetros de cadena de consulta los posibles filtros que puede pasar el cliente que llama a la API.

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    En función de los filtros especificados en la solicitud, la API filtra el conjunto de datos y devuelve una respuesta.

1. Luego, examina la API para realizar pedidos definidos en el archivo **src/functions/placeOrder.ts**. La API comienza por hacer referencia a los datos de ejemplo. Después, define la forma del pedido que envía el cliente en el cuerpo de la solicitud.

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    Cuando la API procesa la solicitud, primero comprueba si la solicitud contiene un cuerpo y si tiene la forma correcta. Si no es así, rechaza la solicitud con un error 400 Solicitud incorrecta.

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    A continuación, la API resuelve la solicitud en platos del menú y calcula el precio total.

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > Observa cómo la API espera que el cliente especifique el plato con una parte del nombre en lugar de su Id. Esto se debe a que los modelos de lenguaje grande funcionan mejor con palabras que con números. Además, antes de llamar a la API para realizar el pedido, Copilot tiene el nombre del plato inmediatamente disponible como parte de la indicación del usuario. Si Copilot tuviera que hacer referencia a un plato por su identificador, primero tendría que recuperarlo, lo que requiere solicitudes de API adicionales que Copilot no puede hacer de momento.

    Cuando la API esté lista, devuelve una respuesta con un precio total y un Id. de pedido inventado y el estado.

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

