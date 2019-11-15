# A simple tutorial

## What is Tau Prolog?
Tau Prolog is a Prolog interpreter fully implemented in JavaScript. Tau Prolog is not just an implementation of the logic programming resolution mechanism, but a complete Prolog interpreter. Furthermore, the implementation of this interpreter has been directed by the ISO Prolog Standard *(see [ISO Prolog: A Summary of the Draft Proposed Standard](http://fsl.cs.illinois.edu/images/9/9c/PrologStandard.pdf), [ISO directives, control constructs and builtins](http://www.deransart.fr/prolog/bips.html), [ISO Prolog Conformity Testing](http://www.complang.tuwien.ac.at/ulrich/iso-prolog/conformity_testing))*.

What makes Tau Prolog special regarding other server-side interpreters is its **integration with web pages' elements**. Tau Prolog installation is quick and simple: you only need to add a JavaScript file to the header of the web page in which it's intended to be run. On [Downloads](http://tau-prolog.org/downloads), you can download a single customized file including Tau Prolog's core and the modules you may require.

## Installation
Once the customized Tau Prolog file is downloaded and properly located, using it is just a matter of adding a script tag on the header of a web page.

```html
<script type="text/javascript" src="tau-prolog.js"></script>
```
You can also download the sources separately and add them to the web page one by one. In that case, keep in mind that **the core script must be loaded before any other Tau Prolog script** since the mechanism that allows instantiating modules is defined on the core. Else, the library won't work.

```html
<script type="text/javascript" src="tau-prolog/core.js"></script>
<script type="text/javascript" src="tau-prolog/lists.js"></script>
...
```

## Sessions

All Tau Prolog functionality is embedded in a JavaScript object named `pl`, which is visible in the global scope. This `pl` object is located under the `window` object (`global` in Node.js).

Tau Prolog is session-oriented. A session allows you to analyse and load multiple programs and modules, as well as submit goals and queries. In order to create a new session, the library provides with the `pl.create` function, which returns a `pl.type.Session` object (every prototype implemented on Tau Prolog is defined on `pl.type`).

```javascript
var session = pl.create();
```

This function can receive an optional argument `limit` which limits the number of resolution steps that the interpreter can make. This way, the browser won't crash, something which could be possible if the interpreter took too much time to find an answer or if it got caught in an infinite SLD tree. If, while looking for an answer, the interpreter returns `null`, this means that it has reached that set limit without finding anything. Nevertheless, if you repeat the query, Tau Prolog will keep looking for an answer starting from the last choice point (where the interpreter was when it returned `null`). The default limit is `1000`.
        
## Load programs and modules

With the aim of analysing and loading programs in a session, the `pl.type.Session` prototype includes a `consult` method, which receives the program as a string and, if it is correct, adds the rules on it to the database, returning `true` afterwards.

```javascript
var parsed = session.consult("                   \
    % load lists module                          \
    :- use_module(library(lists)).               \
                                                 \
    % fruit/1                                    \
    fruit(apple). fruit(pear). fruit(banana).    \
                                                 \
    % fruits_in/2                                \
    fruits_in(Xs, X) :- member(X, Xs), fruit(X). \
"); // true
```
Now, after parsing that program, `session` contains three facts defining the `fruit/1` predicate and a rule defining the `fruits_in/2` predicate. `member/2` is a predicate from `lists` module, so we need to import it using the `use_module` directive. This way, the predicates implemented on the `lists` module will be available in this session.

Let's suppose that we forgot to write a dot after `fruit(banana)`. In this case, the interpreter would load the `lists` and the first two `fruit/1` facts, but after reaching the third fact it would stop parsing and return an error. To discover if there has been an error while parsing, we must check that the returned value is strictly (`===`, `!==`) distinct from `true`.

```javascript
if( parsed !== true ) {
    console.log( parsed ); // throw(error(syntax_error(line(8), column(1), found(fruits_in), cause('. or expression expected'))))
}
```

Errors are generated as Prolog terms *(see [Prototypes and Prolog objects](http://tau-prolog.org/manual/prototypes-and-prolog-objects) from Tau Prolog manual)*, with information about where has the error been raised (line and column), the found token (if any) and the next expected character.
        
## Queries and goals

We can query the database to check if a goal is true or not. In order to do so, we must first add the said goal to the states pile and, later, check for facts and/or rules which satisfy that goal. The `pl.type.Session` prototype has a `query` method, which receives a goal as a string and, after adding the goal to the states pile, returns (if there were no problems) `true`.

```javascript
var parsed = session.query("fruits_in([carrot, apple, banana, broccoli], X)."); // true
```

Once the goal has been added to the pile, the `answer` method in `pl.type.Session` allows us to look for answers (facts or rules) which make the goal true. Tau Prolog is **asynchronous**, which means that `answer` will not return the results, but call a callback function if it finds something. This feature gives the Prolog predicates the ability to make asynchronous operations, as sleeping the execution or to make Ajax queries. In this case, the callback is given to `answer` as an argument.

```javascript
var callback = console.log;
session.answer( callback ); // {X/apple}
session.answer( callback ); // {X/banana}
session.answer( callback ); // false
```

If an answer is found, this will be returned inside a `pl.type.Substitution` object, where every variable of the goal is linked to a value. The `pl.type.Substitution` prototype includes a `toString` method which returns the substitution as a string in the format `{X/a, Y/b, Z/c, ...}` *(see [Prototypes and Prolog objects](http://tau-prolog.org/manual/prototypes-and-prolog-objects) from Tau Prolog manual)*. Another way to express a substitution as a string is the `format_answer` method in `pl`, which receives a substitution object and returns a string in the format `X = a, Y = b, Z = c, ... ;`, or `true ;` if the `Substitution` object has no variables.

```javascript
var callback = function( answer ) { console.log( pl.format_answer( answer ) ); };
session.answer( callback ); // X = apple ;
session.answer( callback ); // X = banana ;
session.answer( callback ); // false.
```

If the interpreter doesn't find an answer, it will call the callback with either `false` or `null`: `false` if there wasn't an answer in the whole database, `null` if the interpreter hasn't found an answer within the resolution steps limit. If the returned value is `>null` and you try to find an answer again, the interpreter will keep looking for answers from the point where it reached the limit last time.