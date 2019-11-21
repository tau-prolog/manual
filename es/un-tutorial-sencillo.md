# Un tutorial sencillo

## ¿Qué es Tau Prolog?

Tau Prolog es un intérprete de Prolog implementado completamente en JavaScript. No es simplemente una implementación del mecanismo de resolución de la lógica de predicados, sino una implementación de Prolog guiada por el estándar ISO Prolog *(véase [ISO Prolog: A Summary of the Draft Proposed Standard](http://fsl.cs.illinois.edu/images/9/9c/PrologStandard.pdf), [ISO directives, control constructs and builtins](http://www.deransart.fr/prolog/bips.html), [ISO Prolog Conformity Testing](http://www.complang.tuwien.ac.at/ulrich/iso-prolog/conformity_testing))*.

Lo que distingue a Tau Prolog de otros intérpretes ejecutados en el lado del servidor, es su capacidad de integración e interacción con los elementos de las páginas web. La instalación de Tau Prolog es muy sencilla, sólo hay que incluir un fichero JavaScript en la cabecera de la página web donde se pretende ejecutar. En la sección [Descargas](http://tau-prolog.org/downloads) es posible descargar un único fichero personalizado que incluya el núcleo del intérprete junto a los módulos requeridos.
		
## Instalación

Una vez descargado y alojado el correspondiente fichero de Tau Prolog, simplemente hay que insertarlo en la cabecera de la página web.

```html
<script type="text/javascript" src="tau-prolog.js"></script>
```

También es posible descargar los ficheros fuente por separado e insertarlos individualmente en la página web. En ese caso, es importante cargar primero el núcleo de la biblioteca antes que cualquier otro módulo.

```html
<script type="text/javascript" src="tau-prolog/core.js"></script>
<script type="text/javascript" src="tau-prolog/lists.js"></script>
...
```

## Sesiones

Todos los métodos de Tau Prolog están contenidos en un objeto JavaScript llamado `pl`, que es visible en el ámbito global bajo el objeto `window` (`global` en Node.js).

El uso de Tau Prolog está orientado a la manipulación de sesiones. Una sesión permite analizar y cargar múltiples programas y módulos, así como lanzar objetivos. Para crear una nueva sesión se provee de la función `pl.create`, que devuelve un objeto `pl.type.Session` (todos los prototipos implementados por Tau Prolog están definidos en `pl.type`).

```javascript
var session = pl.create();
```

Esta función acepta un parámetro opcional `limit`, que indica el número máximo de pasos de resolución que puede dar el intérprete para encontrar una respuesta. Esto evita que el navegador se bloquee, ya sea porque el intérprete tarde mucho en encontrar una respuesta, o porque haya entrado en una rama infinita. Si al buscar una respuesta el intérprete devuelve `null`, indica que se han ejecutado el límite establecido de pasos de resolución sin hallar ninguna respuesta; no obstante, si se vuelve a solicitar una respuesta Tau Prolog seguirá buscando desde el último punto de elección. El valor por defecto es `1000`.
		
## Cargar programas y módulos

Para analizar y cargar programas en una sesión, el prototipo `pl.type.Session` disponde del método `consult`, que recibe un programa en forma de cadena de caracteres y, si todo va bien, añade las reglas analizadas a la base de datos de la sesión y devuelve `true`.

```javascript
var parsed = session.consult("                   \
	% cargar módulo lists                        \
	:- use_module(library(lists)).               \
	                                             \
	% fruit/1                                    \
	fruit(apple). fruit(pear). fruit(banana).    \
	                                             \
	% fruits_in/2                                \
	fruits_in(Xs, X) :- member(X, Xs), fruit(X). \
"); // true
```

Ahora `session` contiene tres hechos que definen el predicado `fruit/1` y una regla que define el predicado `fruits_in/2`. El predicado `member/2` forma parte del módulo `lists` de Tau prolog, así que es necesario importar dicho módulo mediante la directiva `use_module` para incluir sus predicados en la sesión.

Supongamos ahora que al escribir este último programa, se nos ha olvidado poner un punto tras el hecho `fruit(banana)`. El intérprete habría cargado el módulo `lists` y habría analizado correctamente los dos primeros hechos de `fruit/1`, pero en el tercero dejaría de analizar y devolvería un error. Para saber si se ha producido un error, hay que comprobar estrictamente (`===`, `!==`) que el valor devuelto sea distinto de `true`.

```javascript
if( parsed !== true ) {
	console.log( parsed ); // throw(error(syntax_error(line(8), column(1), found(fruits_in), cause('. or expression expected'))))
}
```

Los errores se devuelven en formato de término Prolog *(véase [Prototipos y objetos Prolog](http;//tau-prolog.org/manual/es/prototipos-y-objetos-prolog)</a> del manual de Tau Prolog)*, con información acerca de dónde se ha producido el error, esto es la línea y la columna, el token encontrado (si existe) y el siguiente carácter esperado.
		
## Consultar objetivos

De forma análoga a la carga de programas, para consultar objetivos en una sesión el prototipo `pl.type.Session` disponde del método `query`, que recibe un objetivo en forma de cadena de caracteres y, si todo va bien, añade el objetivo a la pila de estados de la sesión y devuelve `true`.

```javascript
var parsed = session.query("fruits_in([carrot, apple, banana, broccoli], X)."); // true
```

Una vez añadido el objetivo, el método `answer` de `pl.type.Session` permite buscar las respuestas computadas. Tau Prolog es un intérprete asíncrono, por lo tanto `answer` no devuelve ningún resultado, sino que ejecuta una función a modo de callback. Esta asincronía permite que los predicados Prolog realicen operaciones asíncronas, como dormir la ejecución un cierto tiempo o hacer peticiones Ajax.

```javascript
var callback = console.log;
session.answer( callback ); // {X/apple}
session.answer( callback ); // {X/banana}
session.answer( callback ); // false
```

Si se encuentra una respuesta computada, esta se devuelve en un objeto del prototipo `pl.type.Substitution`, donde cada variable del objetivo se liga con un valor. Este prototipo implementa el método `toString` para obtener una representación textual de la substitución, de la forma `{X/a, Y/b, Z/c, ...}` *(véase [Prototipos y objetos Prolog](http://tau-prolog.org/manual/es/prototipos-y-objetos-prolog#substituciones) del manual de Tau Prolog)*. Si se prefiere una representación más amigable, el objeto `pl` dispone del método `format_answer`, que recibe una substitución y devuelve una representación textual de la forma `X = a, Y = b, Z = c, ... ;` o `true ;` cuando la substitución no contiene variables.

```javascript
var callback = function( answer ) { console.log( pl.format_answer( answer ) ); };
session.answer( callback ); // X = apple ;
session.answer( callback ); // X = banana ;
session.answer( callback ); // false.
```

Si no se encuentra ninguna respuesta, el intérprete ejecuta el callback con el valor `false`. De igual forma con `null` cuando se alcanza el límite de pasos de resolución.