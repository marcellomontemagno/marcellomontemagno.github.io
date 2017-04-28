---
layout: post
title: A concise summary about promises
category: Javascript
---

## Why this article?

There is a lot of documentation about promises online but in my opinion it is often verbose and confusing, especially about error handling.
In this article I'll try to cover what I think you should really know about it in few lines and with a lot of examples.

## What is a promise?

A promise is a placeholder for the result of an asynchronous operation

    var promiseInstance = readFile("example.txt");

a promise internal state could be

- pending: the async operation is not completed yet

- fulfilled: the async operation is completed

- rejected: a problem occurred during the async operation

This property isn’t exposed on promise objects, but you can take a specific action when a promise changes state by using the then() and catch() methods

	promiseInstance.then(yourFunction)

yourFunction will be executed when the promise is fulfilled

	promiseInstance.catch(yourFunction)

yourFunction will be executed when the promise is rejected

## Creating you first promise

A promise can be created using the Promise constructor as follow

    new Promise(executorFunction);

here's an example of how to create a promise

    var myFirstPromise = new Promise(function(resolve, reject){
        //we use setTimeout(...) to simulate async code
        setTimeout(function(){
            resolve("Success!!!");
        }, 500);
    });

    myFirstPromise.then(function(successMessage){
        console.log(successMessage); //Success!!!
    });

    myFirstPromise.catch(function(error){
        console.log(error); //never invoked in this example
    });

- the "executorFunction" runs immediately and synchronously
- the then block is executed asynchronously only after resolve has been invoked
- the catch block is executed asynchronously only after reject has been invoked or an exception has been thrown
- if the then or the catch block return a promise that promise will be returned
- if the then or the catch block return any other value that value will be wrapped in a promise and that promise will be returned

## Promises can be chained

Because values returned by the promises are always wrapped in promises, promises can be chained, here an example

    var p1 = new Promise(function(resolve, reject) {
        resolve(42);
    });

    p1.then(function(value) {
        console.log(value); // "42"
        return value + 1;
    }).then(function(value) {
        console.log(value); // "43"
        return value + 1;
    }).then(function(value) {
        console.log(value); // "44"
    });

## Avoiding nesting promises

If inexperienced it could be tempting to write code like the following

    var resolve5 = new Promise(function(resolve, reject) {
        resolve(5);
    });

    resolve5.then(function(value) {
        var add1 = new Promise(function(resolve, reject) {
            resolve(1+value);
        }).then(function(value) {
            var subtract2 = new Promise(function(resolve, reject) {
                resolve(value-2);
            }).then(function(value) {
                console.log(value); // 4
            })
        })
    });

the same could be written as follow enhancing code reuse, separation of concerns, and helping with exception handling (explained later)

    var resolve5 = new Promise(function(resolve, reject) {
        resolve(5);
    });

    var add1 = function(value){
        return new Promise(function(resolve, reject) {
            resolve(1+value);
        });
    };

    var subtract2 = function(value){
        return new Promise(function(resolve, reject) {
            resolve(value-2);
        });
    };

    resolve5
    .then(add1)
    .then(subtract2)
    .then(function(result){
        console.log(result); //4
    });

## Error handling

All the exceptions either thrown in the executorFunction or in any then and catch blocks will be swallowed

    var p1 = new Promise(function(resolve, reject) {
        console.log('executing p1');
        throw new Error("Explosion!");
    });

    console.log("I'm executed the exception has been swallowed");

running the previous example will in fact log:

    executing p1
    promises throw:6 I'm executed the exception has been swallowed

Exceptions/rejections can be handled in a single catch block and handling them may change the resolution flow

    var p1 = new Promise(function(resolve, reject) {
        resolve(1);
    });

    var p2 = new Promise(function(resolve, reject) {
        reject(new Error("Explosion!"));
    });

    var p3 = new Promise(function(resolve, reject) {
        resolve(3);
    });

    p1.then(function(value) {
        console.log(value); // 1
        return p2;
    }).then(function(value) {
        console.log(value); // never invoked because p2 is rejected
        return p3;
    }).catch(function(error) {
        console.log(error); // Explosion!
    });

catch block can return values to continue the flow

    var p1 = new Promise(function(resolve, reject) {
        resolve(1);
    });

    var p2 = new Promise(function(resolve, reject) {
        reject(new Error("Explosion!"));
    });

    var p3 = new Promise(function(resolve, reject) {
        resolve(3);
    });

    p1.then(function(value) {
        console.log(value); // 1
        return p2;
    }).catch(function(error) {
        console.log(error); // Explosion!
        return p3;
    }).then(function(value) {
        console.log(value); // 3
    }).catch(function(error) {
        console.log(error); // never invoked
    });

reject a promise or throw an exception inside a promise is exactly the same because exceptions will be transformed in rejections, unfortunately you have to keep in mind this particular case:

    new Promise(function(resolve,reject) {
      setTimeout(function() {
        throw new Error("Explosion!");
      }, 500);
    }).catch(function(e) {
      console.log(e); // Never invoked
    });

in the previous example the exception is thrown in the asynchronous part of the code and because of that the executor function cannot know about the thrown exception and will not be able to transform it into a rejection.
To fix it you can either use reject as follow

    new Promise(function(resolve,reject) {
      setTimeout(function() {
        reject(new Error("Explosion!"));
      }, 500);
    }).catch(function(e) {
      console.log(e); // Explosion!
    });

or transform setTimeout into a promise based api as follow

    var timeout = function(fn,mills){
        return new Promise(function(resolve,reject){
            setTimeout(function(){
                try {
                    resolve(fn());
                } catch (e) {
                    reject(e);
                }
            }, mills);
        });
    }

    timeout(function(){
        throw new Error("Explosion!");
    }, 500)
    .catch(function(e){
        console.log(e); // Explosion!
    });

## Global Promise Rejection Handling

Having the exceptions swallowed may be confusing, you can log them using the following event handler on the browser (node has a similar one too)

    window.onunhandledrejection = function(event) {
        console.log(event.type); // "unhandledrejection"
        console.log(event.reason.message); // "Explosion!"
    };

    Promise.reject(new Error("Explosion!"));

## Promise.resolve()

Promise.resolve(value) method accepts a value as argument and returns a promise in the fulfilled state, everything happen synchronously.

    var promise = Promise.resolve(42);

    promise.then(function(value) {
        console.log(value);  // 42
    });

## Promise.reject()

    var promise = Promise.reject(42);

    promise.catch(function(value) {
        console.log(value);         // 42
    });

Promise.resolve(value) method accepts a value as argument and returns a promise in the rejected state, everything happen synchronously.

## Responding to Multiple Promises

Sometimes, you’ll want to monitor the progress of multiple promises in order to determine the next action

The Promise.all() method returns a single Promise that resolves when all of the promises in the iterable argument have resolved, or rejects with the reason of the first promise that rejects.

    var p1 = Promise.resolve(3);
    var p2 = 1337;
    var p3 = new Promise(function(resolve, reject){
      setTimeout(resolve, 100, 'foo');
    });

    Promise.all([p1, p2, p3]).then(functionvalues{
      console.log(values); // [3, 1337, "foo"]
    });
