---
lab:
  title: 'Ejercicio 2: Actualización de la definición del complemento de API'
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Ejercicio 2: Actualización de la definición del complemento de API

El siguiente paso es actualizar la definición del complemento de API con tarjetas adaptables que Copilot debe usar para mostrar datos de la API a los usuarios.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Agregar tarjeta adaptable para mostrar un plato

En Visual Studio Code:

1. Abre el archivo **cards/dish.json** y copia su contenido.
1. Abre el archivo **appPackage/ai-plugin.json**.
1. En la propiedad **functions.getDishes.capabilities.response_semantics**, agrega una nueva propiedad denominada **static_template** y establece el valor **cuerpo** al contenido de **dish.json**.
1. El fragmento de código completo es similar al siguiente:

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "Image",
                "url": "${image_url}",
                "size": "large"
              },
              {
                "type": "TextBlock",
                "text": "${name}",
                "weight": "Bolder"
              },
              {
                "type": "TextBlock",
                "text": "${description}",
                "wrap": true
              },
              {
                "type": "TextBlock",
                "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
                "weight": "Lighter"
              },
              {
                "type": "TextBlock",
                "text": "**Price:** €${formatNumber(price, 2)}",
                "weight": "Lighter",
                "spacing": "None"
              }
            ]
          }
      ]
    }
    ```

1. Guarda los cambios.

## Tarea 2: Agregar plantilla Tarjeta adaptable para mostrar el resumen del pedido

En Visual Studio Code:

1. Abre el archivo **cards/order.json** y copia su contenido.
1. Abre el archivo **appPackage/ai-plugin.json**.
1. En la propiedad **functions.placeOrder.capabilities.response_semantics**, agrega una nueva propiedad denominada **static_template** y establece su contenido en la tarjeta adaptable.
1. El archivo completo es similar a este:

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "TextBlock",
            "text": "Order Confirmation 🤌",
            "size": "Large",
            "weight": "Bolder",
            "horizontalAlignment": "Center"
          },
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "Your order has been successfully placed!",
                "weight": "Bolder",
                "spacing": "Small"
              },
              {
                "type": "FactSet",
                "facts": [
                  {
                    "title": "Order ID:",
                    "value": "${order_id} "
                  },
                  {
                    "title": "Status:",
                    "value": "${status}"
                  },
                  {
                    "title": "Total Price:",
                    "value": "€${formatNumber(total_price, 2)}"
                  }
                ]
              }
            ]
          }
        ]
    }
    ```

1. Guarde los cambios.
