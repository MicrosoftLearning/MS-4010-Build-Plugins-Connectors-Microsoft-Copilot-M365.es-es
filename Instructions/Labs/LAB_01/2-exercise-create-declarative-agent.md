---
lab:
  title: 'Ejercicio 1: Creación de un agente declarativo en Visual Studio Code'
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Ejercicio 1: Creación de un agente declarativo

En este ejercicio, crearás un proyecto de agente declarativo a partir de una plantilla, actualizarás el manifiesto, cargarás el agente en Microsoft 365 y lo probarás en Microsoft 365 Copilot. 

El agente declarativo se implementa en una aplicación de Microsoft 365. Crea un paquete de aplicación que contenga:

- app.manifest.json: el archivo de manifiesto de la aplicación describe cómo se configura la aplicación, incluidas sus funcionalidades.
- declarative-agent.json: el manifiesto del agente declarativo describe cómo se configura el agente declarativo.
- color.png y outline.png: un icono de color y esquema usado para representar el agente declarativo en la interfaz de usuario de Microsoft 365 Copilot.

### Duración del ejercicio

- **Tiempo estimado para completarlo**: 15 minutos

## Tarea 1: Habilitación de cargas de aplicaciones personalizadas en el Centro de administración de Teams

Para cargar agentes declarativos en Microsoft 365 a través del kit de herramientas de Teams, deberás habilitar **cargas de aplicaciones personalizadas** en el Centro de administración de Teams.

1. Ve a Teams apps > App setup policies en el Centro de administración de Teams o ve directamente a [App setup policies](https://admin.teams.microsoft.com/policies/app-setup).
1. Selecciona **lobal (Org-wide default)** en la lista de directivas.
1. Activa **Cargar aplicaciones personalizadas**.
1. Selecciona **Guardar** y después **Confirmar** tu elección.

## Tarea 2: Descargar el proyecto de inicio

Empieza descargando el proyecto de ejemplo de GitHub en un explorador web:

1. Ve al repositorio de plantilla [https://github.com/microsoft/learn-declarative-agent-vscode](https://github.com/microsoft/learn-declarative-agent-vscode).
    1. Sigue los pasos para [descargar el código fuente del repositorio](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) en el equipo.
    1. Extrae el contenido del archivo ZIP descargado en tu **carpeta de Documentos**.

El proyecto de inicio contiene un proyecto del Teams Toolkit que incluye un agente declarativo.

1. Abre la carpeta de proyecto en Visual Studio Code.
1. En la carpeta raíz del proyecto, abre el archivo **README.md**. Examina el contenido para obtener más información sobre la estructura del proyecto.

![Captura de pantalla de Visual Studio Code que muestra el archivo léame del proyecto de inicio y la estructura de carpetas en la vista Explorer.](../media/LAB_01/create-complete.png)

## Tarea 3: Examinar el manifiesto del agente declarativo

Vamos a examinar el archivo de manifiesto del agente declarativo:

- Abre el archivo **appPackage/declarativeAgent.json** y examina el contenido:

    ```json
    {
        "$schema": "https://aka.ms/json-schemas/agent/declarative-agent/v1.0/schema.json",
        "version": "v1.0",
        "name": "da-product-support",
        "description": "Declarative agent created with Teams Toolkit",
        "instructions": "$[file('instruction.txt')]"
    }
    ```

El valor de la propiedad **instructions** contiene una referencia a un archivo denominado **instruction.txt**. El Teams Toolkit proporciona la función **$[file(path)]**. El contenido de **instruction.txt** se incluye en el archivo de manifiesto del agente declarativo cuando se aprovisiona en Microsoft 365.

- En la carpeta **appPackage**, abre el archivo **instruction.txt** y revisa el contenido:

    ```md
    You are a declarative agent and were created with Team Toolkit. You should start every response and answer to the user with "Thanks for using Teams Toolkit to create your declarative agent!\n\n" and then answer the questions and help the user.
    ```

## Tarea 4: Actualizar el manifiesto del agente declarativo

Vamos a actualizar las propiedades **name** y **description** para que sean más relevantes para nuestro escenario.

1. En la carpeta **appPackage**, abre el archivo **declarativeAgent.json**.
1. Actualiza el valor de propiedad **name** a **Microsoft 365 Knowledge Expert**.
1. Actualiza el valor de la propiedad **description** a **Microsoft 365 Knowledge Expert que pueda responder a cualquier pregunta que tengas sobre Microsoft 365**.
1. Guarda los cambios

El archivo cargado debe tener el siguiente contenido:

```json
{
    "$schema": "https://aka.ms/json-schemas/agent/declarative-agent/v1.0/schema.json",
    "version": "v1.0",
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]"
}
```

## Tarea 5: Cargar el agente declarativo en Microsoft 365

Luego, carga el agente declarativo en el inquilino de Microsoft 365.

En Visual Studio Code:

1. En la **barra Activity**, abre la extensión **Teams Toolkit**.

    ![Captura de pantalla de Visual Studio Code. El icono del Teams Toolkit está resaltado en la barra de Activity.](../media/LAB_01/teams-toolkit-open.png)

1. Selecciona **Provision** en la sección **Lifecycle**.

    ![Captura de pantalla de Visual Studio Code que muestra la extensión Teams Toolkit. Se resalta la función "Provision" en la sección Lifecycle.](../media/LAB_01/provision.png)

1. En la solicitud, selecciona **Sign In** y sigue las indicaciones para iniciar sesión en el inquilino de Microsoft 365 con el Teams Toolkit. El proceso de aprovisionamiento se inicia automáticamente después de iniciar sesión.

    ![Captura de pantalla de una solicitud de Visual Studio Code que pide al usuario que inicie sesión en Microsoft 365. Se resalta el botón para iniciar sesión.](../media/LAB_01/provision-sign-in.png)

    ![Captura de pantalla de Visual Studio Code que muestra el proceso de aprovisionamiento en curso. Se resalta el mensaje de aprovisionamiento en curso.](../media/LAB_01/provision-in-progress.png)

1. Para continuar, espera a que se complete la carga.

    ![Captura de pantalla de Visual Studio Code en la que se muestra una notificación del sistema que confirma que el proceso de aprovisionamiento se ha completado. Se resalta la notificación del sistema.](../media/LAB_01/provision-complete.png)

Luego, revisa la salida del proceso de aprovisionamiento.

- En la carpeta **appPackage/build**, abre el archivo **declarativeAgent.dev.json**.

Observa que el valor de propiedad **instructions** contiene el contenido del archivo **instruction.txt**. Se incluye el archivo **declarativeAgent.dev.json** en el archivo **appPackage.dev.zip** junto con los archivos **manifest.dev.json**, **color.png** y **outline.png**. Se carga el archivo **appPackage.dev.zip** en Microsoft 365.

> [!IMPORTANT]
> Después de iniciar sesión en la cuenta de Microsoft 365, es posible que veas las siguientes advertencias o mensajes de error en Visual Studio Code. Si acabas de habilitar cargas de aplicaciones personalizadas en Microsoft Teams, la configuración puede tardar algún tiempo en surtir efecto.  Espera unos minutos e inténtalo de nuevo o cierra sesión y vuelve a iniciarla con tu cuenta de Microsoft 365. Se espera el segundo mensaje sobre el acceso a Microsoft 365 Copilot, ya que el inquilino no tiene una licencia completa de Copilot.
> 
> ![Captura de pantalla de las adevertencias de Visual Studio Code.](../media/LAB_01/ttk-login-errors.png)

## Tarea 6: Probar el agente declarativo en Microsoft 365 Copilot Chat

A continuación, vamos a ejecutar el agente declarativo en Microsoft 365 Copilot Chat y validar su funcionalidad.

1. En la **barra Activity**, abre la extensión **Teams Toolkit**.

    ![Captura de pantalla de Visual Studio Code. El icono del Teams Toolkit está resaltado en la barra de Activity.](../media/LAB_01/teams-toolkit-open.png)

1. En la sección **Lifecycle**, selecciona **Publish**. Espera a que se completen las acciones.

1. Abre Microsoft Edge y navega a Microsoft 365 Copilot Chat en [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat).

1. En **Microsoft 365 Copilot Chat**, selecciona el icono de la parte superior derecha para expandir el panel lateral de Copilot. Observa que el panel muestra los chats recientes y los agentes disponibles.

1. En el panel lateral, selecciona **Microsoft 365 Knowledge Expert** para entrar en la experiencia inmersiva y chatear directamente con el agente.

1. Pregunta al agente **¿Qué puedes hacer?** y envía la solicitud.

    ![Captura de pantalla de Microsoft Edge que muestra Microsoft 365 Copilot. El icono para abrir el panel lateral y el agente de soporte técnico del producto en el panel están resaltados.](../media/LAB_01/test-immersive-side-panel.png)

Ir al ejercicio siguiente.