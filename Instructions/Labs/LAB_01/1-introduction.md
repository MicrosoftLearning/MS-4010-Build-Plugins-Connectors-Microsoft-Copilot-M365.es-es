---
lab:
  title: Introducción
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Introducción

Las extensiones de mensajes permiten a los usuarios trabajar con sistemas externos de Microsoft Teams y Microsoft Outlook. Los usuarios pueden usar las extensiones de mensajes para buscar y cambiar datos, y compartir la información de estos sistemas en mensajes y correos electrónicos como una tarjeta con formato enriquecido.

Supongamos que tienes una lista de SharePoint Online con información de productos actual y relevante para tu organización. Deseas buscar y compartir esta información en Microsoft 365. También deseas que Copilot para Microsoft 365 use esta información en sus respuestas.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="Captura de pantalla de la página principal del sitio de equipo de SharePoint Online de soporte de productos. Se muestra una lista de productos lanzados recientemente." lightbox="../media/1-sharepoint-online-product-support-site.png":::

En este módulo, crearás una extensión de mensajes. La extensión de mensajes usa un bot para comunicarse con Microsoft Teams, Microsoft Outlook y Copilot para Microsoft 365.

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams." lightbox="../media/2-search-results-nuget.png":::

Usa Microsoft Entra para autenticar a los usuarios, lo que le permite devolver datos de SharePoint Online mediante Microsoft Graph API en su nombre.

:::image type="content" source="../media/3-sign-in.png" alt-text="Captura de pantalla de un desafío de autenticación en una extensión de mensajes basada en búsqueda. Se muestra un vínculo al inicio de sesión." lightbox="../media/3-sign-in.png":::

Una vez autenticado el usuario, la extensión de mensajes obtendrá información del producto en SharePoint Online mediante Microsoft Graph API. Devuelve los resultados de la búsqueda que se pueden incrustar en mensajes y correos electrónicos como una tarjeta con formato enriquecido para compartirlos después.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Captura de pantalla de los resultados de la búsqueda devueltos por una extensión de mensajes basada en búsqueda en Microsoft Teams. Los resultados de la búsqueda se devuelven desde SharePoint Online. Cada resultado de búsqueda muestra el nombre, la categoría y la imagen del producto." lightbox="../media/4-search-results-sharepoint-online.png":::

:::image type="content" source="../media/5-adaptive-card.png" alt-text="Captura de pantalla del resultado de búsqueda incrustado en un mensaje en Microsoft Teams. Los resultados de la búsqueda se representan como una tarjeta adaptable con el nombre del producto, la categoría, el volumen de llamadas y la fecha de lanzamiento. Se muestra un botón de acción con el título Vista que los usuarios pueden usar para navegar al elemento de lista de productos en SharePoint Online." lightbox="../media/5-adaptive-card.png":::

Funciona con Copilot para Microsoft 365 como complemento, lo que le permite consultar la lista de SharePoint Online en nombre del usuario y usar los datos devueltos en sus respuestas.

:::image type="content" source="../media/6-copilot-answer.png" alt-text="Captura de pantalla de una respuesta en Copilot para Microsoft 365 que contiene la información devuelta por el complemento de extensión de mensajes. Se muestra una tarjeta adaptable que muestra información del producto." lightbox="../media/6-copilot-answer.png":::

Al final de este módulo, podrás crear extensiones de mensajes escritas en C# (que se ejecutan en .NET). Se puede usar en Microsoft Teams, Microsoft Outlook y Copilot para Microsoft 365. Puede consultar datos detrás de las API protegidas y devolver los resultados como tarjetas con formato enriquecido.

## Requisitos previos

- Conocimientos básicos de C#
- Conocimientos básicos de Bicep
- Conocimientos básicos de la autenticación
- Acceso de administrador a un inquilino de Microsoft 365
- Acceso a una suscripción de Azure
- El acceso a Copilot para Microsoft 365 es opcional y solo se requiere para completar un ejercicio
- Visual Studio 2022 17.9 con el [kit de herramientas de Teams](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (componente de herramientas de desarrollo de Microsoft Teams) instalado
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Cuando estés listo para comenzar, selecciona [ir al ejercicio siguiente...](./2-exercise-create-a-message-extension.md).
