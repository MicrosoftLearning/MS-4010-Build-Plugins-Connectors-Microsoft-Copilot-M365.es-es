---
lab:
  title: 'Ejercicio 3: Prueba y depuración'
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Ejercicio 3: Prueba y depuración

En este ejercicio, probarás e implementarás el agente declarativo en Microsoft 365 y lo probarás con Microsoft 365 Copilot Chat.

### Duración del ejercicio

- **Tiempo estimado para completarla**: 5 minutos

## Tarea 1: Probar el agente declarativo en Microsoft 365 Copilot

Para probar el agente declarativo, impleméntalo como una aplicación en el inquilino de Microsoft 365. Después de abrirlo en Microsoft 365 Copilot, comprueba que funciona según lo previsto.

En Visual Studio Code:

1. En la **barra Activity**, abre la extensión **Teams Toolkit**.
1. En el panel **Lifecycle**, elige **Provision**. El kit de herramientas de Teams empaqueta el proyecto de agente declarativo como una aplicación y lo carga en Microsoft 365.
1. Abra un explorador web.

En un explorador web:

1. Vaya a [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat).
1. Inicia sesión con la cuenta profesional que pertenece tu inquilino de Microsoft 365.
1. En Microsoft 365 Copilot, en el panel lateral, selecciona el agente de **directivas de TI de Contoso** para activarlo.
1. En el cuadro de texto de chat, pregunta `What's the acceptable use policy at Contoso?`.
1. Espera a que el agente responda. Observa cómo la respuesta incluye referencias al contenido externo que ingirió el conector de Graph. La dirección URL de cada referencia apunta a la ubicación del sistema externo donde se almacena el contenido.

    ![Captura de pantalla de Microsoft 365 Copilot que responde a la indicación de un usuario.](../media/LAB_04/3-copilot-response.png)