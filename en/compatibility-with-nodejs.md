# Compatibility with Node.js

Tau Prolog is ready to be used with either Node.js or a browser seamlessly. In this page you'll find how to import the Tau Prolog library as well as the modules in a Node.js app.

**Note**: Keep in mind that Node.js looks for packages in its global repository unless the provided path starts with a dot, which means that the path starts at that folder. Hence, even if you save the Tau Prolog library in the same folder as the script using it, the path to it must be something like `"./tau-prolog.js"`.

## Importing the library

In order to import Tau Prolog, we'll use the Node.js function `require`:

```javascript
const pl = require("tau-prolog");
```

The whole functionality of Tau Prolog is inside a JavaScript object called `pl`. This variable could have any name, but, for the sake of compatibility with the browser version, it is recommended to keep the original name, `pl`. From now on, the library can be used just as it is described in other sections of this manual *(see [A simple tutorial](http://tau-prolog.org/manual/a-simple-tutorial))*, excluding the importation tasks.
	
## Loading modules

Since the Tau Prolog library may not be contained on the `pl` object, importing a module (again with `require`) will return a *loader* function. This function receives a reference to the  library and loads the corresponding module on it:

```javascript
const pl = require("tau-prolog");
const loader = require("tau-prolog/modules/lists.js");
loader(pl);
```

A shortened version of that could be:

```javascript
const pl = require("tau-prolog");
require("tau-prolog/modules/lists.js")(pl);
```

This way, we've loaded all the exported predicates contained in the imported module. In the previous example, the predicates from the `lists` module have been imported. Please acknowledge that the core of the library and the modules must be downloaded and imported separately. For instance, if we wanted to load two modules:

```javascript
const pl = require("tau-prolog");
require("tau-prolog/modules/lists.js")(pl);
require("tau-prolog/modules/random.js")(pl);
```

## Example

The next snippet loads a simple Prolog program which keeps information about products and shops selling them.

```javascript
// Import Tau Prolog core and create a session
const pl = require("tau-prolog");
const session = pl.create(1000);
const show = x => console.log(session.format_answer(x));

// Get Node.js argument: node ./script.js item
const item = process.argv[2];

// Program and goal
const program = `
	% Products
	item(id(1), name(bread)).
	item(id(2), name(water)).
	item(id(3), name(apple)).
	% Shops
	shop(id(1), name(tau), location(spain)).
	shop(id(2), name(swi), location(netherlands)).
	% Stock
	stock(item(1), shop(1), count(23), price(0.33)).
	stock(item(2), shop(1), count(17), price(0.25)).
	stock(item(2), shop(2), count(34), price(0.31)).
	stock(item(3), shop(2), count(15), price(0.45)).
`;
const goal = `
	item(id(ItemID), name(${item})),
	stock(item(ItemID), shop(ShopID), _, price(Price)),
	shop(id(ShopID), name(Shop), _).
`;

// Consult program, query goal, and show answers
session.consult(program, {
	success: function() {
		session.query(goal, {
			success: function() {
				session.answers(show);
			}
		})
	}
});
```

This Node.js script receives a product as an argument and queries the database for shops selling said product and its price. If we run the script with several inputs (where `$` stands for the command line prompt), this is what we get:

```prolog
$ nodejs ./script.js bread
ItemID = 1, ShopID = 1, Price = 0.33, Shop = tau ;
false.

$ nodejs ./script.js water
ItemID = 2, ShopID = 1, Price = 0.25, Shop = tau ;
ItemID = 2, ShopID = 2, Price = 0.31, Shop = swi ;
false.

$ nodejs ./script.js apple
ItemID = 3, ShopID = 2, Price = 0.45, Shop = swi ;
false.

$ nodejs ./script.js milk
false.
```