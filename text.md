# isomorphic js

## Presentación

## wtf is isomorphic JavaScript

isomorphic

<blockquote class="twitter-tweet" lang="en"><p>I may be thick, but what does isomorphic mean</p>&mdash; Horse JS (@horse_js) <a href="https://twitter.com/horse_js/status/557976031613550592">January 21, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


adjective
	1.	corresponding or similar in form and relations.

Aplicado a JavaScript podemos entenderlo como que los dos tipos de entornos que tenemos, van a ejecutar un código similar. Es decir, vamos a poder compartir código entre el cliente y el servidor. 
¿Y por qué querríamos compartir código entre el cliente y el servidor?

Ejemplo. Formulario de registro


\* input email
\* input password

Supongamos que este formulario tiene validaciones en cliente. 
Tenemos una validación en cliente que podría ser algo así

email

```js
function validate() {
	return validateEmail(email) && validatePassword(password);
}

function validateEmail(email) { 
    var re = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
    return re.test(email);
} 

function validatePassword(password) {
	return password.length > 5;
}
```

Como sabemos algo básico de seguridad es que no podemos fiarnos de las validaciones que se hacen en servidor, por lo tanto tenemos que implementar la misma validación en servidor.

```java
public boolean validate (String email, String password) {
	return validateEmail(email) && validatePassword(password);
}

private boolean validateEmail(String email) {
           String ePattern = "^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@((\\[[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\])|(([a-zA-Z\\-0-9]+\\.)+[a-zA-Z]{2,}))$";
           java.util.regex.Pattern p = java.util.regex.Pattern.compile(ePattern);
           java.util.regex.Matcher m = p.matcher(email);
           return m.matches();
    }

private boolean function validatePassword(String password) {
	return password.length() > 5;
}

```

Es aquí donde empiezan el problema con la duplicidad del código. Este código debería comportarse exactamente. Pero la expresión regular es complicado decir si es exactamente la misma porque la sintaxis es distinta. Además todo código duplicado tiene problemas de mantenimiento. 
Qué pasa si ahora decidimos que una contraseña segura tiene que tener al menos 7 caracteres. Tenemos que acordarnos de cambiarlo en ambos sitios.

Lo ideal sería reutilizar este código para que se comporte exactamente igual.
Para ello tenemos varias opciones, utilizar alguna de estas locuras. Lenguajes que compilan a JavaScript

https://github.com/jashkenas/coffeescript/wiki/list-of-languages-that-compile-to-js


O, la opción más inteligente, implementar también nuestro servidor en JavaScript. 

Node.JS


Para que el código que estamos utilizando en nuestro cliente funcione en nuestra aplicación Node. Tenemos que hacer algunos cambios

validate.js

```js
module.exports = function validate() {
	return validateEmail(email) && validatePassword(password);
}

function validateEmail(email) { 
    var re = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
    return re.test(email);
} 

function validatePassword(password) {
	return password.length > 5;
}
```

app.js

```js
var validate = require(“validate”);
valiate(“user”, “password”);
```


Pero ahora si intentamos utilizar este fichero en el navegador nos dará un error porque el navegador no tiene el sistema de módulos que tiene Node.js. Esto en el futuro puede cambiar porque la nueva especificación de JavaScript ES6 define un sistema de módulos. Pero por ahora tenemos que buscar alguna herramienta que nos permita utilizar modulos en el navegador.

Browserify

Browserify lets you require('modules') in the browser by bundling up all of your dependencies.

```js
npm install -g browserify
browserify main.js -o bundle.js
<script src="bundle.js"></script>
```



\o/

Success

Si browserify nos permite reutilizar módulos que pasa si además de utilizarlo para nuestros propios módulos. Lo utilizamos con módulos de terceros.

**CAMBIAR POR UNA LIBRERÍA DE VALIDACIÓN**

```
npm install underscore;
```
 

```js
var _ = require(“underscore”);
console.log(_.map([1, 2, 3], function (i) {
	return i * 2;
});
```

browserify main.js -o bundle.js
<script src="bundle.js"></script>

\o/

Doble success!!

Pero hay código que necesariamente tiene que ser distinto entre el cliente y el servidor.

Por ejemplo una petición http

```
$.get('/user/1', function(data, textStatus, xhr){

});
```

Está claro que nuestro servidor no va a poder ejecutar jQuery.

Pero podemos buscar otra librería que sea compatible tanto en cliente como en servidor. Por ejemplo https://github.com/visionmedia/superagent

De esta forma el código de nuestra aplicación puede ser exactamente el mismo.

```js
var request = require(“superagent”);
request.post('/user/1', cat, function(error, res){
});
```

También hay formas de decir de decir que sustituya una librería por otra en tiempo de creación del bundle. Así podemos cargar una versión que sea compatible con el navegador pero que tenga una API similar.

```js
{
  "name": "mypkg",
  "version": "1.2.3",
  "main": "main.js",
  "browser": {
    "lib/foo.js": "lib/browser-foo.js"
  }
}
```

Browserify es una herramienta muy potente
Tutorial de browserify

https://github.com/substack/browserify-handbook


En qué más casos podemos compartir código entre cliente y servidor

* Model validation
* Date & currency formatting
* I18N
* Api interaction
* Routing
* Templating

Evolución de aplicación web

Fat-server. El servidor es el encargado de renderizar todo el HTML. El principal inconveniente es que cada vez que el usuario navegaba de una página a otra, se renderizaba la página nueva completa.

Fat-client: Con la revolución AJAX. Pasamos al modelo inverso. Vamos a poner mucha más lógica de la aplicación en el cliente, que se conectará mediante la api rest. De forma que cuando hagamos una navegación. Se renderizen únicamente los cambios. Esto está muy bien, conseguimos que la aplicación vaya mucho más rápido.


Esta solución no es perfecta y viene con algunos problemas:

* La primera carga es más lenta. Hasta que el cliente no se haya descargado los ficheros de javascript tu aplicación no verá nada. Esto puede llegar a ser un problema para dispositivos más limitados como los dispositivos móviles con una mala conexión.

Hay un concepto importante above-the-fold que consiste en el contenido que se va sin que el usuario haga scroll.

https://developers.google.com/speed/docs/insights/PrioritizeVisibleContent

En 2012 twitter se dio cuenta de esto y volvieron a renderizar en servidor 

https://blog.twitter.com/2012/improving-performance-on-twittercom


* También hay problemas con el SEO. Esto ya no es tanto un problemas desde Mayo de 2014 que google empezó a interpretar javascript 

http://googlewebmastercentral.blogspot.fr/2014/05/understanding-web-pages-better.html

Pero no todo en este mundo es google, existen otros buscadores que todavía no interpretan JavaScript.


Entonces, qué opción elegimos?

Podemos optar por una solución híbrida. Quedarnos con lo mejor de cada una de las opción. Para el primer render utilizaremos servidor y los siguientes los haremos en cliente, cuando nuestra aplicación se haya cargado.

Templating

Debemos buscar una sistema de plantillas que sea capaz de funcionar tanto en cliente como en servidor. Por ejemplo podemos utilizar handlebars.

js
```
<div class="entry">
  <h1>{{title}}</h1>
  <h2>By {{author.name}}</h2>

  <div class="body">
    {{body}}
  </div>
</div>
```

Normalmente nuestros clientes son más complejos y utilizan frameworks que vienen con sus propios sistemas de plantillas, como angular o ember.js

Si utilizamos angular estamos jodidos. 
http://berzniz.com/post/99158163051/isomorphic-javascript-angular-js-is-not-the

El principal problemas es que angular necesita el DOM para renderizarse. Existen implementaciones completas del DOM para node como jsdom https://github.com/tmpvar/jsdom. Pero sería bastante lento. Parece que angular no es la herramienta adecuada para trabajar con isomorphic js.

Ember está trabajando en server-side rendering. Llamaron la feature FastBoot y están escribiendo una serie de posts donde van explicando el progreso.
http://emberjs.com/blog/2014/12/22/inside-fastboot-the-road-to-server-side-rendering.html
Pero a día de hoy es todavía un wip.


React.js

React es a día de hoy la opción más solida para conseguir server-side rendering.

```
var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

React.render(<HelloMessage name="John" />, document.body);
```

Pero también tiene la opción de renderizar en un string

```js
React.renderToString(<HelloMessage name="John" />);
```

Esto genera 

```html
<div data-reactid=".1" data-react-checksum="425337345"><span data-reactid=".1.0">Hello </span><span data-reactid=".1.1">John</span></div>
```

Cuando el cliente react encuentra código renderizado por el servidor, únicamente añade los listeners!

Mismos componentes utilizados en cliente y servidor!

Pa la routas podemos usar una opción similar. Existen varios isomorphic javascript routers. Por ejemplo uno de ellos es 

http://react-components.com/component/monorouter


router

```js

var monorouter = require('monorouter');
var reactRouting = require('monorouter-react');

monorouter()
  .setup(reactRouting())
  .route('/', function(req) {
    this.render(MyView);
  })
  .route('/pets/:name/', function(req) {
    this.render(PetView, {petName: req.params.name});
  });
```

server

```js
var express = require('express');
var router = require('./path/to/my/router');
var monorouterMiddleware = require('connect-monorouter');

var app = express();
app.use(monorouterMiddleware(router));

var server = app.listen(3000, function() {
    console.log('Listening on port %d', server.address().port);
});
```

browser

```js
router
  .attach(document)
  .captureClicks();
```


# Meteor

