---
"$title": Usa JavaScript personalizado en las páginas de AMP
"$order": '7'
formats:
  - sitios web
author:
  - Morss
contributors:
  - cristalonscript
  - fstanis
description: Una guía para usar amp-script, un componente de AMP que le permite escribir JavaScript personalizado
---

`amp-script` permite escribir y ejecutar su propio JavaScript de una manera que mantiene las garantías de rendimiento de AMP. La mayoría de los componentes de AMP permiten interacciones web comunes a través de su propia lógica, lo que le permite crear su página rápidamente sin escribir JavaScript ni importar bibliotecas de terceros. Al usar `amp-script` , puede adoptar una lógica personalizada para casos de uso específicos o necesidades únicas sin perder los beneficios de AMP.

Esta guía proporciona antecedentes sobre este componente y las mejores prácticas para su uso.

## trabajadores web

El exceso de JavaScript puede hacer que los sitios web sean lentos y no respondan. Para controlar qué páginas JavaScript de AMP se cargan y cuándo se ejecutan, las reglas de validación de AMP prohíben a los desarrolladores ejecutar JavaScript en una página web a través de una etiqueta `<script>`

[Los trabajadores web](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) presentan una forma de ejecutar JavaScript de manera más segura. Normalmente, todo JavaScript se [ejecuta en un solo hilo](https://www.youtube.com/watch?v=cCOL7MC4Pl0) , pero cada trabajador se ejecuta en un hilo propio. Esto es posible porque carecen de acceso al DOM o al `window` , y cada trabajador se ejecuta en su propio ámbito global. Por lo tanto, no pueden interferir con el trabajo de los demás ni con las mutaciones causadas por el código en el hilo principal. Solo pueden comunicarse con el hilo principal y entre sí a través de [mensajes que contienen objetos](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope/postMessage) . Los trabajadores ofrecen un camino a una web de subprocesos múltiples, una forma de encapsular JavaScript en un espacio aislado donde no puede bloquear la interfaz de usuario.

Los trabajadores no vienen con acceso al DOM. Para llenar este vacío, el equipo de AMP creó una biblioteca de código abierto llamada [WorkerDOM](https://github.com/ampproject/worker-dom/) . WorkerDOM copia el DOM en un DOM virtual y pone la copia a disposición de un trabajador. WorkerDOM también recrea un subconjunto de la API DOM estándar. Cuando el trabajador realiza cambios en el DOM virtual, WorkerDOM recrea esos cambios en el DOM real. Esto permite que el trabajador manipule el DOM y realice cambios en la página utilizando técnicas estándar. La sincronización DOM solo va en una dirección. Si el subproceso principal modifica el DOM, no existe ningún mecanismo para informar al trabajador.

`amp-script` es un envoltorio alrededor de WorkerDOM que hace que WorkerDOM se pueda usar en AMP, proporcionando conexiones a las funciones de AMP, estableciendo la API del desarrollador e introduciendo restricciones que protegen la experiencia del usuario. WorkerDOM proporciona el núcleo de la funcionalidad de `amp-script` .

## Descripción general de `amp-script`

El lenguaje JavaScript es el mismo en un trabajador que en cualquier otra parte del navegador. Por lo tanto, en `amp-script` , puede usar todas las construcciones habituales que proporciona JavaScript. WorkerDOM también recrea muchas API DOM de uso común y las pone a disposición para su uso. Admite API web comunes como `Fetch` y `Canvas` , y le brinda objetos globales seleccionados como `navigator` y `localStorage` . Puede asignar controladores para eventos del navegador de la forma habitual.

Sin embargo, `amp-script` no es compatible con toda la API DOM o la API web, ya que esto haría que su propio JavaScript fuera demasiado grande y engorroso. Consulte [la documentación](../../../documentation/components/reference/amp-script.md#supported-apis) para obtener más detalles y consulte [estos ejemplos](https://amp.dev/documentation/examples/components/amp-script/) para ver `amp-script` en uso.

`amp-script` reemplaza un puñado de métodos DOM API sincrónicos con alternativas que devuelven una Promesa. Por ejemplo, en lugar de `getBoundingClientRect()` , usa `getBoundingClientRectAsync()` . A veces, esto es necesario para las API de DOM que brindan acceso síncrono al diseño calculado y, a veces, para que `amp-script` pueda llamar al método nativo del navegador y esperar una respuesta.

Para mantener las garantías de rendimiento y estabilidad del diseño de AMP, `amp-script` viene con algunas restricciones. Si el tamaño del `amp-script` no es fijo, su código solo puede realizar una mutación si se desencadena por una interacción del usuario. No puede agregar hojas de estilo o secuencias de comandos adicionales al DOM, y no se admite [`importScripts()`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope/importScripts) Consulte [la documentación](../../../documentation/components/reference/amp-script.md#user-gestures) para obtener más detalles.

## Uso de marcos de JavaScript

Dado que la API de DOM no es totalmente compatible, ¿cuál es la mejor manera de abordar la manipulación de DOM en `amp-script` ? Aquí hay dos posibilidades.

**1) Sepa qué es compatible.** Conozca [el amplio conjunto de API compatibles](https://github.com/ampproject/worker-dom/blob/main/web_compat_table.md) . Debe pensar en la API DOM de manera diferente: menos como un gran conjunto de propiedades y métodos desarrollados durante muchos años, y más como un conjunto conciso de herramientas.

**2) Utilice Preact.** [React](https://reactjs.org/) no solo es una forma popular de crear sitios web, sino que muta el DOM utilizando un subconjunto de la API de DOM. `amp-script` puede incluir soporte completo para esta parte de la API y, por lo tanto, soporte completo para React. Dicho esto, los paquetes de React a menudo superan el [límite de 150K de](../../../documentation/components/reference/amp-script.md#size-of-javascript-code) `amp-script` . Por lo tanto, se recomienda que utilice [Preact](https://preactjs.com/) , una alternativa ligera a React. Preact está diseñado para una migración directa desde React. Con Preact, debería poder crear interacciones elaboradas con mucha menos preocupación por lo que se admite.

El equipo probó `amp-script` con marcos como [Vue](https://vuejs.org/) , [Angular](https://angularjs.org/) , [Aurelia](https://aurelia.io/) y [lit-html](https://lit-html.polymer-project.org/) , pero de forma menos extensa. Si encuentra una brecha, [presente un problema](https://github.com/ampproject/worker-dom/issues) o, mejor aún, [envíe una solicitud de extracción](https://github.com/ampproject/worker-dom/pulls) .

Dado que `amp-script` no puede admitir de manera exhaustiva la API DOM, simplemente copiar una biblioteca como [jQuery](https://jquery.com/) en un `<amp-script>` no funcionará.

## casos de uso

`amp-script` amplía el rango de la funcionalidad de su página web AMP. Dado que admite un subconjunto de DOM y API web, y dado que su uso [tiene restricciones](https://amp.dev/documentation/components/amp-script/#restrictions) , no es una solución de JavaScript para todo uso. Cualquier cuerpo sustancial de JavaScript existente probablemente necesitará modificaciones para funcionar en el contexto de `amp-script`

Sin embargo, `amp-script` presenta una buena manera de cuidar la lógica y las interacciones que los componentes AMP existentes no brindan. Los siguientes son algunos casos de uso excelentes.

### Crear nuevas interacciones

`amp-script` le brinda el poder de JavaScript y la API DOM. Le permite crear interacciones que otros componentes de AMP no pueden, abriendo una puerta a la creatividad total de la Web.

Dicho esto, antes de recurrir a `amp-script` para crear una nueva interacción, verifique si un componente AMP o una combinación de componentes pueden hacer lo mismo. Aprovechar los componentes AMP existentes finalmente hará que su código sea más pequeño y fácil de mantener. Si un componente puede lograr un efecto similar al que desea, puede <a href="#enhance-amp-components" data-md-type="link">personalizar su comportamiento con `amp-script`</a> .

Si usa `amp-script` para crear una interacción que podría ser de interés para otros desarrolladores, considere [sugerir o contribuir con](https://github.com/ampproject/amphtml/blob/main/docs/contributing.md) un nuevo componente.

### Añadir lógica avanzada

Cuando su lógica no encaja perfectamente en una expresión compacta, `amp-script` es la mejor opción. `amp-bind` permite introducir lógica en las interacciones del usuario, pero su JavaScript debe caber en una sola expresión. Debido a que el código está encapsulado en un atributo HTML, no obtendrá el beneficio del resaltado de sintaxis en su IDE, no puede establecer puntos de interrupción y la depuración puede ser un desafío. En tales casos, `amp-script` permite seguir las mejores prácticas de programación.

### Mejora los componentes de AMP<a name="enhance-amp-components"></a>

`amp-script` tiene acceso a las variables de estado de AMP y a los `AMP.getState()` y `AMP.setState()` . Esto proporciona una ruta para mejorar los componentes AMP existentes con su propia lógica. También permite afectar el DOM fuera del propio componente `<amp-script>` Vea [aquí](https://amp.dev/documentation/examples/components/amp-script/#interacting-with-%3Camp-state%3E) y [aquí](https://amp.dev/documentation/examples/components/amp-script/#interacting-with-amp-components) para ver ejemplos.

### Reemplazar amp-bind y amp-list

Si es nuevo en AMP y se siente cómodo con JavaScript, puede ser tentador usar `amp-script` para cada interacción con el navegador. Sin embargo, para interacciones más simples, es probable que desee aprender a usar `amp-bind` y el sistema de [acciones y eventos de AMP.](../learn/amp-actions-and-events.md) Para interacciones más elaboradas, o para casos que requieren más lógica y manipulación de variables de estado compleja, `amp-script` [probablemente será más fácil](#when-to-replace-amp-bind-and-amp-list) .

### Manejar datos del servidor

AMP le permite recuperar datos del servidor usando `amp-list` y formatearlo usando [`amp-mustache`](../../examples/documentation/amp-mustache.html) . En los casos en que las plantillas de bigote no sean suficientes, `amp-script` puede obtener los datos, formatearlos e inyectar los datos formateados en el DOM. Si necesita modificar los datos del servidor antes de enviarlos a `amp-mustache` , una función `amp-script` puede ser la fuente de datos para `amp-list` . Consulte [la documentación](../../../documentation/components/reference/amp-list.md?format=websites#initialization-from-amp-state) para obtener detalles y un ejemplo de código.

### Introducir nuevas capacidades

Puede usar `amp-script` para aprovechar áreas de la API web y la API DOM que actualmente no son accesibles para los componentes de AMP, o para usar estas API de formas que los componentes de AMP no admiten. Por ejemplo, `amp-script` compatible con `WebSockets` ( [ejemplo](https://amp.dev/documentation/examples/components/amp-script/#using-a-websocket-for-live-updates) ), `localStorage` y `Canvas` . Admite una amplia variedad de eventos del navegador, por lo que puede escuchar eventos más allá de [los que AMP pasa a los componentes tradicionales](https://amp.dev/documentation/guides-and-tutorials/learn/amp-actions-and-events/) . Y dado que `amp-script` brinda acceso al `navigator` , puede recuperar información sobre [el navegador del usuario](https://amp.dev/documentation/examples/components/amp-script/#detecting-the-operating-system) o el [idioma preferido](https://amp.dev/documentation/examples/components/amp-script/#personalization) .

## Cuándo reemplazar amp-bind y amp-list<a name="when-to-replace-amp-bind-and-amp-list"></a>

Para un nuevo desarrollador de AMP que se siente cómodo con JavaScript, puede parecer más fácil usar siempre `amp-script` que aprender `amp-bind` y `amp-list` . Sin embargo, en algunos casos, `amp-bind` y `amp-list` encajan mejor.

`amp-bind` es generalmente más sencillo para las interacciones básicas, donde su estrecha integración en las etiquetas HTML es atractiva. En este ejemplo, al presionar un botón se cambia un fragmento de texto. El enlace de datos de AMP hace que esto sea sencillo y fácil de leer:

```html
<p [text]="name">Rajesh</p>
<button on="tap:AMP.setState({name: 'Priya'})">I am Priya</button>
```

De manera similar, cuando usa una API cuya salida controla, puede implementar la lógica comercial en el servidor. Es posible que pueda formatear los datos que genera la API para que encajen sin problemas en una plantilla de `amp-mustache` `amp-list` es una buena opción en estos casos.

`amp-bind` también proporciona un mecanismo sencillo para comunicarse entre los componentes de AMP. En este ejemplo, tocar una imagen en un `<amp-selector>` establece la variable de estado `selectedSlide` en `0` , lo que a su vez hace que un `<amp-carousel>` mueva a su primera diapositiva.

```html
<amp-carousel slide="selectedSlide">
...
</amp-carousel>

<amp-selector>
  <amp-img on="tap:AMP.setState({selectedSlide: 0})"/>
</amp-selector>
```

Los componentes de AMP interactivos tradicionales también pueden ser más adecuados para las interacciones que abarcan grandes secciones de una página web, ya que es posible que no desee envolver gran parte del DOM en un `<amp-script>` . `amp-list` y `amp-bind` están estrechamente integrados con el resto de AMP, lo que facilita el uso de enlaces en cualquier lugar de una página web.

Sin embargo, en páginas que involucran variables de estado más complejas o interacciones múltiples, el uso de `amp-script` resultará en un código más simple y fácil de mantener. Tome este ejemplo, del [sitio de demostración de comercio electrónico de AMP Camp](https://camp.samples.amp.dev/product-details?categoryId=women-shirts&productId=79121) :

```html
<amp-selector
  name="color"
  layout="container"
  [selected]="product.selectedColor"
  on="select:AMP.setState({
      product:
        {
          selectedSlide: product[event.targetOption].option - 1,
          selectedColor: event.targetOption,
          selectedSize: product[event.targetOption].sizes[product.selectedSize] != null ?
                        product.selectedSize :
                        product[event.targetOption].defaultSize,
          selectedQuantity: 1
        }
      })"
></amp-selector>
```

Esta demostración se creó antes de que se lanzara `amp-script` Pero este tipo de lógica sería más fácil de escribir y depurar en JavaScript. Para páginas con más lógica comercial, el uso de `amp-script` le permitirá evitar confusiones y seguir mejores prácticas de programación.

En muchos casos, querrá usar `amp-script` y `amp-bind` en la misma página. Implemente `amp-bind` para interacciones más simples, recurriendo a `amp-script` cuando necesite más lógica o estructura. Además, aunque `amp-script` solo puede realizar mutaciones en sus hijos DOM, [como se indicó anteriormente](#enhance-amp-components) , puede afectar el resto de la página mediante la mutación de las variables de estado. `amp-bind` hace el resto, como en [este ejemplo](https://amp.dev/documentation/examples/components/amp-script/#interacting-with-%3Camp-state%3E) .

Aunque WorkerDOM cambiará el DOM real cuando su código cambie el DOM virtual al que tiene acceso, no existe ningún mecanismo para sincronizar en la dirección inversa. Por lo tanto, no es recomendable usar `amp-bind` u otros medios para modificar los elementos secundarios de su `<amp-script>` . Reserve esa área de la página para su `amp-script` JavaScript.

## Contribuir a `amp-script`

`amp-script` siempre está evolucionando, al igual que AMP. Puedes ayudar a mejorarlo involucrándote. ¡Piense en nuevas funciones que otros desarrolladores también podrían necesitar, [envíe problemas](https://github.com/ampproject/amphtml/issues) y, por supuesto, [sugiera y contribuya con nuevas funciones](https://github.com/ampproject/amphtml/pulls) !
