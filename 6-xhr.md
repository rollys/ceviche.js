# Capítulo 6: Peticiones asíncronas

> Junto al DOM, las peticiones asíncronas son las características más utilizadas en un sitio o aplicación web, y permiten disminuir la carga que contiene una llamada al servidor, dando la impresión de tener un sitio mucho más rápido.

Una petición asíncrona es una operación que, mientras esté siendo procesada, deja libre al navegador para que pueda hacer otras operaciones. Llamaremos peticiones asíncronas a las operaciones que tengan que ver con realizadas llamadas a servidores; sin embargo, existen muchas más operaciones asíncronas en JavaScript, como las que se realizan para leer y escribir en archivos, obtener la geolocalización de un navegador, o manejar base de datos.

Las peticiones asíncronas en el navegador se realizan con la función `XMLHttpRequest`, la cual permite realizar peticiones de tipo `GET` (obtener información), `POST` (enviar información), y otros más.

## XMLHttpRequest

Para poder enviar una petición asíncrona a un servidor se debe crear una instancia de la función `XMLHttpRequest`, de la siguiente manera:

```javascript
var xhr = new XMLHttpRequest();
```

Luego, debemos definir la dirección a donde se enviará la petición, e indicar el tipo de petición (`GET`, `POST`, etc). El último parámetro es importante: es el que define si la petición será asíncrona o no. Si la petición es síncrona, se corre el riesgo de congelar el navegador, ya que este dejará de hacer cualquier operación y se dedicará a realizar la petición síncrona.

```javascript
xhr.open('GET', url, true);
```

Por último, enviamos la petición al servidor con el método `send`. En este momento el navegador continúa ejecutando el código que está después de esta línea, mientras que, por interno, la petición es esperada.

```javascript
xhr.send();
```

Hasta este punto, el proceso está incompleto: Enviamos la petición pero no sabemos en qué momento ha terminado de procesarse, ni cuál es la información que el servidor ha devuelto.

Felizmente, `XMLHttpRequest` *hereda* de `EventTarget`. Recordemos qué hacía `EventTarget`:

> Todos los elementos del DOM, además de `window`, heredan de la interfaz `EventTarget`, el cual permite enlazar eventos a callbacks definidos dentro de la aplicación. La interfaz `EventTarget` tiene 3 métodos: `addEventListener`, `removeEventListener` y `dispatchEvent`.

Las peticiones asíncronas tienen sus propios eventos:

* `abort`: Lanzado cuando la petición ha sido cancelada, vía el método `abort()`.
* `error`: Lanzado cuando la petición ha fallado.
* `load`: Lanzado cuando la petición ha sido completada satisfactoriamente.
* `loadend`: Lanzado cuando la petición ha sido completada, ya sea con éxito o con error.
* `loadstart`: Lanzado cuando la petición ha sido iniciada.
* `progress`: Lanzado cuando la petición esté enviando o recibiendo información.
* `readystatechange`: Lanzado cuando el atributo `readyState` cambie de valor.
* `timeout`: Lanzado cuando la petición ha sobrepasado el tiempo de espera límite (definido por la propiedad `timeout`).

Así que debemos escuchar al menos un evento para saber si la petición devuelve algún tipo de información. En este caso escucharemos 2 eventos importantes: `error` para saber si hubo un error en la petición, y `readystatechange` para saber los distintos estados de la petición:

```javascript
xhr.addEventListener('error', function(e) {
  console.log('Un error ocurrió', e);
});

xhr.addEventListener('readystatechange', function() {
  console.log('xhr.readyState:', xhr.readyState);
});
```

Juntando cada parte, tenemos el siguiente código, el cual obtiene los últimos *tweets* que contengan la palabra *ceviche*:

```javascript
var xhr = new XMLHttpRequest();

var url = 'http://coffeemaker.herokuapp.com/twitter.json?q=ceviche';

xhr.open('GET', url, true);

xhr.addEventListener('error', function(e) {
  console.log('Un error ocurrió', e);
});

xhr.addEventListener('readystatechange', function() {
  console.log('xhr.readyState:', xhr.readyState);
});

xhr.send();
```

La ejecución de este código daría el siguiente resultado en la consola:

```javascript
// xhr.readyState: 2
// xhr.readyState: 3
// xhr.readyState: 4 
```

La propiedad `readyState` indica el estado de la petición y tiene los siguientes valores:

* `0`: El valor inicial.
* `1`: Luego de haber ejecutado el método `open()`.
* `2`: El navegador envió la petición (método `send()`) pero aún no recibe una respuesta.
* `3`: El navegador está esperando por la respuesta a la petición.
* `4`: La petición obtiene información de respuesta.

Ahora ya sabemos los estados por los que pasa una petición, pero aún no sabemos cuál es la respuesta. Para obtenerla utilizamos la propiedad `responseText`.

```javascript
xhr.addEventListener('readystatechange', function() {
  if (xhr.readyState === 4) {
    console.log(xhr.responseText);
  }
});
```

En este caso, verificamos que el `readyState` sea 4, dado que la petición solo tendrá una respuesta cuando tenga dicho estado.

### Peticiones POST

Las peticiones asíncronas son, generalmente, peticiones GET, por lo que si se envían valores en la petición, estos estarán expuestos fácilmente en la URL de la misma petición, creando un potencial problema de seguridad. Así mismo, las URLs tienen un límite de caracteres, por lo que no se podrá enviar toda la información que uno desee. Estos dos puntos son cruciales al momento de realizar peticiones, asíncronas o no. Es aquí donde aparecen las peticiones POST: peticiones que pueden enviar gran cantidad de información, la cual no es accesible de forma fácil.

En el caso de `XMLHttpRequest`, crear una petición POST es sencillo y agrega dos pasos a lo descrito anteriormente: indicar el tipo de petición y agregar los valores que se deseen ingresar.

Para indicar el tipo de petición simplemente cambiamos el primer parámetro del método `open()`:

```javascript
xhr.open('POST', url, true);
```

Cabe notar que la url debe aceptar peticiones POST, lo cual es definido en el servidor.

Para agregar los valores que se desean enviar se utiliza una instancia de `FormData`, donde se agregan los valores utilizando el método `append()`:

```javascript
var data = new FormData();

data.append('nombre', 'valor');
```

Luego, el nuevo objeto `FormData` debe ser pasado como parámetro en el método `send()` de la instancia de `XMLHttpRequest`:

```javascript
xhr.send(data);
```

Así, el código final quedaría de esta forma:

```javascript
var xhr = new XMLHttpRequest();

var url = 'http://coffeemaker.herokuapp.com/form';

xhr.open('POST', url, true);

xhr.addEventListener('error', function(e) {
  console.log('Un error ocurrió', e);
});

xhr.addEventListener('readystatechange', function() {
  if (xhr.readyState === 4) {
    console.log(xhr.responseText);
  }
});

var data = new FormData();

data.append('nombre', 'valor');

xhr.send(data);
```

Y el resultado de la petición sería:

```javascript
// {"nombre":"valor"}
```

---

#### Carga asíncrona de archivos

Una de las características de `FormData` es que no solo permite adjuntar texto, si no también archivos. Esto se logra agregando instancias de `File` con el método `append`. Recordemos que cada campo de formulario de tipo archivo (`<input type="file">`) tiene una propiedad llamada `files`, el cual contiene una lista de instancias `File`. De esta forma, podemos subir archivos a un servidor de manera **asíncrona**.

La ventaja de `FormData` es que, al crear una instancia, podemos pasarle como parámetro un elemento formulario (`<form>`), por lo que automáticamente toma todos los campos del formulario, siempre y cuando tengan un nombre (atributo `name`), incluyendo los campos de tipo archivo.

```javascript
var form = document.querySelector('#formulario_comentario');
var data = new FormData(form);
```

---

## JSON

En los ejemplos donde se utiliza `coffeemaker.herokuapp.com` vemos que las respuestas vienen en forma de texto, pero con un formato que nos recuerda a objetos u arreglos en JavaScript. Este formato se llama JSON (JavaScript Object Notation), y permite enviar y recibir información de una manera simple y liviana.

Para poder leer este formato utilizamos el método `JSON.parse`, el cual es nativo en todos los navegadores, y en Internet Explorer 9 y superiores:

```javascript
JSON.parse('{"nombre":"valor"}');
// Object {nombre: "valor"}
```

En el caso opuesto, si deseamos convertir un objeto a una cadena en formato JSON (por ejemplo, si deseamos guardarlo en `localStorage`), utilizamos el método `JSON.stringify`, el cual convertirá un objeto a su contraparte en JSON.

```javascript
JSON.stringify({nombre: 'valor'});
// "{"nombre":"valor"}"
```

Este método podría no funcionar en casos donde un objeto o un arreglo contiene una referencia a sí mismo:

```javascript
var a = [];

a.push(a);

JSON.stringify(a);
// Uncaught TypeError: Converting circular structure to JSON

JSON.stringify(window);
// Uncaught TypeError: Converting circular structure to JSON
```

En el caso de `window`, este tiene propiedades como `top`, `parent` o `self` que son referencias a sí mismos.