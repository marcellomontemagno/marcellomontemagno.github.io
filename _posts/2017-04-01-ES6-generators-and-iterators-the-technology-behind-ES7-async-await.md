---
layout: post
title: ES6 generators and iterators, the technology behind ES7 async await
category: Javascript
---

## Intro

There is a lot to learn about generators but they are very interesting, especially when dealing with asynchronous code.
This article is very dense and longer than my usual ones, but trust me, you'll be rewarded in the end, and you'll finally understand what's behind the awesome ES7 async await feature.

## What Are Iterators?

Iterators are just objects with a specific interface designed for iteration. All iterator objects have a next() method that returns a result object. The result object has two properties: value, which is the next value, and done, which is a boolean that’s true when there are no more values to return. 
If you call next() after the last value has been returned, the method returns done as true and value contains the return value for the iterator or undefined.

Here a simplified implementation of an iterator:
 
    function makeIterator(array) {
        var nextIndex = 0;    
        return {
           next: function() {
               return nextIndex < array.length ?
                   {value: array[nextIndex++], done: false} :
                   {done: true};
           }
        };
    }
    
    const iterator = makeIterator([1,2,3])
    
    let next = iterator.next();
    
    while(!next.done){
        console.log(next.value);
        next = iterator.next();
    }
    
the previous code prints

	1
	2
	3

## What Are Generators?
A generator is a function that returns an iterator.

	// generator
	function *createIterator() {
	    yield 1;
	    yield 2;
	    yield 3;
	}

	//generators are called like regular functions but return an iterator
	let iterator = createIterator();

	console.log(iterator.next().value);
	console.log(iterator.next().value);
	console.log(iterator.next().value);

the previous code will prints:

	1
	2
	3

NOTE: Creating an arrow function that is also a generator is not possible.

The yield keyword, specifies values the resulting iterator should return when next() is called.
The most interesting aspect of generator functions is that they stop execution after each yield statement. After yield 1 executes in this code, the function doesn’t execute anything else until the iterator’s next() method is called. At that point, yield 2 executes. This ability to stop execution in the middle of a function is extremely powerful and leads to some interesting uses of generator functions

The yield keyword can only be used inside of generators. Use of yield anywhere else is a syntax error, including functions that are inside of generators, such as:

	function *createIterator(items) {
	    items.forEach(function(item) {
	        // syntax error
	        yield item + 1;
	    });
	}

Even though yield is technically inside of createIterator(), this code is a syntax error because yield cannot cross function boundaries. In this way, yield is similar to return, in that a nested function cannot return a value for its containing function.

Iterables and for-of

All collection objects (arrays, sets, and maps) and strings are iterables in ECMAScript 6 and so they have a default iterator specified.
Symbol.iterator symbol specifies a function that returns an iterator for the given object

	const a = [1,2,3];
	const iterator = a[Symbol.iterator]();
	iterator.next(); //Object {value: 1, done: false}

Iterables are designed to be used with a the for-of loop.

A for-of loop calls next() on an iterable each time the loop executes and stores the value from the result object in a variable. The loop continues this process until the returned object’s done property is true.

	let values = [1, 2, 3];

	for (let num of values) {
	    console.log(num);
	}

the previous code prints:

	1
	2
	3

## Built-in Iterators

ECMAScript 6 has three types of collection objects: arrays, maps, and sets. All three have the following built-in iterators to help you navigate their content:

- entries() - Returns an iterator whose values are a key-value pair
- values() - Returns an iterator whose values are the values of the collection
- keys() - Returns an iterator whose values are the keys contained in the collection

Each collection type also has a default iterator that is used by for-of whenever an iterator isn’t explicitly specified:

	let colors = [ "red", "green", "blue" ];
	let tracking = new Set([1234, 5678, 9012]);
	let data = new Map();

	data.set("title", "Understanding ECMAScript 6");
	data.set("format", "print");

	// same as using colors.values()
	for (let value of colors) {
	    console.log(value);
	}

	// same as using tracking.values()
	for (let num of tracking) {
	    console.log(num);
	}

	// same as using data.entries()
	for (let entry of data) {
	    console.log(entry);
	}

the previous code prints:

	"red"
	"green"
	"blue"
	1234
	5678
	9012
	["title", "Understanding ECMAScript 6"]
	["format", "print"]

## Passing Arguments to Iterators

You can also pass arguments to the iterator through the next() method.
Arguments passed to next() become the values returned by the yield where the execution was resumed.

	function *createIterator() {
	    let first = yield 1;
	    let second = yield first + 2;       // 4 + 2
	    yield second + 3;                   // 5 + 3
	}

	let iterator = createIterator();

	console.log(iterator.next());           // "{ value: 1, done: false }"
	console.log(iterator.next(4));          // "{ value: 6, done: false }"
	console.log(iterator.next(5));          // "{ value: 8, done: false }"
	console.log(iterator.next());           // "{ value: undefined, done: true }"


Remember that generator functions stop execution after each yield statement till a next is invoked.
See the image below to understand what is executed and when.

![Generator execution](/img/generators.png "Generator execution")

- when createIterator() is invoked no code of the function is executed
- when the first next is invoked the yellow part of the function is executed and 1 is yielded in the iterator value
- when the second next is invoked the argument of the next is returned by the yield in the yellow part, then the blue part of the function is executed yielding 4+2
- when the third next is invoked the argument of the next is returned by the yield in the blue part and the purple part of the function is executed yielding 5+3

Here's another interesting example:

	function* fibonacci() {
	  var fn1 = 0;
	  var fn2 = 1;
	  while (true) {  
	    var current = fn1;
	    fn1 = fn2;
	    fn2 = current + fn1;
	    var reset = yield current;
	    if (reset) {
	        fn1 = 0;
	        fn2 = 1;
	    }
	  }
	}

	var sequence = fibonacci();
	console.log(sequence.next().value);     // 0
	console.log(sequence.next().value);     // 1
	console.log(sequence.next().value);     // 1
	console.log(sequence.next().value);     // 2
	console.log(sequence.next().value);     // 3
	console.log(sequence.next(true).value); // 0
	console.log(sequence.next().value);     // 1
	console.log(sequence.next().value);     // 1
	console.log(sequence.next().value);     // 2
	console.log(sequence.next().value);     // 3

Till no argument is passed to next the value returned by yield is undefined and therefore fn1 and fn2 are not reset.
Notice that this example is also interesting because we have an infinite loop but we are actually stopping the execution of that loop till the another next is invoked. 

## Throwing Errors in Iterators
It’s possible to pass not just data into iterators but also error conditions. Iterators can choose to implement a throw() method that instructs the iterator to throw an error when it resumes.

	function *createIterator() {
	    let first = yield 1;
	    let second = yield first + 2;       // yield 4 + 2
	    yield second + 3;                   // never executed
	}

	let iterator = createIterator();

	console.log(iterator.next());                   // "{ value: 1, done: false }"
	console.log(iterator.next(4));                  // "{ value: 6, done: false }"
	console.log(iterator.throw(new Error("Boom"))); // error thrown from generator

Knowing this, you can catch such errors inside the generator using a try-catch block:

	function *createIterator() {
	    let first = yield 1;
	    let second;

	    try {
	        second = yield first + 2;       // yield 4 + 2, then throw
	    } catch (ex) {
	        second = 6;                     // on error, assign a different value
	    }
	    yield second + 3;
	}

	let iterator = createIterator();

	console.log(iterator.next());                   // "{ value: 1, done: false }"
	console.log(iterator.next(4));                  // "{ value: 6, done: false }"
	console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
	console.log(iterator.next());                   // "{ value: undefined, done:true }"

Notice that something interesting happened: the throw() method returned a result object just like the next() method. Because the error was caught inside the generator, code execution continued on to the next yield and returned the next value, 9.

It helps to think of next() and throw() as both being instructions to the iterator. The next() method instructs the iterator to continue executing (possibly with a given value) and throw() instructs the iterator to continue executing by throwing an error. What happens after that point depends on the code inside the generator.

## Generator Return Statements

You can use the return statement to exit early 

	function *createIterator() {
	    yield 1;
	    return;
	    yield 2;
	    yield 3;
	}

	let iterator = createIterator();

	console.log(iterator.next());           // "{ value: 1, done: false }"
	console.log(iterator.next());           // "{ value: undefined, done: true }"

You can also use the return to specify a yielded value for the last call to the next() method. 
If no return is present, the last call to next() on an iterator returns undefined, but if a return is present, the done property is set to true and the value, if provided, becomes the value field.

	function *createIterator() {
	    yield 1;
	    return 42;
	    return 2; //unreachable code
	}

	let iterator = createIterator();

	console.log(iterator.next());           // "{ value: 1, done: false }"
	console.log(iterator.next());           // "{ value: 42, done: true }"
	console.log(iterator.next());           // "{ value: undefined, done: true }"

Note: The spread operator and for-of ignore any value specified by a return statement. As soon as they see done is true, they stop without reading the value. 

## Delegating Generators

Generators can delegate to other iterators using a special form of yield with a star (*) character.

	function *createNumberIterator() {
	    yield 1;
	    yield 2;
	}

	function *createColorIterator() {
	    yield "red";
	    yield "green";
	}

	function *createCombinedIterator() {
	    yield *createNumberIterator();
	    yield *createColorIterator();
	    yield true;
	}

	var iterator = createCombinedIterator();

	console.log(iterator.next());           // "{ value: 1, done: false }"
	console.log(iterator.next());           // "{ value: 2, done: false }"
	console.log(iterator.next());           // "{ value: "red", done: false }"
	console.log(iterator.next());           // "{ value: "green", done: false }"
	console.log(iterator.next());           // "{ value: true, done: false }"
	console.log(iterator.next());           // "{ value: undefined, done: true }"

The iterator returned from createCombinedIterator() appears, from the outside, to be one consistent iterator that has produced all of the values. 

Generator delegation also lets you make further use of generator return values. 

	function *createNumberIterator() {
	    yield 1;
	    yield 2;
	    return 3;
	}

	function *createRepeatingIterator(count) {
	    for (let i=0; i < count; i++) {
	        yield "repeat";
	    }
	}

	function *createCombinedIterator() {
	    let result = yield *createNumberIterator();
	    yield *createRepeatingIterator(result);
	}

	var iterator = createCombinedIterator();

	console.log(iterator.next());           // "{ value: 1, done: false }"
	console.log(iterator.next());           // "{ value: 2, done: false }"
	console.log(iterator.next());           // "{ value: "repeat", done: false }"
	console.log(iterator.next());           // "{ value: "repeat", done: false }"
	console.log(iterator.next());           // "{ value: "repeat", done: false }"
	console.log(iterator.next());           // "{ value: undefined, done: true }"

Here, the createCombinedIterator() generator delegates to createNumberIterator() and assigns the return value to result.

## Asynchronous Task Running

That was a lot of stuff to lear, but where is your reward?

Since generators + iterators allow you to effectively pause and control functions code from the outside of that function, they open up a lot of possibilities related to asynchronous processing.

But before digging in asynchronous processing let's create a synchronous task runner that is able to perform a generator function till the end:

	function run(taskDef) {

	    // create the iterator, make available elsewhere
	    let task = taskDef();

	    // start the task
	    let result = task.next();

	    // recursive function to keep calling next()
	    function step() {

	        // if there's more to do
	        if (!result.done) {
	            result = task.next(result.value);
	            step();
	        }
	    }

	    // start the process
	    step();

	}

This simple function invokes the next() on the iterator returned from a generator till done is false.
Is also important to notice that every result.value is also passed to the following next() as an argument.
Here an example of the task runner usage:

	run(function*() {
	    let value = yield 1;
	    console.log(value);         // 1
	    value = yield value + 3;
	    console.log(value);         // 4
	});

but how this helps with asynchronous processing?

If you don't consider promises, The traditional way to perform asynchronous operations is to call a function that has a callback. For example, consider reading a file from the disk in Node.js:

	let fs = require("fs");

	fs.readFile("config.json", function(err, contents) {
	    if (err) {
	        throw err;
	    }
	    doSomethingWith(contents);
	    console.log("Done");
	});

After the operation read operation is finished, the callback function is called with the result.
With this in mind, you can modify the task runner to take so that anytime result.value is a function, the task runner will execute it instead of just passing that value to the next() method. Here’s the updated code:

	function run(taskDef) {

	    // create the iterator, make available elsewhere
	    let task = taskDef();

	    // start the task
	    let result = task.next();

	    // recursive function to keep calling next()
	    function step() {

	        // if there's more to do
	        if (!result.done) {
	            if (typeof result.value === "function") {
	                result.value(function(err, data) {
	                    if (err) {
	                        result = task.throw(err);
	                        return;
	                    }

	                    result = task.next(data);
	                    step();
	                });
	            } else {
	                result = task.next(result.value);
	                step();
	            }

	        }
	    }

	    // start the process
	    step();

	}

This new version of the task runner is ready for all asynchronous tasks, here an example:
    
    const readConfigFile = function*() {
        let contents = yield readFile("config.json");
        doSomethingWith(contents);
        console.log("Done");
    }

    run(readConfigFile);

This example is performing the asynchronous readFile() operation without making any callbacks visible in the main code, cool isn't it?
This pattern + promises instead of callbacks is basically what's behind the async await feature in ES7.
Don't you think that "function*", "yield" and "run" look a lot like "async", "await" and invoking a function marked as "async"?
