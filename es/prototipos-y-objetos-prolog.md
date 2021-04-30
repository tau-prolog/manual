# Prototipos y objetos Prolog

En esta página se describen los prototipos implementados para modelar los elementos de Prolog y sus métodos disponibles:

* [Variables, números, átomos y términos](#Variables,-números,-átomos-y-términos)
* [Substituciones](#Substituciones)
* [Sesiones e hilos](#Sesiones-e-hilos)
* [Reglas y hechos](#Reglas-y-hechos)
* [Errores](#Errores)

## Variables, números, átomos y términos

El prototipo `pl.type.Var` se utiliza para representar las variables lógicas de los programas Prolog. El constructor recibe como argumento una cadena de carácteres que representa el identificador de la variable.

```javascript
var x = new pl.type.Var("X"); // X
x.id; // "X"
```

El prototipo `pl.type.Num` se utiliza para representar los números de los programas Prolog. El constructor recibe como argumentos un número que representa el valor y, opcionalmente, un valor lógico que indica si se trata de un valor real (`true`) o entero (`false`).

```javascript
new pl.type.Num(3, false);      // 3
new pl.type.Num(3);             // 3
new pl.type.Num(3, true);       // 3.0
var pi = new pl.type.Num(3.14); // 3.14
pi.value;    // 3.14
pi.is_float; // true
```

El prototipo `pl.type.Term` se utiliza para representar los átomos y términos compuestos de los programas Prolog. El constructor recibe como argumentos una cadena de carácteres que representa el identificador del término y, opcionalmente, un array de objetos Prolog que representan los parámetros del término (si es compuesto).

```javascript
// foo
new pl.type.Term("foo");
// foo  
new pl.type.Term("foo", []);
// [1,2,3]
new pl.type.Term(".", [
    new pl.type.Num(1),
    new pl.type.Term(".", [
        new pl.type.Num(2),
        new pl.type.Term(".", [
            new pl.type.Num(3),
            new pl.type.Term("[]")])])]);
// foo(bar, 0, X)
var term = new pl.type.Term("foo", [    
    new pl.type.Term("bar"),
    new pl.type.Num(0, false),
    new pl.type.Var("X")]);
term.id;      // "foo"
term.args;    // Array(3) [ {…}, {…}, {…} ]
term.args[0]; // bar 
```

Todos estos prototipos comparten los siguientes métodos:

* `toString()`: Devuelve una representación textual del objeto.
* `clone()`: Devuelve una copia del objeto.
* `apply(substitution)`: Aplica una substitución al objeto.
* `unify(object, occurs_check)`: Devuelve el unificador más general resultante de unificar los dos objetos. Devuelve `null` si la unificación falla. El segundo argumento especifica si se debe realizar la ocurrencia de variables o no.
* `variables()`: Devuelve la lista de variables contenidas en el objeto.
* `rename(thread)`: Renombra las variables del objeto. Devuelve un término con variables frescas para en `thread`.



## Substituciones

El prototipo `pl.type.Substitution` se utiliza para representar las substituciones en el proceso de resolución, así como en respuesta a las consultas. El constructor recibe como argumento, opcionalmente, un objeto JavaScript que relaciona las variables con objetos.

```javascript
// {X/foo, Y/2}
var sub = new pl.type.Substitution({
    "X": new pl.type.Term("foo"),
    "Y": new.pl.type.Num(2)
});
sub.links["X"]; // foo
sub.links["Y"]; // 2
```

## Sesiones e hilos

El prototipo `pl.type.Session` se utiliza para representar las sesiones. El constructor recibe como argumento un entero que representa el límite de pasos de resolución que se pueden dar antes de encontrar una respuesta.

> **Nota:** Para crear una nueva sesión se provee del método `pl.create`, no se recomienda utilizar el prototipo directamente.

El prototipo `pl.type.Thread` se utiliza para representar hilos de ejecución en las sesiones. El constructor recibe como argumento la sesión a la que pertenece. Cuando se crea una nueva sesión, esta crea uno hilo de ejecución por defecto en la misma. Los hilos de la misma sesión comparten cierta información, como las reglas analizadas o la tabla de operadores, pero otra información, como los objetivos o la pila de puntos de elección son independientes entre ellos.

## Reglas y hechos

El prototipo `pl.type.Rule` se utiliza para representar las reglas de los programas Prolog. El constructor recibe como argumentos la cabeza y el cuerpo de la regla. Si la regla es un hecho, el cuerpo vacío se representa con el valor `null`.

```javascript
var x = new pl.type.Var("X");

// p(foo).
var fact = new pl.type.Rule(
    new pl.type.Term("p", [
        new pl.type.Term("foo")]),
    null);
fact.head; // p(foo)
fact.body; // null

// p(X) :- q(X).
var rule = new pl.type.Rule(
    new pl.type.Term("p", [x]),
    new pl.type.Term("q", [x]));
rule.head; // p(X)
rule.body; // q(X)

// p(X) :- q(X), r(X).
var rule = new pl.type.Rule(
    new pl.type.Term("p", [x]),
    new pl.type.Term(",", [
        new pl.type.Term("q", [x]),
        new pl.type.Term("r", [x])]));
rule.head; // p(X)
rule.body; // q(X), r(X)
```

## Errores

ISO Prolog proporciona un mecanismo de manejo de excepciones, basado en las construcciones de control incorporadas [catch/3](http://tau-prolog.org/documentation/prolog/builtin/catch/3) y [throw/1](http://tau-prolog.org/documentation/prolog/builtin/throw/1). Cuando ocurre un error, el objetivo actual es reemplazado por un objetivo de la forma `throw(error(Error_term, Implementation_defined_term))`. Si el error no es tratado, se obtiene como respuesta el error contenido como parámetro del término `throw/1`.