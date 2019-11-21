# Creando tus propios paquetes

Un paquete es una serie de módulos Prolog que exportan un conjunto de predicados. Tau Prolog ofrece algunos módulos para trabajar con listas, manipular el DOM, obtener estadísticos, etcétera. En esta página se describe cómo crear estos paquetes.

## Estructura de un paquete

Un paquete es un fichero JavaScript que introduce uno o varios módulos. Un módulo se caracteriza por un nombre, un conjunto de predicados, y una lista de predicados a exportar. La estructura es la siguiente:

```javascript
var pl;
(function( pl ) {
	// Nombre del módulo
	var name = "mi_modulo";
	// Objeto con el conjunto de predicados, indexados por indicador (nombre/aridad)
	var predicates = function() {
		return {
			// p/1
			"p/1": [ /** ... */ ],
			// q/2
			"q/2": function(thread, point, atom) { /** ... */ }
		};
	};
	// Lista de predicados que exporta el módulo
	var exports = ["p/1", "q/2" /** , ... */];
	// NO EDITAR
	if( typeof module !== 'undefined' ) {
		module.exports = function(tau_prolog) {
			pl = tau_prolog;
			new pl.type.Module( name, predicates(), exports );
		};
	} else {
		new pl.type.Module( name, predicates(), exports );
	}
})( pl );
```

Como puedes observar en el código anterior, los predicados no se definen directamente como un objeto, sino como una función que devuelve dicho objeto. Esto es debido a que al trabajar con Node.js, el objeto global `pl` que contiene toda la funcionalidad de Tau Prolog no está disponible al momento de declarar los predicados. De esta forma, el módulo podrá importarse tanto en Node.js como en navegador sin ningún problema. Por la misma razón, es necesario declarar (sin asignar ningún valor) la variable `pl` en la primera línea del fichero.

Para cargar el paquete, sólo hay que cargar este fichero en la página o en Node.js como si se tratase de cualquier otro paquete de Tau Prolog, e introducir en el programa la directiva `:- use_module(library(mi_modulo)).` para importar los predicados de un modulo.

## Definición de predicados

Existen dos formas de definir un predicado:

* Como una lista de cláusulas Prolog: `[ /** ... */ ]`
* Como una función JavaScript que recibe el hilo de ejecución, el punto de elección actual, y el átomo seleccionado: `function(thread, point, atom) { /** ... */ }`

### Definición de predicados como cláusulas Prolog

Para definir un predicado como una cláusula Prolog, hay que utilizar la representación interna de los objetos definidos por Tau Prolog. Por ejemplo, el predicado `p/1` formado por la siguiente cláusula, `p(X) :- q(X, X).`, se expresaría de la siguiente forma:

```javascript
"p/1": [
	new pl.type.Rule(
		new pl.type.Term("p", [new pl.type.Var("X")]),
		new pl.type.Term("q", [new pl.type.Var("X"), new pl.type.Var("X")])
	)
]
```

Escribir manualmente un predicado Prolog de esta forma puede resultar algo tedioso, por ello los objetos de Tau Prolog incluyen un método `compile` para generar este código automáticamente. Por lo tanto, para obtener esta representación sólo hay que cargar el programa Prolog en una sesión, e imprimir por la salida estándar el código compilado:

```javascript
var session = pl.create();
session.consult("p(X) :- q(X, X).");
console.log(session.compile());
// {"p/1": [new pl.type.Rule(new pl.type.Term("p", [new pl.type.Var("X")]), new pl.type.Term("q", [new pl.type.Var("X"),new pl.type.Var("X")]))]};
```

### Definición de predicados como funciones JavaScript

A veces es necesario recurrir a JavaScript para implementar alguna funcionalidad, bien porque no es posible hacerlo directamente desde Prolog, o bien por razones de eficiencia. Por ello, un predicado también puede ser definido como una función JavaScript que manipula directamente la pila de puntos de elección del hilo de ejecución. Esta función recibe como parámetros:

* el hilo de ejecución,
* el punto de elección actual,
* y el átomo seleccionado del objetivo actual, esto es, el átomo más a la izquierda de dicho objetivo.

Un esquema frecuente a la hora de implementar estas funciones es obtener los argumentos del átomo seleccionado, comprobar si hay algún error (de instanciación, tipo, domino, etcétera) y, en su caso, manipular la pila de puntos de elección. Por ejemplo, la siguiente función del módulo `random` implementa el predicado `random/3`, que genera un número aleatorio entre dos números dados:

```javascript
"random/3": function( thread, point, atom ) {
	var lower = atom.args[0], upper = atom.args[1], rand = atom.args[2];
	if( pl.type.is_variable( lower ) || pl.type.is_variable( upper ) ) {
		thread.throw_error( pl.error.instantiation( atom.indicator ) );
	} else if( !pl.type.is_number( lower ) ) {
		thread.throw_error( pl.error.type( "number", lower, atom.indicator ) );
	} else if( !pl.type.is_number( upper ) ) {
		thread.throw_error( pl.error.type( "number", upper, atom.indicator ) );
	} else if( !pl.type.is_variable( rand ) && !pl.type.is_number( rand ) ) {
		thread.throw_error( pl.error.type( "number", rand, atom.indicator ) );
	} else {
		if( lower.value < upper.value ) {
			var float = lower.is_float || upper.is_float;
			var gen = lower.value + Math.random() * (upper.value - lower.value);
			if( !float )
				gen = Math.floor( gen );
			thread.prepend( [new pl.type.State(
				point.goal.replace( new pl.type.Term( "=", [rand, new pl.type.Num( gen, float )] ) ),
				point.substitution,
				point 
			)] );
		}
	}
}
```

En la primera línea de esta función, se recogen los argumentos del átomo seleccionado en las variables `lower`, `upper` y `rand`. Después, se comprueba que los dos primeros argumentos no son variables mediante la función `pl.type.is_variable`. Si alguno de los dos es una variable, se lanza un error mediante la función `Thread.prototype.throw_error`, generado por la función `pl.error.instantiation`. Del mismo modo se comprueban a continuación el tipo de los argumentos, y se lanza un error si procede.

Si todo es correcto, se genera un número aleatorio entre los dos números dados y se inserta un nuevo punto de elección. La función `Thread.prototype.prepend` recibe un arreglo de estados y los inserta al principio de la pila de puntos de elección. Un estado recibe tres valores: un objetivo, una sustitución, y la referencia al estado padre. Normalmente, el objetivo del nuevo estado se genera reemplazando el átomo seleccionado en el objetivo actual por otro término, y para ello está la función `Term.prototype.replace`. En lugar de generar el unificador más general y modificar la sustitución directamente, es recomendable reemplazar el átomo seleccionado por un término de la forma `argumento = valor` en el nuevo objetivo.

También es frecuente que cuando un predicado tenga éxito simplemente haya que insertar un estado donde el átomo seleccionado debe ser removido, sin insertar un nuevo término, y sin modificar la sustitución. Por ejemplo, el predicado `maybe/0` del paquete `random`.

```javascript
"maybe/0": function( thread, point, _ ) {
	if( Math.random() < 0.5 ) {
		thread.success( point );
	}
}
```

Para eso se utiliza la función `Thread.prototype.success`, que añade al principio de la pila de puntos de elección un nuevo estado donde el átomo seleccionado es eliminado. Esto mismo se puede lograr insertando un estado con la función `prepend`, reemplazando el átomo seleccionado del objetivo por `null`, y dejando la sustitución sin modificar.

**Nota**: Es importante que las funciones que implementan estos predicados no devuelvan nigún valor. Como veremos más adelante, el hecho de devolver un valor que se evalue a `true` en lugar de `undefined` le indicaría a Tau Prolog que la función es asíncrona. **Para que un predicado NO tenga éxito, basta con no insertar ningún estado nuevo en la pila de puntos de elección.**

## Definición de predicados asíncronos

Algunos predicados implementados como funciones de JavaScript requieren el uso de funciones asíncronas. Por ejemplo, la siguiente función `sleep/1` del módulo `system` de Tau Prolog duerme el hilo de ejecución durante unos segundos.

```javascript
"sleep/1": function( thread, point, atom ) {
	var time = atom.args[0];
	if( pl.type.is_variable( time ) ) {
		thread.throw_error( pl.error.instantiation( thread.level ) );
	} else if( !pl.type.is_integer( time ) ) {
		thread.throw_error( pl.error.type( "integer", time, thread.level ) );
	} else {
		setTimeout( function() {
			thread.success( point );
			thread.again();
		}, time.value );
		return true;
	}
}
```
Si no hay ningún error, el predicado `sleep/1` utiliza la función `setTimeout` para realizar una acción al pasar unos segundos, y devuelve el valor `true` (sólo en el caso de que la función vaya a tener éxito). Esto le indica a Tau Prolog que se ha ejecutado un predicado asíncrono, y que no debe seguir aplicando pasos de resolución, ya que el propio predicado volverá a reactivar el proceso de inferencia en algún momento. En el caso de `sleep/1`, tras pasar unos segundos se inserta un nuevo punto de elección en la pila, y se invoca al método `Thread.prototype.again` para que la inferencia se reanude.