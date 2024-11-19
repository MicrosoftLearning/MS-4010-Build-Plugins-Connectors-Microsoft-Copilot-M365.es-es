---
lab:
  title: 'Ejercicio 2: Ejecución del ejemplo como complemento de Copilot'
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# Ejercicio 2: Ejecución del ejemplo como complemento de Copilot

En este ejercicio, ejecutarás la aplicación como un complemento para Microsoft 365 Copilot. Experimentarás con varios mensajes y observarás cómo se invoca el complemento mediante distintos parámetros.

> [!NOTE]  
> Para realizar el ejercicio siguiente, la cuenta debe tener una licencia válida de Microsoft 365 Copilot.

## Tarea 1: Prueba en Microsoft 365 Copilot (parámetro único)

1. En el raíl de la aplicación a la izquierda, selecciona la aplicación **Copilot**.

1. En el lado derecho del cuadro de redacción, selecciona el icono de **complemento** 1️⃣ y habilita el complemento **Northwind Inventory** 2️⃣.

    ![Captura de pantalla del panel pequeño con una alternancia para cada complemento.](../media/3-02-plugin-panel.png)

1. Para obtener los mejores resultados, selecciona el icono **Nuevo chat** situado en la parte superior derecha antes de cada solicitud o conjunto de mensajes relacionados.

    ![Captura de pantalla de Copilot en la que muestra su nueva pantalla de chat.](../media/3-01-new-chat.png)

1. Prueba las siguientes indicaciones que usan un único parámetro de la extensión de mensajes:

    - _Find information about Chai in Northwind Inventory._

    - _Find discounted seafood in Northwind. Show a table with the products, supplier names, average discount rate, and revenue per period._

El último debe hacer referencia a los documentos que cargaste en OneDrive. A medida que hagas las pruebas, mira los mensajes de registro en Visual Studio Code. Deberías poder ver cuándo Copilot llama al complemento y envía una consulta. Por ejemplo, después de solicitar **alimentos marinos con descuento**, Copilot emitió esta consulta con el comando `discountSearch`.

![Captura de pantalla del archivo de registro que muestra una búsqueda de descuentos para alimentos marinos.](../media/3-02-a-query-log-1.png)

Es posible que veas citas de los datos de Northwind de 3 maneras. Si hay una sola referencia, Copilot podría mostrar toda la tarjeta.

![Captura de pantalla de la tarjeta adaptable para Chai insertada en una respuesta de Copilot.](../media/3-03-a-response-on-chai.png)

Si hay varias referencias, Copilot podría mostrar un número pequeño junto a cada una. Puedes mantener el puntero sobre estos números para mostrar la tarjeta adaptable. Las referencias también se mostrarán debajo de la respuesta.

![Captura de pantalla que muestra los números de referencia insertados en una respuesta de Copilot: mantener el puntero sobre el número muestra la tarjeta adaptable.](../media/3-03-response-on-chai.png)

Prueba estas tarjetas adaptables para tomar medidas sobre los productos. Ten en cuenta que esto no afecta a las respuestas anteriores de Copilot.

No dudes en intentar crear tus propias indicaciones. Descubrirás que solo funcionan si Copilot puede consultar el complemento para obtener la información necesaria. Esto destaca la necesidad de prever los tipos de mensajes que emitirán los usuarios y proporcionar los tipos correspondientes de consultas para cada uno. Tener varios parámetros hará que esto sea más eficaz.

## Tarea 2: Prueba en Microsoft 365 Copilot (varios parámetros)

En este ejercicio probarás algunas indicaciones que ejercen la característica de varios parámetros en el complemento de ejemplo. Estos mensajes solicitarán datos que se pueden recuperar por **nombre**, **categoría**, **estado de inventario**, **ciudad del proveedor** y **nivel de existencias**, tal como se define en el **manifiesto de la aplicación**.

Por ejemplo, intenta solicitar **_Find Northwind beverages with more than 100 items in stock_**. Para generar la respuesta, Copilot debe identificar los productos:

- Donde la categoría es **beverages**.
  
  _Y_

- Donde el estado del inventario es **in stock**.

  _Y_

- Donde el **stock level** es mayor que **100**.

Si miras el archivo de registro, puedes ver que Copilot pudo comprender este requisito y rellenar 3 de los parámetros en el primer comando de extensión de mensajes.

![Captura de pantalla del registro que muestra una consulta para categoryName=beverages y stockLevel=100- .](../media/3-06-find-northwind-beverages-with-more-than-100.png)

El código del complemento aplica los tres filtros, lo que proporciona un conjunto de resultados de solo 4 productos. Copilot usa la información sobre las tarjetas adaptables resultantes, entregando un resultado similar al siguiente:

![Captura de pantalla que muestra a Copilot generando una lista con viñetas de productos con referencias.](../media/3-06-b-find-northwind-beverages-with-more-than-100.png)

Con este mensaje, Copilot también puede buscar en los archivos de OneDrive para encontrar los términos de pago con el contrato de cada proveedor. En este caso, observarás que algunas de las referencias no tendrán el icono de **Northwind Inventory**, sino el icono de **Word**.

![Captura de pantalla que muestra a Copilot extrayendo los términos de pago de los contratos en SharePoint.](../media/3-06-c-payment-terms.png)

Estos son algunos mensajes más para probar:

- _Find Northwind dairy products that are low on stock. Show me a table with the product, supplier, units in stock and on order._

- _We’ve been receiving partial orders for Tofu. Find the supplier in Northwind and draft an email summarizing our inventory and reminding them that they should stop sending partial orders per our MOQ policy._

- _Northwind will have a booth at Microsoft Community Days in London. Find products with local suppliers and write a LinkedIn post to promote the booth and products. Emphasize how delicious the products are and encourage people to attend our booth._

- _What beverage is high in demand due to social media that is low stock in Northwind in London. Reference the product details to update stock._

¿Qué mensajes te funcionan mejor? Intenta crear tus propios mensajes y observa los mensajes de registro para ver cómo Copilot accede al complemento.

### Sugerencia de solución de problemas

Si tienes dificultades al probar el complemento, puedes habilitar el **modo de desarrollador**. El modo de desarrollador proporciona información sobre el complemento seleccionado por el orquestador de Copilot para responder a la solicitud. También muestra las funciones disponibles en el complemento y el código de estado de la llamada API.

Para habilitar el modo de desarrollador, escribe lo siguiente en Copilot:

```console
-developer on
```

Ejecuta los resultados de tu solicitud y las salidas del modo de desarrollador que tienen un aspecto similar al siguiente: 

![Captura de pantalla de Copilot en modo de desarrollador.](../media/3-03-b-developer-mode.png)

Como puedes observar, debajo de la respuesta generada por Copilot, tenemos una tabla que nos proporciona información detallada sobre lo que sucedió en segundo plano:

- En **Enabled plugins**, puedes ver que Copilot ha identificado que el complemento de Northwind Inventory está habilitado.

- En **Matched functions**, puedes ver que Copilot ha determinado que el complemento de Northwind Inventory ofrece tres funciones: `inventorySearch`, `discountSearch` y `companySearch`.

- En **Selected functions for execution**, puedes ver que Copilot ha seleccionado la función `inventorySearch` para responder a la solicitud.

- En **Function execution details**, puedes ver información detallada sobre la ejecución, como la respuesta HTTP devuelta por el complemento al motor de Copilot.

## Comprobar el trabajo

Después de completar las tareas de este ejercicio, deberías ser capaz de usar el complemento **Northwind Inventory** en Microsoft 365 Copilot. 

Ahora que has completado ese ejercicio, está listos para agregar un nuevo comando a la extensión de mensajería para que puedas expandir las funcionalidades del complemento y realizar más tareas. 

[Ir al ejercicio siguiente...](./5-exercise-3-add-new-command.md)
