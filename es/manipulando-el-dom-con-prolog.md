# Manipulando el DOM con Prolog

El módulo`dom` de Tau Prolog incorpora nuevos tipos de términos y predicados para la manipulación del DOM y la gestión de los eventos del navegador. Puedes encontrar más información sobre estos predicados en los [predicados de referencia del módulo dom](http://tau-prolog.org/documentation#dom).

## Selectores

Tau Prolog incluye tres predicados (no deterministas) para buscar elementos del DOM:

* [get_by_id/2](http://tau-prolog.org/documentation/prolog/dom/get_by_id/2): busca el elemento con el identificador especificado.
* [get_by_class/2](http://tau-prolog.org/documentation/prolog/dom/get_by_class/2): busca todos los elementos (por reevaluación) que contengan la clase especificada.
* [get_by_tag/2](http://tau-prolog.org/documentation/prolog/dom/get_by_tag/2): busca todos los elementos (por reevaluación) de la etiqueta especificada.

```html
<div id="block-a" class="circle">content a</div>
<div id="block-b" class="circle">content b</div>
<div id="block-c" class="square">content c</div>
```

```javascript
var session = pl.create();
session.consult(":- use_module(library(dom)).");

session.query("get_by_tag(div, B), html(B, X).");
session.answer(); // {B/<html>(block-a), X/'content a'}
session.answer(); // {B/<html>(block-b), X/'content b'}
session.answer(); // {B/<html>(block-c), X/'content c'}
session.answer(); // false

session.query("get_by_class(circle, B), html(B, X).");
session.answer(); // {B/<html>(block-a), X/'content a'}
session.answer(); // {B/<html>(block-b), X/'content b'}
session.answer(); // false
```

Si no existe un elemento con el identificador, la clase o la etiqueta especificada, los predicados simplemente fallan silenciosamente. Además, este módulo incluye predicados para navegar por el DOM a partir de otros objetos HTML:

* [parent_of/2](http://tau-prolog.org/documentation/prolog/dom/parent_of/2): si el primer parámetro está instanciado y el segundo no, devuelve el padre del primer elemento. Si el segundo parámetro está instanciado pero el primero no, devuelve (por reevaluación) todos los hijos del segundo. Si ambos parámetros están instanciados, comprueba que el primer elemento es hijo del segundo.
* [sibling/2](http://tau-prolog.org/documentation/prolog/dom/sibling/2): si el primer parámetro está instanciado y el segundo no, devuelve el hermano inmediantamente a la derecha del primero. Si el segundo parámetro está instanciado pero el primero no, devuelve el hermano inmediatamente a la izquierda del segundo. Si ambos parámetros están instanciados, comprueba que ambos elementos son hermanos inmediatos.
		
## Modificar el DOM
		
El predicado [create/2](http://tau-prolog.org/documentation/prolog/dom/create/2) recibe un átomo representando una etiqueta HTML (`div`,`a`,`table`, etcétera) y crea un nuevo objeto HTML. Los nuevos objetos HTML creados pueden ser insertados en el DOM mediante los siguientes predicados:

* [append_child/2](http://tau-prolog.org/documentation/prolog/dom/append_child/2): inserta el segundo elemento como último hijo del primer elemento.
* [insert_after/2](http://tau-prolog.org/documentation/prolog/dom/insert_after/2): inserta el segundo elemento justo a la derecha del primer elemento.
* [insert_before/2](http://tau-prolog.org/documentation/prolog/dom/insert_before/2): inserta el segundo elemento justo a la izquierda del primer elemento.

Si el nuevo elemento que se intenta insertar ya pertenece al DOM, el elemento no puede ser insertado nuevamente y el predicado falla. En cualquier otro caso, el elemento es insertado y el predicado se satisface.

Es posible consultar o modificar el contenido, los atributos y los estilos de un objeto HTML mediante los sigueintes predicados:

* [get_attr/3](http://tau-prolog.org/documentation/prolog/dom/get_attr/3): recibe un objeto HTML y un átomo representando un atributo, y devuelve el valor del atributo para dicho elemento.
* [set_attr/3](http://tau-prolog.org/documentation/prolog/dom/set_attr/3): recibe un objeto HTML y un átomo representando un atributo, y establece el valor del atributo para dicho elemento.
* [get_html/2](http://tau-prolog.org/documentation/prolog/dom/get_html/2): recibe un objeto HTML, y devuelve el contenido HTML de dicho elemento.
* [set_html/2](http://tau-prolog.org/documentation/prolog/dom/set_html/2): recibe un objeto HTML, y establece el contenido HTML de dicho elemento.
* [get_style/3](http://tau-prolog.org/documentation/prolog/dom/get_style/3): recibe un objeto HTML y un átomo representando una propiedad CSS, y devuelve el valor de la propiedad para dicho elemento.
* [set_style/3](http://tau-prolog.org/documentation/prolog/dom/set_style/3): recibe un objeto HTML y un átomo representando una propiedad CSS, y establece el valor de la propiedad para dicho elemento.
* [add_class/2](http://tau-prolog.org/documentation/prolog/dom/add_class/2): añade la clase especificada al objeto HTML. El predicado tiene éxito aunque el objeto ya tenga dicha clase.
* [remove_class/2](http://tau-prolog.org/documentation/prolog/dom/remove_class/2): elimina la clase especificada del objeto HTML. El predicado tiene éxito aunque el objeto no tenga dicha clase.
* [has_class/2](http://tau-prolog.org/documentation/prolog/dom/has_class/2): tiene éxito cuando el objeto HTML tiene la clase especificada.
		
## Eventos

El módulo`dom` de Tau Prolog también permite manejar dinámicamente la asignación de eventos, para ejecutar un objetivo Prolog ante un determinado evento del navegador. El predicado [bind/4](http://tau-prolog.org/documentation/prolog/dom/bind/4) recibe un objeto HTML, un átomo representando un tipo de evento (`click`,`mouseover`,`mouseout`, etcétera), un evento y un objetivo, y asocia al elemento HTML el objetivo especificado para dicho tipo de evento. El tercer argumento queda instanciado con un nuevo término que representa un evento HTML, del cual es posible extraer información del evento producido mediante el predicado [event_property/3](http://tau-prolog.org/documentation/prolog/dom/event_property/3) *(véase [My little doge](http://tau-prolog.org/examples/my-little-doge) para un ejemplo completo funcional)*.
		
```html
<div id="output"></div>
```

```javascript
var session = pl.create();
session.consult(":- use_module(library(dom)).");

session.query("get_by_id(output, Output), get_by_tag(body, B), bind(B, keypress, Event, ( \
	event_property(Event, key, Key),                                                      \
	html(Output, Key)                                                                     \
)).");
session.answer(); // {Output/&lt;html>(output), Body/&lt;html>(body), Event/&lt;event>(keypress)}</code></pre>
```

En el ejemplo anterior, se ha añadido al cuerpo de la página un evento`keypress` para que cuando se pulse una tecla, se indique en el objeto HTML con identificador `output` la tecla que se ha pulsado. Nótese que el predicado [event_property/3](http://tau-prolog.org/documentation/prolog/dom/event_property/3) y el término que contiene el evento (`Event` en el ejemplo), sólo tienen sentido dentro del objetivo de un evento, ya que no contendrán ninguna información útil hasta que un evento sea capturado. Cada vez que un evento es capturado, Tau Prolog crea automáticamente un nuevo hilo de la sesión que asignó el evento y ejecuta el objetivo (sólo para la primera respuesta).

Los predicados [unbind/2](http://tau-prolog.org/documentation/prolog/dom/unbind/2) y [unbind/3](http://tau-prolog.org/documentation/prolog/dom/unbind/3) permiten eliminar los eventos asociados a un objeto HTML. El predicado[prevent_default/1](http://tau-prolog.org/documentation/prolog/dom/prevent_default/1) permite prevenir el comportamiento por defecto de un evento del navegador (por ejemplo, puede evitar que un formulario sea enviado).

Puede consultarse una lista con los eventos soportados por el navegador en [Event reference | MDN](https://developer.mozilla.org/en-US/docs/Web/Events).
		
## Efectos

Por último, este módulo incluye otros predicados para crear efectos de animación:

* [hide/1](http://tau-prolog.org/documentation/prolog/dom/hide/1): oculta el elemento HTML.
* [show/1](http://tau-prolog.org/documentation/prolog/dom/show/1): muestra el elemento HTML.
* [toggle/1](http://tau-prolog.org/documentation/prolog/dom/toggle/1): oculta el elemento HTML si es visible, o lo muestra en caso contrario.