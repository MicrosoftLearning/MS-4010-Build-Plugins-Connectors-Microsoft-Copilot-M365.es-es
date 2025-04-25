---
lab:
  title: 'Ejercicio 2: Creación de un agente declarativo e integración del conector de Graph'
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Ejercicio 2: Creación de un agente declarativo e integración del conector de Graph

En este ejercicio, crearás un nuevo agente declarativo desde cero y lo configurarás para que use la conexión externa como origen de fundamentación.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Crear un agente declarativo

Una manera de crear un agente declarativo es mediante el kit de herramientas de Teams. El kit de herramientas de Teams ofrece un proyecto de plantilla para crear agentes declarativos, lo que te da un excelente punto de partida para empezar a configurar los ajustes del agente e incluir funcionalidades adicionales.

Para crear un agente declarativo, abre Visual Studio Code.

En Visual Studio Code:

1. En la **barra Activity**, abre la extensión **Teams Toolkit**.
1. En el panel de Teams Toolkit, selecciona el botón **Create a New App**.
1. En el cuadro de diálogo **New Project**, elige la opción **Agent**.
1. En el cuadro de diálogo siguiente, elige la opción **Declarative Agent**.
1. Para no agregar un complemento, selecciona la opción **No plugin**.
1. Selecciona una carpeta donde quieras almacenar el proyecto en el equipo.
1. Asigne el nombre `da-it-policies` al proyecto.

## Tarea 2: Configurar instrucciones del agente declarativo

Teams Toolkit crea un nuevo proyecto de agente declarativo. Para limitarlo a tu escenario, actualiza la descripción e instrucciones del agente.

En Visual Studio Code:

1. Abre el archivo **appPackage/declarativeAgent.json**
1. Actualiza el valor de la propiedad **name** a `Contoso IT policies`
1. Actualiza el valor de la propiedad **description** a `Assistant specialized in Contoso IT policies`.
1. Guarda los cambios.
1. El contenido del archivo tendrá el siguiente aspecto:

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. Abre el archivo **appPackage/instruction.txt**.
1. Actualiza el contenido a:

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. Guarda los cambios.

## Tarea 3: Integrar el conector de Microsoft Graph con un agente declarativo

Después de crear un agente declarativo, el siguiente paso consiste en integrarlo con un conector de Microsoft Graph para que pueda acceder a datos externos.

En Visual Studio Code:

1. Abre el archivo **appPackage/declarativeAgent.json**.
1. Después de la propiedad **instructions**, agrega una nueva propiedad denominada **capabilities** con el código siguiente:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. En la propiedad **connection_id**, especifica `policieslocal` como el id. de la conexión externa. `policieslocal` es el id. de la conexión externa que creó el conector de Graph en los pasos anteriores.
1. El contenido del archivo tendrá el siguiente aspecto:

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. Guarda los cambios.
