# Promises interface

A promise object represents the eventual completion or failure of an asynchronous operation and its resulting value. Tau Prolog's `promises` package extends the `pl.type.Session` and `pl.type.Thread` prototypes to add new methods for consulting programs and querying goals, returning promises. 

## Tau Prolog using promises 

The `promises` package adds three new methods:

* `promiseConsult`: consults a program and returns a promise that is resolved when the program loads successfully, or rejected when there is an error. It takes the same arguments as the `consult` method.
* `promiseQuery`: queries a goal and returns a promise that is resolved when the goal loads successfully, or rejected when there is an error. It takes the same arguments as the `query` method.
* `promiseAnswer`: finds the next computed answer and returns a promise that is resolved when it finds an answer or there are no more answers, or is rejected when there is an error or the limit of inferences has been reached. It takes the same arguments as the `answer` method.

Also, the package adds a fourth method, `promiseAsnwers`, to find all computed answers, returning an asynchronous generator.

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

## Tau Prolog using Observables

Note that it is possible to use the promises interface together with reactive programming libraries, if they support the conversion of asynchronous generator functions to observables, such as `RxJS` from version `7.0.0`.

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