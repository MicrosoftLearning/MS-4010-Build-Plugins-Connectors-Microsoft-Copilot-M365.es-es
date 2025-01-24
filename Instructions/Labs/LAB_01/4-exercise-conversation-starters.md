---
lab:
  title: 'Ejercicio 3: Adición de inicios de conversación al agente declarativo'
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Ejercicio 3: Adición de inicios de conversación al agente declarativo

En este ejercicio, actualizarás el agente declarativo para incluir los inicios de conversación que proporcionan a los usuarios avisos de ejemplo para ayudarles a comprender los tipos de preguntas que pueden formular.

### Duración del ejercicio

- **Tiempo estimado para completarla**: 5 minutos

## Tarea 1: Adición de inicios de conversación

En Visual Studio Code:

1. En la carpeta **appPackage**, abre el archivo **declarativeAgent.json**.
1. Agrega el siguiente fragmento de código al archivo:

   ```json
   "conversation_starters": [
       {
           "title": "Product information",
           "text": "Tell me about Eagle Air"
       },
       {
           "title": "Returns policy",
           "text": "What is the returns policy?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Repair information",
           "text": "Can you provide information on how to get a product repaired?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
   ]
   ```

1. Guarda los cambios.

El archivo **declarativeAgent.json** debe tener este aspecto:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
  "version": "v1.0",
  "name": "Product support",
  "description": "Product support agent that can help answer customer queries about Contoso Electronics products",
  "instructions": "$[file('instruction.txt')]",
  "capabilities": [
    {
      "name": "OneDriveAndSharePoint",
      "items_by_url": [
        {
          "url": "https://{tenant}-my.sharepoint.com/personal/{user}/Documents/Products"
        }
      ]
    }
  ],
  "conversation_starters": [
    {
      "title": "Product information",
      "text": "Tell me about Eagle Air"
    },
    {
      "title": "Returns policy",
      "text": "What is the returns policy?"
    },
    {
      "title": "Product information",
      "text": "Can you provide information on a specific product?"
    },
    {
      "title": "Product troubleshooting",
      "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
    },
    {
      "title": "Repair information",
      "text": "Can you provide information on how to get a product repaired?"
    },
    {
      "title": "Contact support",
      "text": "How can I contact support for help?"
    }
  ]
}
```

## Tarea 2: Prueba del agente declarativo en Microsoft 365 Copilot

A continuación, carga los cambios e inicia una sesión de depuración.

En Visual Studio Code:

1. En la **barra Activity**, abre la extensión **Teams Toolkit**.
1. Selecciona **Provision** en la sección **Lifecycle**.
1. Espera a que se complete la carga.
1. En la **barra Activity**, cambia a la vista **Run and Debug**.
1. Selecciona el botón **Start Debugging** situado junto al elemento desplegable de la configuración o presiona <kbd>F5</kbd>. Se inicia una nueva ventana del explorador y navega a Microsoft 365 Copilot.

Después, prueba el agente declarativo en Microsoft 365 y valida los resultados.

Continúa en el explorador web:

1. En **Microsoft 365 Copilot**, selecciona el icono en la parte superior derecha para expandir el **panel lateral de Copilot**.
1. Busca **Soporte técnico del producto** en la lista de agentes y selecciónalo para entrar en la experiencia inmersiva para chatear directamente con el agente. Presta atención a los temas de conversación definidos en la pantalla del manifiesto en la interfaz de usuario.

![Captura de pantalla de Microsoft Edge que muestra el agente declarativo de soporte técnico del producto en la experiencia inmersiva con los temas de conversación personalizados.](../media/LAB_01/test-conversation-starters.png)

Cierra el explorador para detener la sesión de depuración en Visual Studio Code.