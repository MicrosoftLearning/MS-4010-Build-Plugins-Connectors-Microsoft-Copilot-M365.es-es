---
lab:
  title: Introducción
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Introducción

Los agentes de Microsoft 365 Copilot permiten crear asistentes con tecnología de IA optimizados para escenarios específicos. Con instrucciones, se define el contexto del agente y se especifican configuraciones como el tono de voz o cómo debe responder. Al configurar las aptitudes del agente, le ofreces la capacidad de interactuar con sistemas externos, desencadenar cierto comportamiento en condiciones del sistema o usar lógica de flujo de trabajo personalizada. Un tipo de aptitud supone acciones que permiten que un agente declarativo se comunique con las API para recuperar y modificar datos.

![Diagrama que muestra la anatomía de un agente declarativo para Microsoft 365 Copilot.](../media/LAB_02/1-anatomy-declarative-agent.png)

## Escenario de ejemplo

Supongamos que trabajas en una organización donde pedís comida regularmente a un restaurante local. El restaurante funciona con un menú diario que publican en Internet. Quieres poder ver rápidamente qué platos están disponibles, pero también tener en cuenta los alergenos en caso de que tengas invitados. El restaurante expone su menú a través de una API. En lugar de crear una aplicación independiente, quieres integrar la información en Microsoft 365 Copilot para encontrar fácilmente los platos disponibles y conocer sus ingredientes. Quieres usar lenguaje natural para examinar el menú y realizar pedidos.

## ¿Qué hará?

En este módulo, crearás una acción para un agente declarativo con un complemento de API. La acción permite al agente interactuar con un sistema externo mediante su API anónima. Aprenderá a:

- **Creación**: crea un complemento de API que se conecte a una API anónima.
- **Configuración**: configura el complemento de API para mostrar los datos de la API.
- **Extensión**: extiende un agente declarativo con una acción mediante un complemento de API.
- **Aprovisionar**: cargarás el agente declarativo en Microsoft 365 Copilot y validarás los resultados.

![Captura de pantalla de un agente declarativo que responde a un usuario con información de una API externa.](../media/LAB_02/1-agent-response-api-plugin.png)

## Duración del laboratorio

- **Tiempo estimado para completarlo**: 35 minutos

## Objetivos de aprendizaje

Al final de este módulo, sabrás cómo integrar agentes declarativos con complementos de API conectados a API anónimas, para permitirles interactuar con sistemas externos en tiempo real.