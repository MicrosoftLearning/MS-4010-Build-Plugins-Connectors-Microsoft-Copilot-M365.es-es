---
lab:
  title: Introducción
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Introducción

Los agentes de Microsoft 365 Copilot permiten crear asistentes con tecnología de IA optimizados para escenarios específicos. Con instrucciones, se define el contexto para Copilot y se configura su tono de voz o cómo debe responder. Al configurar el conocimiento del agente, se le concede acceso a datos externos que no forman parte del modelo de lenguaje grande (LLM), de modo que pueda responder con mayor precisión. 

## Escenario de ejemplo

Supongamos que trabajas en un departamento de TI en una organización grande. Tu organización normaliza la TI a través de diferentes directivas que la almacenan en un sistema especializado. Tú y tus compañeros del departamento de TI reciben periódicamente preguntas que se tratan en las directivas. La búsqueda de respuestas en el sistema de administración de directivas consume mucho tiempo. Te gustaría proporcionar a tu organización un asistente con tecnología de inteligencia artificial capaz de responder a las preguntas de tus compañeros mediante información autoritativa de las directivas.

## Objetivos de aprendizaje

Al final de este módulo, podrás crear agentes declarativos para Microsoft 365 Copilot. Comprenderás cómo configurar sus instrucciones para optimizarlas para un escenario específico. También sabrás cómo integrarlos con conectores de Microsoft Graph para darles acceso a datos externos, que no forma parte del LLM de Microsoft 365 Copilot.

## Requisitos previos

- Conocimientos de lo que es Microsoft 365 Copilot y cómo funciona a nivel principiante
- Conocimientos sobre qué es un agente declarativo para Microsoft 365 Copilot
- Conocimientos sobre cómo crear un conector de Graph
- Inquilino de Microsoft 365 con Microsoft 365 Copilot y privilegios de administrador de inquilinos
- [Visual Studio Code](https://code.visualstudio.com/) con la extensión [Kit de herramientas de Teams](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) instalada
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
