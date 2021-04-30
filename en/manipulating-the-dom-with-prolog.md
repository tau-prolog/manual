# Manipulating the DOM with Prolog

Tau Prolog's `dom` module adds new term types and predicates that allow the user to modify the DOM and to handle browser events. You can find more information about these new predicates in the [dom module reference predicates](http://tau-prolog.org/documentation#dom).

## Selectors

Tau Prolog includes three non-deterministic predicates to look for DOM elements:

* [get_by_id/2](http://www.tau-prolog.org/documentation/prolog/dom/get_by_id/2): look for the DOM element with the specified id.
* [get_by_class/2](http://www.tau-prolog.org/documentation/prolog/dom/get_by_class/2): look for all the elements (by reevaluation) which have the specified class.
* [get_by_tag/2](http://www.tau-prolog.org/documentation/prolog/dom/get_by_tag/2): look for all the elements (by reevaluation) with the specified tag.

If there are no elements in the DOM with the specified identifier, class or tag, the predicates will fail.

```html
<div id="block-a" class="circle">content a</div>
<div id="block-b" class="circle">content b</div>
<div id="block-c" class="square">content c</div>
```

```javascript
var session = pl.create();
session.consult(":- use_module(library(dom)).", {
	success: function() {
		session.query("get_by_tag(div, B), html(B, X).", {
			success: function() {
				session.answer(console.log); // {B/<html>(block-a), X/'content a'}
				session.answer(console.log); // {B/<html>(block-b), X/'content b'}
				session.answer(console.log); // {B/<html>(block-c), X/'content c'}
				session.answer(console.log); // false
			}
		});
	}
});
```

```javascript
var session = pl.create();
session.consult(":- use_module(library(dom)).", {
	success: function() {
		session.query("get_by_class(circle, B), html(B, X).", {
			success: function() {
				session.answer(console.log); // {B/<html>(block-a), X/'content a'}
				session.answer(console.log); // {B/<html>(block-b), X/'content b'}
				session.answer(console.log); // false
			}
		});
	}
});
```

The `dom` module also includes predicates to go through the DOM starting at the HTML objects retrieved with the previous predicates.

* [parent_of/2](http://www.tau-prolog.org/documentation/prolog/dom/parent_of/2): If the first argument is bound but the second one is not, the second argument will be bound to the first element's parent. If the second argument is bound but the first one is not, it will be bound (by reevaluation) to all the children of second argument. If both arguments are bound, the predicate checks if the first parameter is a children of the second one.
* [sibling/2](http://www.tau-prolog.org/documentation/prolog/dom/sibling/2): if the first argument is bound but the second one is not, it will be bound to the first argument's most immediate right sibling. If the first argument is unbound but the second one is not, the first argument will be bound to the second argument's most immediate left sibling. If both arguments are bound, the predicate checks if they are both immediate siblings.
		
## Modifying the DOM
		
The [create/2](http://www.tau-prolog.org/documentation/prolog/dom/create/2) method recives an atom as a representation of a HTML tag (`div`, `a`, `table`, etc.) and created a new HTML object. Newly created HTML objects can be inserted in the DOM using the next predicates:

* [append_child/2](http://www.tau-prolog.org/documentation/prolog/dom/append_child/2): inserts the second argument as the last child of the first one.
* [insert_after/2](http://www.tau-prolog.org/documentation/prolog/dom/insert_after/2): inserts the second argument on the same level as the first one, as its most immediate right sibling.
* [insert_before/2](http://www.tau-prolog.org/documentation/prolog/dom/insert_before/2): inserts the second argument on the same level as the first one, as its most immediate left sibling.

If we try to insert an element which is already part of the DOM, the predicate fails and the element is not inserted again. In any other case, the element in inserted and the predicate is satisfied.

These predicates allow you to inquire or modify the content, the attributes or the styles of an HTML object:

* [get_attr/3](http://www.tau-prolog.org/documentation/prolog/dom/get_attr/3): receives an HTML object, and an atom standing for an attribute, and returns the value of said attribute in that HTML object.
* [set_attr/3](http://www.tau-prolog.org/documentation/prolog/dom/set_attr/3): receives an HTML object and an atom standing for an attribute, and sets the value of said attribute in that HTML object.
* [get_html/2](http://www.tau-prolog.org/documentation/prolog/dom/get_html/2): receives an HTML object, and returns the HTML contained in that HTML object.
* [set_html/2](http://www.tau-prolog.org/documentation/prolog/dom/set_html/): receives an HTML object, and sets the HTML content of that HTML object.
* [get_style/3](http://www.tau-prolog.org/documentation/prolog/dom/get_style/3): receives an HTML object, and an atom standing for a CSS property, and returns the value of said property in that HTML object.
* [set_style/3](http://www.tau-prolog.org/documentation/prolog/dom/set_style/3): receives an HTML object, and an atom standing for a CSS property, and sets the value of said property in that HTML object.
* [add_class/2](http://www.tau-prolog.org/documentation/prolog/dom/add_class/2): adds a class to an HTML object. The predicate succeeds even if the object had that class already.
* [remove_class/2](http://www.tau-prolog.org/documentation/prolog/dom/remove_class/2): removes the specified class from an HTML object. The predicate succeeds even if the object didn't have that class.
* [has_class/2](http://www.tau-prolog.org/documentation/prolog/dom/has_class/2): this predicate succeeds when the HTML object has the specified class.
		
## Events

Tau Prolog's `dom` module also enables the dynamic assignation of events, in order to run a Prolog goal when some event is triggered. The [bind/4](http://www.tau-prolog.org/documentation/prolog/dom/bind/4) method receives an HTML object, an atom representing an event type (`click`, `mouseover`, `mouseout`, etc), an event and a goal, and bind the HTML object with said goal for that type of event. The third argument is bound to a new term that represents an HTML event, from which we can read information using the [event_property/3](http://www.tau-prolog.org/documentation/prolog/dom/event_property/3) predicate *(see [My little doge](http://tau-prolog.org/examples/my-little-doge)</a> for a complete, functional example)*.

```html
<pre><code class="html"><div id="output"></div>
```

```javascript
var session = pl.create();
session.consult(":- use_module(library(dom)).", {
	success: function() {
		session.query("get_by_id(output, Output), get_by_tag(body, B), bind(B, keypress, Event, ( \
			event_property(Event, key, Key),                                                      \
			html(Output, Key)                                                                     \
		)).", {
			success: function() {
				session.answer(console.log); // {Output/<html>(output), Body/<html>(body), Event/<event>(keypress)}
			}
		});
	}
});
```

In this example, the `keypress` event has been added to the page body, so when a key is pressed, the HTML object whose id is `output` shows what key has been pressed. Notice that the [event_property/3](http://www.tau-prolog.org/documentation/prolog/dom/event_property/3) predicate and the `Event` term only make sense inside an event's goal, since they don't hold any information until the event is triggered. Any time an event is triggered, Tau Prolog creates a new thread in the session that assigned the event and runs the goal (just for the first answer).

The [unbind/2](http://www.tau-prolog.org/documentation/prolog/dom/unbind/2) and [unbind/3](http://www.tau-prolog.org/documentation/prolog/dom/unbind/3) predicates allow us to remove the events attached to an HTML object. The [prevent_default/1](http://www.tau-prolog.org/documentation/prolog/dom/prevent_default/1) predicate allows us to prevent the browser default behaviour regarding an event (for instance, to keep a form from being sent). A list with the events supported by the browser can be read in [Event reference | MDN](https://developer.mozilla.org/en-US/docs/Web/Events).</p>
		
## Effects

Lastly, this module also includes predicates to create animations:

* [hide/1](http://www.tau-prolog.org/documentation/prolog/dom/hide/1): hide the HTML object.
* [show/1](http://www.tau-prolog.org/documentation/prolog/dom/show/1): show the HTML object.
* [toggle/1](http://www.tau-prolog.org/documentation/prolog/dom/toggle/1): hide the HTML object if it's visible; show it otherwise.