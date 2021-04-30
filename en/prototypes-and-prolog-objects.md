# Prototypes and Prolog objects

In this page, the JavaScript prototypes used for modelling Prolog's elements can be found:

* [Variables, numbers, atoms and terms](#Variables,-numbers,-atoms-and-terms)
* [Substitutions](#Substitutions)
* [Sessions and threads](#Sessions-and-threads)
* [Rules and facts](#Rules-and-facts)
* [Errors](#Errors)

## Variables, numbers, atoms and terms

The `pl.type.Var` prototype is used to represent the logical variables in Prolog. The only argument that the constructor receives is the identifier of the variable as a string.

```javascript
var x = new pl.type.Var("X"); // X
x.id; // "X"
```

The `pl.type.Num` prototype is used to represent numbers in Prolog. The constructor receives two arguments: the number representing the numeric value, and a boolean object which indicates if the number is a real value (`true`) or not (`false`).

```javascript
new pl.type.Num(3, false);      // 3
new pl.type.Num(3);             // 3
new pl.type.Num(3, true);       // 3.0
var pi = new pl.type.Num(3.14); // 3.14
pi.value;    // 3.14
pi.is_float; // true
```

The `pl.type.Term` prototype is used to represent atoms and compound terms in Prolog. The constructor receives a string identifying the term and, if the term is compound, an array of Prolog objects.

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

All of these prototypes share the following methods:

* `toString()`: Returns a string representation of the object.
* `clone()`: Returns a copy of the object.
* `apply(substitution)`: Applies a substitution to the object.
* `unify(object, occurs_check)`: Returns the most general unifier resulting from unifying both objects if it succeeds, or `null` otherwise. The second argument indicates wheter it must do the occurs check.
* `variables()`: Returns a list of the variables contained in the object, if any.
* `rename(thread)`: Renames the object. Returns an object with fresh variables in `thread`.



## Substitutions

The `pl.type.Substitution` prototype is used to represent the substitutions in the resolution process and in the answers. The constructor receives, optionally, a JavaScript object linking variables with objects.

```javascript
// {X/foo, Y/2}
var sub = new pl.type.Substitution({
    "X": new pl.type.Term("foo"),
    "Y": new.pl.type.Num(2)
});
sub.links["X"]; // foo
sub.links["Y"]; // 2
```

## Sessions and threads

The `pl.type.Session` prototype is used to represent sessions. The constructor receives an integer which sets the limit of resolution steps that the interpreter can take when trying to find an answer.

> **Note:** In order to create a new session, the `pl.create` method is provided. It's not recommended to use the prototype directly.

The `pl.type.Thread` prototype is used to represent running threads in a session. The constructor receives the session to which it belongs as an argument. When a session is created, it creates an inner thread by default. The threads of the same session share some information, as the program or the operators table, but elements as the choice points stack are thread-dependent.

## Rules and facts

The `pl.type.Rule` prototype is used to represent the rules of Prolog programs. The constructor receives both the head and the body of a rule as arguments. If the rule is a fact, the empty body is represented with a `null` value.

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

## Errors

ISO Prolog provides with an exception handling mechanism, based on the built-in control constructions [catch/3](http://tau-prolog.org/documentation/prolog/builtin/catch/3) and [throw/1](http://tau-prolog.org/documentation/prolog/builtin/throw/1). When there is an error, the current goal is replaced with a goal of the form `throw(error(Error_term, Implementation_defined_term))`. If the error is not handled, the error parameter in `throw/1` is returned.