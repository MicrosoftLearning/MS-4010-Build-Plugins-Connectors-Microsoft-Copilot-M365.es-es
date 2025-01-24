---
lab:
  title: 'Ejercicio 2: Definición del complemento de API de compilación'
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Ejercicio 2: Definición del complemento de API de compilación

El siguiente paso es agregar la definición del complemento al proyecto. La definición del complemento contiene la siguiente información:

- Qué acciones puede realizar el complemento.
- Cuál es el formato de los datos que espera y devuelve.
- Cómo debe llamar el agente declarativo a la API subyacente.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Adición de la estructura básica de definición del complemento

En Visual Studio Code:

1. En la carpeta **appPackage**, agrega un nuevo archivo denominado **ai-plugin.json**.
1. Pega el contenido siguiente:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    El archivo contiene una estructura básica para un complemento de API con una descripción para el usuario y el modelo. La descripción **description_for_model** incluye información detallada sobre lo que el complemento puede hacer para ayudar al agente a comprender cuándo debe considerar la posibilidad de invocarlo.
1. Guarda los cambios.

## Tarea 2: Definición de funciones

El complemento de API define una o varias funciones que se asignan a las operaciones de API definidas en la especificación de API. Cada función consta de un nombre y una descripción, así como una definición de respuesta que indica al agente cómo mostrar los datos a los usuarios.

### Definición de una función para recuperar el menú

Comienza definiendo una función para recuperar la información sobre el menú del día.

En Visual Studio Code:

1. Abre el archivo **appPackage/ai-plugin.json**.
1. En la matriz **functions**, agrega el siguiente fragmento de código:

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    Empieza definiendo una función que invoque la operación **getDishes** de la especificación de API. Luego, proporciona una descripción de la función. Esta descripción es importante porque Copilot la usará para decidir qué función invocar para la indicación de un usuario.

    En la propiedad response_semantics, especifica cómo Copilot debe mostrar los datos que recibe de la API. Dado que la API devuelve la información sobre los platos del menú de la propiedad **dishes**, establece la propiedad **data_path** en la expresión `$.dishes` JSONPath.

    Luego, en la sección **properties**, asigna las propiedades de la respuesta de API que representan el título, la descripción y la dirección URL. Ya que en este caso los platos no tienen una dirección URL, solo asignas el **título** y la **descripción**.

1. El fragmento de código completo es similar al siguiente:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Guarda los cambios.

### Definición de una función para realizar el pedido

A continuación, define una función para realizar el pedido.

En Visual Studio Code:

1. Abre el archivo **appPackage/ai-plugin.json**.
1. Al final de la matriz **functions**, agrega el fragmento de código siguiente:

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    Comienza consultando la operación de API con Id. **placeOrder**. Luego, proporciona la descripción que Copilot usará para hacer coincidir esta función con la indicación de un usuario. Después, indica a Copilot cómo devolver los datos. Estos son los datos que devolverá la API después de realizar un pedido:

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    Como los datos que deseas mostrar se encuentran directamente en la raíz del objeto de respuesta, establece **data_path** en **$** lo que indica el nodo superior del objeto JSON. El título se define para mostrar el número del pedido y el subtítulo es el precio.

1. El archivo completo es similar a este:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Guarda los cambios.

## Tarea 3: Definir entornos de ejecución

Después de definir las funciones para que Copilot las invoque, el siguiente paso es indicarle cómo las debe llamar. Esto lo haces en la sección **runtimes** de la definición del complemento.

En Visual Studio Code:

1. Abre el archivo **appPackage/ai-plugin.json**.
1. En la matriz runtimes, agregue el código siguiente:

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    Empieza indicando a Copilot que le has proporcionado información de OpenAPI sobre la API (**type: OpenApi**) para llamar y que sea anónimo (**auth.type: None**). Luego, en la sección **spec**, especifica la ruta de acceso relativa a la especificación de API que se encuentra en el proyecto. Por último, en la propiedad **run_for_functions**, enumeras todas las funciones que pertenecen a esta API.

1. El archivo completo es similar a este:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Guarda los cambios.

