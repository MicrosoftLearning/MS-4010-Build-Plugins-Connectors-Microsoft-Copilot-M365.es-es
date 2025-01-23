---
lab:
  title: 'Ejercicio 3: Conexión de la definición del complemento al agente declarativo'
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Ejercicio 3: Conexión de la definición del complemento al agente declarativo

Después de completar la compilación de la definición del complemento de API, el siguiente paso es registrarla con el agente declarativo. Cuando los usuarios interactúan con el agente declarativo, la indicación del usuario coincide con los complementos de API definidos e invoca las funciones relevantes.

### Duración del ejercicio

- **Tiempo estimado para completarla**: 5 minutos

## Tarea 1: Conectar la definición del complemento al agente declarativo

En Visual Studio Code:

1. Abre el archivo **appPackage/declarativeAgent.json**.
1. Después de la propiedad **instrucciones**, agrega el siguiente fragmento de código:

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    Con este fragmento de código, se conecta el agente declarativo al complemento de API. Especifica un identificador único para el complemento e indica al agente dónde puede encontrar la definición del complemento.

1. El archivo completo es similar a este:

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. Guarda los cambios.

## Tarea 2: Actualizar la información y la instrucción del agente declarativo

El agente declarativo que vas a crear en este ejercicio ayuda a los usuarios a examinar el menú del restaurante italiano local y hacer pedidos. Para optimizar el agente para este escenario, actualiza su nombre, descripción e instrucciones.

En Visual Studio Code:

1. Actualiza la información del agente declarativo:
    1. Abre el archivo **appPackage/declarativeAgent.json**.
    1. Actualiza el valor de la propiedad **nombre** a **Il Ristorante**.
    1. Actualiza el valor de la propiedad **descripción** a **Pida los platos y bebidas italianos más apetitosos desde la comodidad de su escritorio.**.
    1. Guarde los cambios.
1. Actualiza las instrucciones del agente declarativo:
    1. Abre el archivo **appPackage/instruction.txt**.
    1. Reemplace su contenido por lo siguiente:

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        Ten en cuenta que en las instrucciones definimos el comportamiento general del agente y le indicamos de qué es capaz. También se incluyen instrucciones para un comportamiento específico en torno a la realización de un pedido, incluyendo la forma de los datos que espera la API. Se incluye esta información para asegurar que el agente funciona según lo previsto.

    > [!NOTE]
    > Para conservar el formato, es posible que tengas que realizar varias operaciones de copiar y pegar en el Bloc de notas antes de copiar en Visual Studio Code.

    1. Guarde los cambios.
1. Para ayudar a los usuarios a entender para qué pueden usar el agente, agrega inicios de conversación:
    1. Abre el archivo **appPackage/declarativeAgent.json**.
    1. Después de la propiedad **instrucciones**, agrega una nueva propiedad denominada **inicios_conversación**:

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. El archivo completo es similar a este:

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. Guarda los cambios.

## Tarea 3: Actualizar la dirección URL de la API

Para poder probar el agente declarativo, debed actualizar la dirección URL de la API en el archivo de especificación de API. Ahora mismo, la dirección URL se establece en `http://localhost:7071/api`, que es la dirección URL que Azure Functions usa cuando se ejecuta localmente. Sin embargo, dado que quieres que Copilot llame a la API desde la nube, debes exponer la API a Internet. El kit de herramientas de Teams expone automáticamente tu API local a través de Internet mediante la creación de un túnel de desarrollo. Cada vez que empieces a depurar tu proyecto, el kit de herramientas de Teams inicia un nuevo túnel de desarrollo y almacena su dirección URL en la variable **OPENAPI_SERVER_URL**. Puedes ver cómo el kit de herramientas de Teams inicia el túnel y almacena su dirección URL en el archivo **.vscode/tasks.json**, en la tarea **Iniciar túnel local**:

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

Para usar este túnel, debes actualizar la especificación de tu API para usar la variable **OPENAPI_SERVER_URL**.

En Visual Studio Code:

1. Abre el archivo **appPackage/apiSpecificationFile/ristorante.yml**.
1. Cambia el valor de la propiedad **servers.url** a **${{OPENAPI_SERVER_URL}}/api**.
1. El archivo cambiado tiene el siguiente aspecto:

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. Guarda los cambios.

Tu complemento de API está terminado e integrado con un agente declarativo. Continúa con la prueba del agente en Microsoft 365 Copilot.