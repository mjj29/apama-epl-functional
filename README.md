# EPL Functional
Apama EPL Classes for functional operations

## Supported Apama version

This works with Apama 10.5 or later, and probably most of the earlier ones as well, although only later versions support lambdas. With previous versions you need to define explicit actions to use instead.

## Functional operators

This library provides a selection of functional operations modelled similar to Python's `functools` or `itertools` libraries, for example filter, map and reduce. These operate on EPL container types (`sequence` and `dictionary`) and on generators provided by this library (see below). To help using these functional operators, there are also several functors and predicates provided within the library.

There are two APIs for accessing the functional operators. Firstly, all of the operators are provided as static functions on the [`com.apamax.functional.Fn`](https://mjj29.github.io/apama-epl-functional/com/apamax/functional/Fn.html) type. Each of these functions takes a container as its first argument and returns a new container with the new contents as the result, in each case using an `any` as the type. For example, to filter a sequence of numbers for just even numbers:

	sequence<integer> evens := <sequence<integer>> Fn.filter(numbers, Fn.even);

This example also shows the use of one of the functors, also provided on the `Fn` event. You can use an action or action variable with the signature `action<integer> returns boolean`, or (in later versions of Apama) a matching lambda. Some of these are provided within the library, but you can also write your own. You can combine several of these operations into a pipeline:

	integer evenSum := <integer> Fn.reduce(Fn.filter(numbers, Fn.even), Fn.sum);

This will return the sum of all the even numbers within the `numbers` container. The reduce function takes an additional first argument of the current value of the accumulator and returns the new value of the accumulator, so in this case the signature would be `action<integer, integer> returns integer`.

If you are operating on a dictionary instead of a sequence, then you can use functions with one of two signature types. `action<VALUETYPE> returns RETURNTYPE` signatures will be invoked with each value in turn (ignoring the keys). `action<KEYTYPE, VALUETYPE> returns RETURNTYPE` signatures will be passed the key and the value in turn. 

The second API is using the [`com.apamax.functional.Functional`](https://mjj29.github.io/apama-epl-functional/com/apamax/functional/Functional.html) type. This wraps your container and then provides the functional operators as instance methods, each one returning a new `Functional` object. At the end of the chain you can either use an operator which directly returns a value, like `reduce`, or you can call `get` to return the underlying result object. For example:

	sequence<integer> evens := <sequence<integer>> Functional(numbers).filter(Fn.even).get();
	integer evenSum := <integer> Functional(numbers).filter(Fn.even).reduce(Fn.sum);

`Functional` wraps all of the operators provided as static functions on `Fn`. As you can see, you still use `Fn` to access the predicates and functors to use with the operators.

Here is a list of the operators provided on `Fn` and `Functional`.

| Operator | Arguments | Return | Description |
| -------- | --------- | ------ | ----------- |
| filter | sequence, dictionary or generator<br/>`action<TYPE> returns boolean` or `action<KEY, VALUE> returns boolean`|Fn: A new container of the same type as the input container<br/>Functional: a new Functional|Filters the container to only have the elements where the provided predicate is true. |
| map | sequence, dictionary or generator<br/>`action<TYPE> returns NEWTYPE` or `action<KEY, VALUE> returns NEWTYPE`|Fn: A new container of the same sort as the input container, but the type returned from the functor<br/>Functional: a new Functional|Uses the functor to replace all the values in the container with new values. Keys in dictionaries are unchanged. |
| reduce | sequence or dictionary<br/>`action<RESULT, TYPE> returns RESULT` or `action<RESULT, KEY, VALUE> returns RESULT`|The result of calling the functor across all of the values|Repeatedly calls a functor on each value, using the output of functor to update an accumulator passed to the next call and returning the final result. The first call will be passed a default-initialized RESULT type.|
| reduceFrom | sequence or dictionary<br/>Initial value for the reduction<br/>`action<RESULT, TYPE> returns RESULT` or `action<RESULT, KEY, VALUE> returns RESULT`|The result of calling the functor across all of the values|Repeatedly calls a functor on each value, using the output of functor to update an accumulator passed to the next call and returning the final result. The first call will be passed the initial value.|
| accumulate | sequence, dictionary or generator<br/>`action<RESULT, TYPE> returns RESULT` or `action<RESULT, KEY, VALUE> returns RESULT`|Fn:A generator which will iterate over the results<br/>Functional:A new Functional|Repeatedly calls a functor on each value, using the output of functor to update an accumulator passed to the next call and returning each result in turn. The first call will be passed a default-initialized RESULT type.|
| accumulateFrom | sequence, dictionary or generator<br/>Initial value for the accumulation<br/>`action<RESULT, TYPE> returns RESULT` or `action<RESULT, KEY, VALUE> returns RESULT`| Fn:A generator which will iterate over the result<br/>Functional:A new Functional|Repeatedly calls a functor on each value, using the output of functor to update an accumulator passed to the next call and returning each result in turn. The first call will be passed a default-initialized RESULT type. The first call will be passed the initial value.|
| argmap | sequence or generator<br/>`action<TYPE...> returns NEWTYPE`|Fn: A new container of the same sort as the input container, but the type returned from the functor<br/>Functional: a new Functional|Each item in the input is treated as an argument, or sequence or arguments, for the functor. The result will be a container with the results of calling the functor on those arguments|
| slice | sequence or generator<br/>Start offset (0+)<br/>End offset(0+, or -1 for the whole sequence)<br/>Distance to increment each time(1+)|Fn:A sequence containing the selected elements<br/>Functional: a new Functional|Selects a subset of a sequence, or generator. Immediately consumes enough of the generator to create a concrete sequence.|
| consume | A generator<br/>The number of times to step it. | The generator stepped n times.|Steps a generator the given number of times, discarding the results.|
| quantify | sequence or dictionary<br/>`action<TYPE> returns boolean`|The number of items in the sequence or dictionary for which the predicate returns true.|Runs a predicate on each item in the container and counts how many times it returns true.|

`Fn` also provides some predicates to use with `filter`:

| Predicate | Description |
|---|---|
| \_not | Inverts another predicate.<br/>eg: `Fn.filter(numbers, Fn._not(Fn.even))` |
| istrue | True if a boolean is true |
| even | True if an integer is even |
| odd | True if an integer is odd | 
| negative | True if an integer, float or decimal is less than 0 |
| positive | True if an integer, float or decimal is greater than 0<br/>For including 0 use `Fn._not(Fn.negative)`|
| whole | True if a float or decimal does not have a fractional part.<br/>Always true for integers |
| \_any | True if a container of booleans contains at least one True<br/>False for the empty container |
| \_all | True if a container of booleans contains no False<br/>True for the empty container |

`Fn` also provides some functors to use with `map`, `reduce` and `accumulate`:

| Functor | Operand type(s) |Description |
|---|---|---|
| increment | integer | Increments to the next integer |
| sum | integer, float or decimal | Add up all the values |
| mul | integer, float or decimal | Calculate the product of the values |
| concat | string | Concatenates all the strings |
| callAction | any | Calls the named function with the given args on a value. Must be used with `Fn.partial` to provide function name and args. |
| getEntry | any | Returns the named field within a value. |

## Functional listeners

`Fn` provides some actions which interact with events and listeners. These allow you to use a functional style of code to also listen for events.

For examples of how you might use these where you might have wanted to write:

	on all Event(f="val1") as e or all Event(f="val2") as e or ... { eventArrived(e); }

However, you have a variable number of possible values, and they aren't in a contiguous range. With `Fn` you can write:

	sequence<listeners> ls := Fn.listenForAnyOf(["val1", "val2"], "Event", "f", {}, eventArrived);
	on Stop() {
		any _ := Fn.map(ls, Fn.partial(callAction, "quit", []));
	}

Another common pattern is having an asynchronous process with a completed event. You have a similar issue listening to a variable number of processes. With `Fn` you can now write this:

	on Completed(id=1) and Completed(id=2) and ... and not wait(TIMEOUTSECS) { onCompleted(); }
	on wait(TIMEOUTSECS) and not (on Completed(id=1) and Completed(id=2) and ... ) { onTimeout(); }

as:

	Fn.waitForAllCompleted(sequenceIDs, "Completed", "id", TIMEOUTSECS, onCompleted, onTimeout);

Lastly we have wanting to receive all events up until a termination condition and then processing them as a collection. Rather than accumulating them all in a container manually with multiple listeners like this:

	sequence<ValueEventName> vals := new sequence<ValueEventName>;
	on all ValueEventName(fields=valueEventFields) as v and not EndEventName(fields=endEventFields) or wait(timeout) { vals.append(v); }
	on EndEventName(fields=endEventFields) and not wait(timeout) { onComplete(vals); }
	on wait(timeout) and not EndEventName(fields=endEventFields) { onTimeout(vals); }

Instead you can write:

	Fn.getAllEvents("ValueEventName", {...}, "EndEventName", {...}, TIMEOUT, onComplete, onTimeout);

Here is a list of the event/listener actions on `Fn`.

| Action | Arguments | Return | Description |
| ------ | --------- | ------ | ----------- |
| listenForAnyOf | Sequence of values<br/>Event type and field name<br/>Additional fields<br/>`action<Eventtype>` | `sequence<listener>` | Create multiple listeners one for each value in the sequence. Call the given action for each matching event which arrives |
| waitForAllCompleted | Sequence of values<br/>Event type and field name<br/>timeout<br/>`action<>` on success and `action<sequence<any>>` on timeout | nothing | Take a list of values, wait for an event with each value to be received within a timeout. Call a success or timeout action |
| getAllEvents | Event type name<br/>Dictionary of event fields<br/>Event type name<br/>Dictionary of event fields<br/>timeout<br/>`action<sequence<any>>` on success and `action<sequence<any>>` on timeout| nothing| Wait for all events of the first type and arguments until receiving an event of the second type and arguments, then call a method with a sequence of all received events. |

## Generators

The functional library also provides a concept of generators. Generators are objects which lazily calculate an infinite list. The simplest form of a generator is a current value and a functor which takes the previous value and calculates the next value. To get the next value you call the `generate` function, which steps the generator and returns the next value. You can use most of the functional operators above and they will return another generator which lazily evaluates the function each time you step the resulting generator. To create a generator you can use the `generator` static function on `Fn`:

	Generator g := Fn.generate((integer i) -> i+1);
	print g.generate().toString(); // returns 1
	print g.generate().toString(); // returns 2
	g := <Generator> Fn.filter(g, Fn.even);
	print g.generate().toString(); // returns 4
	print g.generate().toString(); // returns 6

There are also several static functions which create pre-defined generators on `Fn`. For example:

	Generator g := Fn.count(); // increments from 0
	Generator g := Fn.repeat("A"); // an infinite series of "A"

Each of these static functions also exist as a static function on `Functional` which return a `Functional` object which can have operators called on them fluently:

	integer evenSum := <integer> Functional.count().filter(Fn.even).slice(0,10,1).reduce(Fn.sum); // sum of the first 10 even numbers

Here is the list of all the generator functions on `Fn` and `Functional`:

|Generator|Arguments|Returns|
|---|---|---|
|generator|`action<TYPE> returns TYPE`|Returns the result of calling the functor on the previous value, starting from a default-initialized TYPE.|
|generatorFrom|Initial value<br/>`action<TYPE> returns TYPE`|Returns the result of calling the functor on the previous value, starting from the initial value.|
|count|None|A sequence of increasing integers starting at 1.|
|repeat|A value|The given value repeated infinitely.|
|cycle|A sequence|Generates each value in the sequence in turn, going back to the first element after completing the sequence.|
|range|Start integer<br/>End integer (End>start)<br/>Number to skip each time (1+)|Returns a finite sequence, not a generator, of the numbers in the given range.|
|sequenceOf|A value<br/>The number of times to repeat the value|Returns a finite sequence, not a generator, containing the given value a given number of times.|

Generators can be wrapped in a `Functional` and the fluent methods called on it directly. If you have a generator in a `Functional`, the resulting object has a `generate` method which can be called to step the underlying generator and return the value, without having to call `get` first.

Although there exists a `Generator` type for generators returned from built-in functions, any event which has an `action generate() returns TYPE` can be used in any context you can use a generator.

## Partial function evaluation

If you are using a version of Apama without lambdas, then you can use action variables instead. However, if you want to capture local variables, while that's simple in a lambda:

	integer factor := 5;
	Fn.map(container, (integer i)->i*factor);

To use an action you would need to write an event type to wrap the local you wanted to capture and provide the function as an action on it:

	event Multiplier { integer factor; action func(integer i) returns integer { return factor * i; } }
	...
	integer factor := 5;
	Fn.map(container, Multiplier(5).func);

As an alternative, `Fn` provides a function to partially satisfy function arguments and return an object which can be used in the place of a functor to later evaluate with the full arguments. For example:

	action mul(integer factor, integer i) returns integer { return factor * i; }
	...
	Fn.map(container, Fn.partial(mul, 5));

The partial stores the first argument and then map is called with the second argument, evaluating the function once all arguments are available. 

You can also store and directly execute a partially evaluated function with the `exec` function:

	Partial p := Fn.partial(mul, 5);
	p.exec(3);

You can also chain partials and stash multiple arguments using a sequence:

	Fn.partial(fn, [1,2,3]).partial(4).exec(5); // executes fn(1,2,3,4,5)

## Using functional elements from EPL

The library is provided as a single EPL file. To use this library from your EPL code, simply import the file `Functional.mon` into your project, then add the types to your EPL file with:

	using com.apamax.functional.Fn;
	using com.apamax.functional.Functional;
	using com.apamax.functional.Generator;
	using com.apamax.functional.Partial;

Then you can just start using functional operations in your code.	

## Running tests

Several tests for the functional operators are included with this repository. To run the tests you will need to use an Apama command prompt to run the tests from within the `tests` directory:

    pysys run

## API documentation

API documentation can be found here: [API documentation](https://mjj29.github.io/apama-epl-functional/)
