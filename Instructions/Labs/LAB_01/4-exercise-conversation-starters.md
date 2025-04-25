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
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
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
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]",
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "https://learn.microsoft.com/microsoft-365/"
                }
            ]
        }
    ],
  "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
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
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
  ]
}
```

## Tarea 2: Probar el agente declarativo en Microsoft 365 Copilot Chat

A continuación, carga los cambios e inicia una sesión de depuración.

En Visual Studio Code:

1. En la **barra Activity**, abre la extensión **Teams Toolkit**.
1. En la sección **Lifecycle**, selecciona **Provision** y **Publish**.
1. Selecciona el campo **Confirm** para confirmar que desea enviar una actualización al Catálogo de aplicaciones.
1. Espera a que se complete la carga.

Continúa en el explorador web:

1. En **Microsoft 365 Copilot**, selecciona el icono de la parte superior derecha para **expandir el panel lateral de Copilot**.
1. Busca **Soporte técnico del producto** en la lista de agentes y selecciónalo para entrar en la experiencia inmersiva para chatear directamente con el agente. Presta atención a los temas de conversación definidos en la pantalla del manifiesto en la interfaz de usuario.

![Captura de pantalla de Microsoft Edge que muestra el agente declarativo de Microsoft 365 Knowledge Expert en la experiencia inmersiva con los temas de conversación personalizados.](../media/LAB_01/test-conversation-starters.png)

Cierra el explorador para detener la sesión de depuración en Visual Studio Code.