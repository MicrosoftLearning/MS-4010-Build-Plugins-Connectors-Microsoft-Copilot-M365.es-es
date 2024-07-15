---
lab:
  title: Introducción
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Introducción

Supongamos que tienes un sistema externo en el que almacenas artículos de knowledge base. Estos artículos contienen información sobre los distintos procesos de tu organización. Deseas poder encontrar y detectar fácilmente información relevante de Microsoft 365. También deseas que Copilot para Microsoft 365 incluya información de estos artículos de knowledge base en sus respuestas.

Crearás un conector personalizado de Microsoft Graph para exponer esta información externa dentro de Microsoft 365. Los conectores de Microsoft Graph se conectan al sistema externo (1) para recuperar contenido, usar la información de Microsoft Entra ID para autenticarse con Microsoft 365 (2) e importar el contenido a Microsoft 365 mediante Microsoft Graph API (3).

:::image type="content" source="../media/1-graph-connector-concept.png" alt-text="Diagrama que muestra el funcionamiento conceptual de un conector de Microsoft Graph.":::

En este módulo, aprenderás qué son los conectores de Microsoft Graph y por qué debes considerar su uso en tu organización. Crearás un conector de Microsoft Graph que importe archivos markdown locales a Microsoft 365. También aprenderás a asegurarte de que el contenido externo que importes solo sea accesible para los usuarios con los permisos asignados adecuados. Por último, optimizarás el conector de Microsoft Graph para su uso con Copilot para Microsoft 365.

## Requisitos previos

- Conocimientos básicos de C#
- Conocimientos básicos de la autenticación
- Acceso a un [espacio empresarial de desarrollador de Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program?ocid=MSlearn)
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Cuando estés listo para comenzar, [ve al ejercicio siguiente...](./2-exercise-configure-connection-schema.md)