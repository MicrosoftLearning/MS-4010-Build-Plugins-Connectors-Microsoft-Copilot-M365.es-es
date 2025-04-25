---
lab:
  title: 'Ejercicio 2: Prueba del agente declarativo en Microsoft 365 Copilot Chat'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Ejercicio 2: Prueba del agente declarativo en Microsoft 365 Copilot Chat

En este ejercicio, probarás e implementarás el agente declarativo en Microsoft 365 y lo probarás con Microsoft 365 Copilot Chat.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 10 minutos

## Tarea 1: Probar el agente declarativo con el complemento de API en Microsoft 365 Copilot Chat

El último paso es probar el agente declarativo con el complemento de API en Microsoft 365 Copilot.

En Visual Studio Code:

1. En la barra Activity, activa la extensión **Teams Toolkit**.
1. En el panel de extensión de **Teams Toolkit**, en la sección **Accounts**, asegúrate de que has iniciado sesión en tu inquilino de Microsoft 365.

  ![Captura de pantalla del kit de herramientas de Teams en la que se muestra el estado de la conexión a Microsoft 365.](../media/LAB_05/3-teams-toolkit-account.png)

1. En la barra Activity, cambia a la vista Run and Debug.
1. En la lista de configuraciones, elige **Debug in Copilot (Edge)** y presiona el botón de reproducir para iniciar la depuración.

  ![Captura de pantalla de la ventana de depuración en Visual Studio Code.](../media/LAB_05/3-vs-code-debug.png)

  Visual Studio Code abre un nuevo explorador web con Microsoft 365 Copilot Chat. Si se te solicita, inicia sesión en tu cuenta de Microsoft 365.

En el explorador web:

1. En el panel lateral, selecciona el agente **da-repairs-keylocal**.

  ![Captura de pantalla del agente personalizado que se muestra en Microsoft 365 Copilot.](../media/LAB_05/3-copilot-agent-sidebar.png)

1. En el cuadro de texto de solicitud, escribe `What repairs are assigned to Karin?` y envía la solicitud.
1. Confirma que deseas enviar datos al complemento de API mediante el botón **Always allow**.

  ![Captura de pantalla de la solicitud para permitir el envío de datos a la API.](../media/LAB_05/3-allow-data.png)

1. Espera a que el agente responda.

  ![Captura de pantalla de la respuesta del agente personalizado a la solicitud del usuario.](../media/LAB_05/3-copilot-response.png)

Detén la sesión de depuración en Visual Studio Code cuando hayas terminado de realizar las pruebas.