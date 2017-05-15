---
layout: post
title: The "event loop" and why it could be more convenient than thread based concurrency
metaDescription: why node is faster than java, an introduction to the event loop and a brief comparison with thread based concurrency
category: Javascript
---

Given that javascript runs on a single thread, have you ever wondered how the browser and node are able to do things concurrently? Furthermore have you ever wondered why Node appears faster than Java in many use cases?

The response to this questions can be found in the javascript concurrency model, javascript has in fact a concurrency model based on an "event loop" despite languages like java where concurrency is based on Treads.

## How the event loop works?

The concurrency model of javascript is composed of four actors: a stack, the heap, a queue and the event loop.

### Stack

Function calls are stored as a stack of frames.

	function foo(b) {
	  var a = 10;
	  return a + b + 11;
	}

	function bar(x) {
	  var y = 3;
	  return foo(x * y);
	}

	console.log(bar(7));

When calling bar, a first frame is created containing bar's arguments and local variables. When bar calls foo, a second frame is created and pushed on top of the first one containing foo's arguments and local variables. When foo returns, the top frame element is popped out of the stack (leaving only bar's call frame). When bar returns, the stack is empty.

### Heap

Objects are allocated in a heap which is just a name to denote a large mostly unstructured region of memory.

### Queue

A JavaScript runtime contains a message queue, which is a list of messages to be processed. A function is associated with each message.

### Event loop

The event loop its simply a loop that is waiting for messages in the queue. When the stack is empty, a message is taken out of the queue and processed. The processing consists of calling the associated function and thus pushing and popping from the stack again. Each poll from the queue is usually referred to as a “tick”.

## "Run-to-completion"

One thread == one call stack == one thing at a time

Each message is processed completely before any other message is processed. So whenever a function runs, it will run entirely before any other code runs. This offers some nice properties when reasoning about your asynchronous code, for example you cannot have memory consistency errors.

Unfortunately this approach also have a downside, if a message takes too long to complete, the application is unable to process anything else, not even user interactions like click or scroll.

## Adding messages

Calling setTimeout will add a message to the queue after the time passed as a second argument. If there is no other message in the queue, the message is processed right away; however, if there are messages, the setTimeout message will have to wait for other messages to be processed. For that reason the second argument indicates a minimum time and not a guaranteed time.

## Zero delays

Zero delay time passed as argument doesn't actually mean the call back will fire-off after zero milliseconds. The execution depends also on the number of awaiting tasks in the queue.

## How this looks while coding?

	(function() {

	  console.log('this is the start');

	  setTimeout(function cb() {
	    console.log('this is a msg from call back');
	  });

	  console.log('this is just a message');

	  setTimeout(function cb1() {
	    console.log('this is a msg from call back1');
	  }, 0);

	  console.log('this is the end');

	})();

will print

	// "this is the start"
	// "this is just a message"
	// "this is the end"
	// "this is a msg from call back"
	// "this is a msg from call back1"

here a simplified explanation of what's going on

- the function runs till the end adding two messages to the queue through setTimeout
- the event loop pop the first added message and execute the associated function
- the event loop pop the second added message and execute the associated function

## Can I play with the event loop?

Philip Roberts did an excellent job with the talk "What the heck is the event loop anyway?" furthermore he built the tool linked below that can show a visual representation of the event loop in action, you find it linked [here](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D "loupe"){:target="_blank"}

## How is all this stuff helping? (Non blocking I/O)

- A very interesting property of the event loop model is that JavaScript, unlike a lot of other languages, never blocks. When the application is waiting for an IndexedDB query to return, a file to be read from the filesystem or an XHR request to return, it can still process other things like user input or new web requests.

- Because js executes only one function at a time the developer doesn't have to worry about memory inconsistency problems like in thread based languages.

## The comparison between the two tipical use cases

### Thread based server

The “traditional” mode of web servers has always been one of the thread-based model. For every request a thread is usually retrieved from a pool and starts to execute blocking during input/output operation like reading from disk. Under a big amount of requests the thread pool may become empty resulting in the server to stop serving requests until a thread is available again.

### Event Loop based server

In a node application for every request a message is added to the queue. Once the stack is empty a message is retrieved from the queue and executed, but this time, if an input/output operation is requested the process will not stop but simply add a new message to the queue, giving the chance to other messages to be processed. The result is that the server will not stop serving request.

## Should I always always go for the event loop then?

Not really, it depends on what you want to do with your server/application, the short response is, if you need a lot of I/O this model is likely to be faster and eventually less difficult to code, on the other side if you need expensive and long cpu processing like image processing the event loop model is likely to be slower because only one function at a time is executed.

## Can I have threads in javascript? (Web Workers)

Using Web Workers enables you to offload an expensive operation to a separate thread of execution, freeing up the main thread to do other things. The worker includes a separate message queue, event loop, and memory space independent from the original thread that instantiated it. Communication between the worker and the main thread is done via message passing, which looks very much like the traditional, event loop mechanism.
