---
lab:
  title: 'Ejercicio 4: Prueba del agente declarativo en Microsoft 365 Copilot Chat'
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Ejercicio 34: Prueba del agente declarativo en Microsoft 365 Copilot

En este ejercicio, probarás e implementarás el agente declarativo en Microsoft 365 y lo probarás con Microsoft 365 Copilot Chat.

### Duración del ejercicio

- **Tiempo estimado para completarla**: 5 minutos

## Tarea 1: Probar el agente declarativo con el complemento de API en Microsoft 365 Copilot

El último paso es probar el agente declarativo con el complemento de API en Microsoft 365 Copilot.

En Visual Studio Code:

1. En la barra Activity, activa la extensión **Teams Toolkit**.
1. En el panel de extensión de **Teams Toolkit**, en la sección **Accounts**, asegúrate de que has iniciado sesión en tu inquilino de Microsoft 365.

    ![Captura de pantalla del kit de herramientas de Teams en la que se muestra el estado de la conexión a Microsoft 365.](../media/LAB_05/3-teams-toolkit-account.png)

1. En la barra Activity, cambia a la vista **Run and Debug**.
1. En la lista de configuraciones, elige **Debug in Copilot (Edge)** y presiona el botón de reproducir para iniciar la depuración.

    ![Captura de pantalla de la ventana de depuración en Visual Studio Code.](../media/LAB_05/3-vs-code-debug.png)

    Visual Studio Code abre un nuevo explorador web con Microsoft 365 Copilot. Si se te solicita, inicia sesión en tu cuenta de Microsoft 365.

En el explorador web:

1. En el panel lateral, selecciona el agente **da-repairs-oauthlocal**.

    ![Captura de pantalla del agente personalizado que se muestra en Microsoft 365 Copilot.](../media/LAB_05/5-copilot-agent-sidebar.png)

1. Escribe `Show repair records assigned to Karin Blair` en el cuadro de texto de indicación y envía la indicación.

    > [!TIP]
    > En lugar de escribir la indicación, puedes seleccionarla en los temas de conversación.

    ![Captura de pantalla de una conversación iniciada en el agente declarativo personalizado.](../media/LAB_05/5-conversation-starter.png)

1. Confirma que deseas enviar datos al complemento de API mediante el botón **Always allow**.

    ![Captura de pantalla de la solicitud para permitir el envío de datos a la API.](../media/LAB_05/5-allow-data.png)

1. Cuando se te solicite, inicia sesión en la API para seguir usando la misma cuenta que usas para iniciar sesión en el inquilino de Microsoft 365; para ello, selecciona **Sign in to da-repairs-oauthlocal**.

    ![Captura de pantalla de la solicitud para iniciar sesión en la aplicación que protege la API.](../media/LAB_05/5-sign-in.png)

1. Espera a que el agente responda.

    ![Captura de pantalla de la respuesta del agente declarativo a la indicación del usuario.](../media/LAB_05/5-agent-response.png)

Aunque la API sea accesible de forma anónima porque se ejecuta en el equipo local, Microsoft 365 Copilot llama a la API autenticada como se especifica en la especificación de API. Puedes comprobar que la solicitud contiene un token de acceso estableciendo un punto de interrupción en la función de **reparaciones** y enviando otra indicación al agente declarativo. Cuando el código alcance el punto de interrupción, expande la colección req.headers y busca el encabezado de autorización que contiene un JSON Web Token (JWT).

![Captura de pantalla de Visual Studio Code con un punto de interrupción y el panel de depuración que muestra el encabezado de autorización en la solicitud entrante.](../media/LAB_05/5-vs-code-breakpoint-jwt.png)

Detén la sesión de depuración en Visual Studio Code cuando hayas terminado de realizar las pruebas.
