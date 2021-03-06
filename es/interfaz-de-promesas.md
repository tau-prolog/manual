# Interfaz de promesas

Una promesa es un objeto que representa la terminación o el fracaso de una operación asíncrona. El paquete `promises` de Tau Prolog extiende los prototipos de `pl.type.Session` y `pl.type.Thread` para añadir nuevos métodos para cargar programas y consultar objetivos, devolviendo promesas.

## Tau Prolog utilizando promesas

El paquete `promises` añade tres nuevos méotodos:

* `promiseConsult`: carga un programa y devuelve una promesa que se resuelve cuando el programa se carga correctamente, o se rechaza cuando hay un error. Recibe los mismos parámetros que el método `consult`.
* `promiseQuery`: consulta un objetivo y devuelve una promesa que se resuelve cuando el objetivo se carga correctamente, o se rechaza cuando hay un error. Recibe los mismos parámetros que el método `query`.
* `promiseAnswer`: busca la siguiente respuesta computada y devuelve una promesa que se resuelve cuando encuentra una respuesta o no hay más respuestas, o se rechaza cuando hay un error o se ha llegado al límite de inferencias. Recibe los mismos parámetros que el método `answer`.

Además, añade un cuarto método, `promiseAsnwers`, para buscar todas las respuestas computadas, devolviendo un generador asíncrono.

```javascript
const pl = require("tau-prolog");
require("tau-prolog/modules/promises.js")(pl);

(async() => {

    const program = `
        plus(z, Y, Y).
        plus(s(X), Y, s(Z)) :- plus(X, Y, Z).
    `;
    const goal = "plus(X, Y, s(s(s(z)))).";
    const session = pl.create();
    await session.promiseConsult(program);
    await session.promiseQuery(goal);

    for await (let answer of session.promiseAnswers())
        console.log(session.format_answer(answer));
    // X = z, Y = s(s(s(z))) ;
    // X = s(z), Y = s(s(z)) ;
    // X = s(s(z)), Y = s(z) ;
    // X = s(s(s(z))), Y = z.

})();
```

## Tau Prolog utilizando Observables

Nótese que es posible utilizar la interfaz de promesas junto a librerías de programación reactiva, si soportan la conversión de generadores asíncronos a observables, como `RxJS` a partir de la versión `7.0.0`.

```javascript
const Rx = require("rxjs");
const pl = require("tau-prolog");
require("tau-prolog/modules/promises.js")(pl);

(async() => {

    const program = `
        plus(z, Y, Y).
        plus(s(X), Y, s(Z)) :- plus(X, Y, Z).
    `;
    const goal = "plus(X, Y, s(s(s(z)))).";
    const session = pl.create();
    await session.promiseConsult(program);
    await session.promiseQuery(goal);

    const plus = Rx.from(session.promiseAnswers());
    plus.subscribe(x => console.log(session.format_answer(x)));
    // X = z, Y = s(s(s(z))) ;
    // X = s(z), Y = s(s(z)) ;
    // X = s(s(z)), Y = s(z) ;
    // X = s(s(s(z))), Y = z.

})();
```