---
layout: post
title: How "this" keyword works in JS
category: Javascript
---

## Why this article?

The 'this' keyword could be misleading for programmers new to javascript, it behaves differently compared to other languages like Java.
In this article I'll try to describe in a complete but concise way how it works.

### "This" in the Global Context

"the value of this in the global context (outside of any function), refers to the global object, whether in strict mode or not"

    // In web browsers, the window object is also the global object:
    this === window // true

    // In Node:
    this === global; // true

### "This" in the Function Context

"Inside a function, the value of this depends on how the function is invoked"

Let's see how it changes depending on the function invocation pattern

#### 1) Simple function call

"When a function is invoked as a simple call his value of this refers to the global object or undefined if running in strict mode"

    function f1(){
      return this;
    }

    // In a browser:
    f1() === window; // the window is the global object in browsers

    // In Node:
    f1() === global;

    function f2(){
      "use strict";
      return this;
    }

    // both in a browser or in node:
    f2() === undefined;

#### 2) As an object method

"When a function is called as a method of an object, its 'this' value is set to the object the method is called on, whether in strict mode or not"

    var obj = {
      prop: 30,
      f: function() {
        console.log('-->',this);
      }
    };

    obj.f(); //will log --> Object {prop:30}

it doesn't matter how you define the object, what matter is that the function is invoked as a member of obj 'obj.f()'

    var objF = function() {
        console.log('-->',this);
    }

    var obj = {
      prop: 30
    };

    obj.f = objF;

    obj.f(); //will still log --> Object {prop:30}

this is always true even when we involve the object's prototype chain or object getter and setter.

#### 3) With the new keyword (as a constructor function)

"When a function invoked with the new keyword, its 'this' value is the new object being constructed, whether in strict mode or not"

    function C(){
      this.prop = 30;
      console.log('-->',this);
    }

    var o = new C(); //will log --> C {prop: 30}
    console.log(o) //will still log --> C {prop: 30}

Even though the above is always true keep in mind that, if a constructor function return an object, that object will be returned instead of the value of 'this'

    function C2(){
      this.a = 30;
      console.log('-->',this);
      return {a:31};
    }

    var c2 = new C2(); //will log --> C2 {prop:30}
    console.log(c2) //will instead log Object {prop:31}

obviously invoking a method on the new constructed object will fall in the "object method" scenario

#### 4) With call, apply or bind

[Function.prototype.call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call "call"){:target="_blank"}, [Function.prototype.apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply "apply"){:target="_blank"} and [Function.prototype.bind](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind "bind"){:target="_blank"} can be used to manually instruct which value should be used for 'this' during the function execution.

### What's different in strict mode?

"For a strict mode function, the specified this is not boxed into an object, and if unspecified, it will be undefined instead of the global Object.

When not running in strict mode the object passed to a function as 'this' will always be an object, in detail:

- the provided object if a object-valued 'this' is passed
- a boxed value, if a Boolean, string, or number 'this' is passed
- the global object if an undefined or null 'this' is passed

Not only is automatic boxing a performance cost, but exposing the global object in browsers is a security hazard, because the global object provides access to functionality that "secure" JavaScript environments must restrict.


