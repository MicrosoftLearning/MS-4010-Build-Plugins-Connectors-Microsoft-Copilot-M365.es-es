---
lab:
  title: 'Ejercicio 1: Creación de una extensión de mensajes'
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Ejercicio 1: Creación de una extensión de mensajes

En este ejercicio, crearás una solución de extensión de mensajes. El kit de herramientas de Teams se usa en Visual Studio para crear los recursos necesarios y, a continuación, iniciar una sesión de depuración y probarla en Microsoft Teams.

![Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams.](../media/1-search-results.png)

### Duración del ejercicio

  - **Tiempo estimado para completarla:** 25 minutos

## Tarea 1: Creación de un nuevo proyecto con el kit de herramientas de Teams para Visual Studio

Empieza por crear un nuevo proyecto de aplicación de Microsoft Teams configurado con una extensión de mensajes que contenga un comando de búsqueda. Aunque puedes crear un proyecto con una plantilla de proyecto del kit de herramientas de Teams para visual Studio, es necesario realizar cambios en el proyecto con scaffolding para poder completar este módulo. En su lugar, usa una plantilla de proyecto personalizada que está disponible como un paquete NuGet. La ventaja de usar una plantilla personalizada es que crea una solución con los archivos y dependencias necesarios, lo que te ahorrará tiempo.

1. Abre una sesión de PowerShell como administrador.

1. Cambia a un directorio de trabajo adecuado mediante la ejecución de:

    ```Powershell
    cd ~\Documents
    ```

1. Comienza con la instalación del paquete de plantilla desde NuGet mediante la ejecución de:

    ```PowerShell
    dotnet new install M365Advocacy.Teams.Templates
    ```

1. Crea un proyecto mediante la ejecución de:

    ```PowerShell
    dotnet new teams-msgext-search --name "ProductsPlugin" `
      --internal-name "msgext-products" `
      --display-name "Contoso products" `
      --short-description "Product look up tool." `
      --full-description "Get real-time product information and share them in a conversation." `
      --command-id "Search" `
      --command-description "Find products by name" `
      --command-title "Products" `
      --parameter-name "ProductName" `
      --parameter-title "Product name" `
      --parameter-description "The name of the product as a keyword" `
      --allow-scripts Yes
    ```

1. Espera a que se cree el proyecto.

1. Ejecuta `cd ProductsPlugin` para cambiar al directorio de proyecto.

1. Ejecuta `.\ProductsPlugin.sln` para abrir la solución en Visual Studio.

1. Selecciona **Visual Studio 2022** en la ventana de selección de la aplicación y, después, selecciona **Siempre**.

## Tarea 2: Creación de un túnel de desarrollo

Cuando el usuario interactúa con la extensión de mensajes, el servicio de bot envía solicitudes al servicio web. Durante el desarrollo, el servicio web se ejecuta localmente en el equipo. Para permitir que el servicio de bot llegue al servicio web, debes exponerlo más allá del equipo con un túnel de desarrollo.

![Captura de pantalla de la ventana de túneles dev en Visual Studio.](../media/14-select-dev-tunnel.png)

Continúa en Visual Studio:

1. En la barra de herramientas, selecciona la lista desplegable situada junto al botón **Iniciar**, expande el menú **Túneles dev (sin túnel activo)** y selecciona **Crear un túnel**.

1. En el cuadro de diálogo especifica los valores siguientes:

    1. **Cuenta**: inicia sesión con la cuenta de Microsoft 365 proporcionada. Selecciona Cuenta **profesional o educativa**.

    1. **Nombre**: msgext-products

    1. **Tipo de túnel**: temporal

    1. **Acceso**: público

1. Para crear el túnel, selecciona **Aceptar.**. Se muestra un mensaje que indica que el nuevo túnel es ahora el túnel activo actual

1. Cierra el mensaje seleccionado **Aceptar**.

## Tarea 3: Preparación de recursos

Ahora que está todo listo, usa el kit de herramientas de Teams para ejecutar el proceso **Preparar dependencias de la aplicación de Teams** para crear los recursos necesarios.

![Captura de pantalla que muestra el menú expandido del kit de herramientas de Teams en Visual Studio Code.](../media/15-prepare-teams-app-dependencies.png)

El proceso de preparar dependencias de la aplicación de Teams actualiza las variables de entorno **BOT_ENDPOINT** y **BOT_DOMAIN** en el archivo **TeamsApp\\env\\.env.local** mediante la dirección URL activa del túnel dev y ejecuta las acciones descritas en el archivo **TeamsApp\\teamsapp.local.yml**.

Dedica un momento a explorar los pasos del archivo **teamsapp.local.yml**.

Continúa en Visual Studio:

1. Abre el menú **Proyecto** (como alternativa, puedes seleccionar el proyecto **TeamsApp** en el Explorador de soluciones), expande el menú **Kit de herramientas de Teams** y selecciona **Preparar dependencias de la aplicación de Teams**.

1. En el cuadro de diálogo **Cuenta de Microsoft 365**, selecciona o inicia sesión en una cuenta existente para acceder al inquilino de Microsoft 365 y, después, selecciona **Continuar**.

1. En el cuadro de diálogo **Aprovisionar**, selecciona la cuenta existente que usarás para implementar recursos en Azure y especifica los valores siguientes:

      1. **Nombre de suscripción**: selecciona la suscripción que deseas usar en el elemento desplegable.

      1. **Grupo de recursos**: selecciona el grupo de recursos rellenado en la lista desplegable.

1. Selecciona **Aprovisionar** para crear los recursos en Azure.

1. En el mensaje de advertencia del kit de herramientas de Teams, selecciona **Aprovisionar**.

1. En el mensaje de información del kit de herramientas de Teams, selecciona **View provisioned resources** para abrir una nueva ventana del explorador.

Dedica un momento a explorar los recursos creados en Azure y mira también las variables de entorno creadas en el archivo **.env.local**.

> [!NOTE]
> Al cerrar y volver a abrir Visual Studio, la dirección URL del túnel dev cambiará y ya no estará seleccionado como túnel activo. Si esto sucede, tendrás que volver a seleccionar el túnel y ejecutar el proceso **Preparar dependencias de la aplicación de Teams** para reflejar la dirección URL actualizada en el manifiesto de la aplicación.

## Tarea 4: Ejecución y depuración

El kit de herramientas de Teams usa perfiles de inicio de varios proyectos. Para ejecutar el proyecto, debes habilitar una característica en vista previa (GB) en Visual Studio.

En Visual Studio:

1. Abre el menú **Herramientas**, selecciona **Opciones...**

1. En el cuadro de búsqueda, escribe **multi-project**.

1. En **Entorno**, selecciona **Características en vista previa**.

1. Activa la casilla situada junto a **Habilitar perfiles de inicio de varios proyectos** y selecciona **Aceptar** para guardar los cambios.

Para iniciar una sesión de depuración e instalar la aplicación en Microsoft Teams:

1. Presiona <kbd>F5</kbd> o selecciona **Iniciar** en la barra de herramientas.

1. Confía o aprueba las advertencias de certificación SSL que aparezcan al iniciar la aplicación por primera vez.

1. Espera hasta que se abra una ventana del explorador y aparezca el cuadro de diálogo de instalación de la aplicación en el cliente web de Microsoft Teams. Si se te solicita, escribe las credenciales de tu cuenta de Microsoft 365.

1. En el cuadro de diálogo de instalación de la aplicación, selecciona **Agregar**.

Para probar la extensión de mensajes:

1. Abre un nuevo chat (<kbd>Alt+N</kbd>) y empieza a escribir **Contoso** en el cuadro **Para**, seleccionando **Contoso Product Support**.

    > [!NOTE]
    > No funcionará si chateas con tu propia cuenta de usuario. Debe ser otro usuario o grupo.

1. En el área de redacción de mensajes, escribe **/apps** para abrir el selector de aplicaciones.

1. En la lista de aplicaciones, selecciona **Contoso products** para abrir la extensión de mensajes

1. En el cuadro de texto, escribe **hello**. Es posible que tengas que escribir la búsqueda varias veces.

1. Espera a que aparezcan los resultados de la búsqueda.

1. En la lista de resultados, selecciona **hello** para insertar una tarjeta en el cuadro de redacción del mensaje

![Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams.](../media/1-search-results.png)

Vuelve a Visual Studio y selecciona **Detener** en la barra de herramientas o presiona <kbd>Mayús</kbd> + <kbd>F5</kbd> para detener la sesión de depuración.

[Ir al ejercicio siguiente...](./3-exercise-add-single-sign-on.md)