---
layout: post
title: Everything you need to know about objects in JS, From object literals to ES6 classes, inheritance included
metaDescription: A comprehensive guide on objects in javascript with a comparison between different styles and examples of inheritance
category: Javascript
---

## How many ways you have to create object javascript?

Unfortunately you have many

- Using Object.create(...)
- Using Object literals
- Using constructor functions
- Using ES6 classes

All of them will involve inheritance so is important to have some basic info about it before to start.

## Class based vs prototypal inheritance

In class based inheritance you have classes

- Classes are not object instances.
- Classes are a blueprint, a description used to create new object instances.
- Classes can inherit from other classes.
- Classes can be used to create new object instances through the "new" operator.

In prototypal inheritance classes do not exist

- You only have real object instances.
- Objects inherit directly from other real object instances.

Javascript doesn't have class based inheritance, it has prototypal inheritance. The class keyword introduced in ES2015 is only syntactical sugar, under the hood it creates a structure of objects instances mimicking the class based model.

## What is a prototype?

A prototype is an object instance nothing more than that. JavaScript objects usually inherit properties and methods from a prototype.

## The prototype chain, properties delegation, and properties shadowing

In prototypal inheritance having an object C extending B and B extending an object A means that the instance C is linked to the instance B through a reference and that B is linked to A through another reference. This creates a chain called the prototype chain. When you ask for a property on the object C the following happen:

- if the property is present its value is returned
- if the property is not present the hidden [[prototype]] property is used to retrieve the object prototype (its parent) and the same logic is performed on it until either a property is found or the end of the prototype chain is reached

The hidden [[prototype]] property of objects is the one holding the link to their prototype (parent), inspecting an object in chrome this property has the name "\_\_proto\_\_".

Note: is not safe to access the property "\_\_proto\_\_" directly, we will see later a safe way to do it.

The mechanism of going from the bottom to the top of the prototype chain in order to retrieve a property is called properties delegation.

The property delegation returns only the first property with the requested name encountered while going through the chain, this means that all the other occurrences of property with the same name that may be present later on in the chain will be shadowed (properties shadowing).

## Object.create

The most basic way to create object in javascript is through the Object.create method.

The Object.create(proto[, propertiesObject]) method creates a new object with the specified prototype object and properties.

### The basic way to create an object with no prototype?

    var obj = Object.create(null);
    obj.field1 = 1;

obj will be a simple plain object, inspecting obj in chrome will show no "\_\_proto\_\_" property hence no parent is present, obj has no prototype.

### The basic way to create an object with a prototype

    var protoObj = Object.create(null);
    protoObj.field1 = 1;

    var childObj = Object.create(protoObj);

inspecting childObj in chrome will show a "\_\_proto\_\_" property with value protoObj, protoObj is the childObj's prototype.

    console.log(childObj.field1) // 1

Even though childObj doesn't contain the field1 directly childObj.field1 will print 1 because of the property delegation through the prototype chain.

In real life you will not use Object.create with null as an argument, these examples are there as a prelude to better explain the following chapters.

## Object literals

In real life when creating objects you usually want also a set of feature that all object should have like a valueOf method, a toString method etc. Javascript provide a particular object providing all these feature, you can inspect it in the browser as follow:

    Object.prototype

The object literal notation take this in account and create an object whose prototype is Object.prototype by default

    var obj = {field1:1};
    console.log(obj.__proto__ === Object.prototype); //true

because of properties delegation now you could also call obj.toString(), obj.valueOf() etc.

Note: in the previous example "\_\_proto\_\_" has been accessed directly only for explanation purposes, is not safe to access directly to this property.

### Object.getPrototypeOf

The proper way to access the [[prototype]] property value is with Object.getPrototypeOf

The Object.getPrototypeOf() method returns the prototype (i.e. the value of the internal [[Prototype]] property) of the specified object.

    Object.getPrototypeOf(obj);

### Object.setPrototypeOf

The Object.setPrototypeOf() method sets the prototype (i.e., the internal [[Prototype]] property) of a specified object to another object or null. Notice that this operation is usually slow. If your application has critical performance needs, create a new object with the desired [[Prototype]] using Object.create() instead of using Object.setPrototypeOf to change it after the creation.

    var protoObj = {field1:1};
    var childObj = {field2:2};
    Object.setPrototypeOf(childObj,protoObj);
    console.log(childObj.field1); //1

the previous example is the same as:

    var protoObj = {field1:1};
    var childObj = Object.create(protoObj);
    childObj.field2 = 2;
    console.log(childObj.field1); //1

## Constructor functions

Constructor functions mimic class inheritance creating a particular structure of objects under the hood.

A constructor function is a normal function invoked with the keyword "new".

    new yourFunction([arg1[, arg2[, ...argN]],]){...};

Note: by convention a constructor function name will have the first letter uppercase.

We'll show what is going on under the hood when a constructor functions is used through an example:

    var ObjClass = function(){this.field1=1}; //notice that the function returns nothing
    var objInstance = new ObjClass(); //the new operator does something magic for us but we'll see later

    console.log(ObjClass.prototype.constructor === ObjClass); //true
    console.log(Object.getPrototypeOf(ObjClass.prototype) === Object.prototype); //true

    console.log(Object.getPrototypeOf(objInstance)===ObjClass.prototype); //true

we basically have to explain what is the value of the "prototype" property of the constructor function.

The prototype property of the constructor function links the object that will be used as the value of [[prototype]] for the objects returned when the function is invoked with the "new" operator.

The value of the prototype property of the constructor function is an object having

- A constructor property linking the constructor function
- A [[prototype]] property linking Object.prototype (like any other normal object in js)

I strongly suggest you to run the previous example in the console and inspect objInstance. Inspecting objInstance in the console you can see that it is an object whose [[prototype]] links ObjClass.prototype and so inheriting its constructor property

    console.log(objInstance.constructor === ObjClass); //true

even if constructor property is not directly present on objInstance.

Is important to notice that the prototype property value of ObjClass will be shared between all the instances of ObjClass.

### The "new" operator

We understood that defining a constructor function is like defining an object type, but the function we defined in the previous example uses "this" and returns nothing, where is the magic? The response is, the "new" operator take care of something for us, in detail:

When the code "new ObjClass()" is executed, the following happen:

- A new object is created, inheriting from ObjClass.prototype.
- The constructor function ObjClass is called with the specified arguments, and with this bound to the newly created object. (see [here](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind "details about bind"){:target="_blank"} for details about bind)
- If the constructor function doesn't explicitly return an object, the object created in step 1 is returned. (Normally constructors don't return a value, but they can choose to do so if they want to override the normal object creation process.)
- If the constructor function explicitly returns an object it will be the result of the whole new expression.

Many developer argue about the "new" operator saying that the only way to distinguish between a function that is meant to be invoked with "new" operator and one that is not is through the naming convention. Invoking a function that is meant to be a constructor function without the "new" operator is actually dangerous and will break the behaviour of that function because "this" will have another value and the return value will also be different.

    function Polygon(height, width) {
      this.height = height;
      this.width = width;
    }

    var p1 = new Polygon(1,2);
    var p2 = Polygon(1,2);

    console.log(p1); // Polygon {height: 1, width: 2}
    console.log(p2); // undefined

There is actually a defensive technique that involve checking the value of "this" with "instanceof" during the object construction. The instanceof operator tests whether an object has in its prototype chain the prototype property of the constructor given as argument.

    object instanceof constructor

Here the implementation of the defensive technique:

    function Polygon(height, width) {
      if (!(this instanceof Polygon)) {
        throw new Error("cannot be invoked without 'new'");
      }
      this.height = height;
      this.width = width;
    }

    var p1 = new Polygon(1,2); //object created successfully
    var p2 = Polygon(1,2); //cannot be invoked without 'new'

And is also true that ES6 classes (explained later) take care of this for you

    class Polygon {
      constructor(height, width) {
        this.height = height;
        this.width = width;
      }
    }

    Polygon(1,2); //Uncaught TypeError: Class constructor Polygon cannot be invoked without 'new'

### Important considerations about the prototype

Is important to notice that because of the way "new" works a constructor function prototype property value is actually shared between all the constructed instances.

    var ObjClass = function(fieldValue){this.field=fieldValue};
    var objInstance1 = new ObjClass(1);
    var objInstance2 = new ObjClass(2);
    Object.getPrototypeOf(objInstance1) === Object.getPrototypeOf(objInstance2);

this means that adding a method on ObjClass.prototype will actually make it usable from all the ObjClass instances because ObjClass.prototype is linked in all the instances prototype chain

    var ObjClass = function(fieldValue){this.field=fieldValue};
    var objInstance1 = new ObjClass(1);
    var objInstance2 = new ObjClass(2);
    ObjClass.prototype.aMethod = function(){console.log('invoked!')};
    objInstance1.aMethod(); //invoked!
    objInstance2.aMethod(); //invoked!
    console.log(objInstance1.aMethod === objInstance2.aMethod); //true

a different things is instead adding a method on one of the instances

    var ObjClass = function(fieldValue){this.field=fieldValue};
    var objInstance1 = new ObjClass(1);
    var objInstance2 = new ObjClass(2);
    objInstance1.aMethod = function(){console.log('invoked!')};
    objInstance1.aMethod(); //invoked!
    objInstance2.aMethod(); //Uncaught TypeError: objInstance2.aMethod is not a function

and also a different is adding a method through "this" while creating the object

    var ObjClass = function(fieldValue){
        this.field=fieldValue;
        this.aMethod = function(){console.log('invoked!')};
    };
    var objInstance1 = new ObjClass(1);
    var objInstance2 = new ObjClass(2);
    objInstance1.aMethod(); //invoked!
    objInstance2.aMethod(); //invoked!
    console.log(objInstance1.aMethod === objInstance2.aMethod); //false

because of the way "new" works in the latest example we actually have two different instances of the function "aMethod" and they are actually stored on the objInstance1 and objInstance2 directly, not in their prototype chain.

### JS implicit constructor functions

Without knowing it we were already using some constructor function provided by javascript, one example is Object, the example below proves it:

    var obj = {};
    console.log(Object.getPrototypeOf(obj) === Object.prototype); //true
    console.log(obj.constructor === Object); //true
    console.log(obj.constructor === Object.prototype.constructor); //true

notice also that

    var a = Object.create(Object.prototype);
    var b = {};
    var c = new Object();

all perform the same operation.

Another implicit constructor function that we use very often without knowing is Function, the following example proves it:

    var myFunction = function(){};
    console.log(Object.getPrototypeOf(myFunction) === Function.prototype); //true
    console.log(myFunction.constructor === Function); //true
    console.log(myFunction.constructor === Function.prototype.constructor); //true

Through this example you can also prove that a function in javascript is actually an object, it inherits from Function.prototype that inherits from Object.prototype

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(myFunction)) === Object.prototype); //true

### Extending a constructor function correctly

Imagine you want to create a structure where Square extends Polygon that extends Object.

In this case you want the following to happen

- When creating a Square instance the creation logic of Polygon needs to be invoked to build the Square instance
- A Square instance [[prototype]] must link Square.prototype that is shared between all the Square instances
- The prototype of the prototype of the square instance must link Polygon.prototype that is shared between all the Polygon instances
- The prototype of the prototype of prototype of the square instance must link Object.prototype that is shared between all the Object instances

here the example implemented properly

    function Polygon(height, width) {
      this.height = height;
      this.width = width;
    }

    function Square(sideLength){
      Polygon.call(this,sideLength,sideLength);
    }

    Square.prototype = Object.create(Polygon.prototype);
    Square.prototype.constructor = Square; //we lost this property value with the previous line, with this line we are just restoring it

    Square.prototype.area = function(){
      return this.height * this.width;
    }

    var square = new Square(2);

    console.log(Object.getPrototypeOf(square) === Square.prototype); //true
    console.log(Object.getPrototypeOf(square).constructor === Square); //true

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(square)) === Polygon.prototype); //true
    console.log(Object.getPrototypeOf(Object.getPrototypeOf(square)).constructor === Polygon); //true

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(square))) === Object.prototype); //true
    console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(square))).constructor === Object); //true

### Can I have the same exact structure with no "new" operator?

well you could, here the same example mimicking class inheritance without the "new" operator:

    function Polygon(height, width) {
        var instance = Object.create(Polygon.prototype);
        instance.height = height;
        instance.width = width;
        return instance;
    }

    Square.prototype = Object.create(Polygon.prototype);
    Square.prototype.constructor = Square;
    Square.prototype.area = function(){
        return this.height * this.width;
    }

    function Square(sideLength){
        return Object.assign(Object.create(Square.prototype),Polygon.call(this,sideLength,sideLength));
    }

    var square = Square(2);

    console.log(Object.getPrototypeOf(square) === Square.prototype); //true
    console.log(Object.getPrototypeOf(square).constructor === Square); //true

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(square)) === Polygon.prototype); //true
    console.log(Object.getPrototypeOf(Object.getPrototypeOf(square)).constructor === Polygon); //true

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(square))) === Object.prototype); //true
    console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(square))).constructor === Object); //true

### Can I just use prototypal inheritance without faking the way class inheritance works?

Well here a simplify version of the same example, is prototypal inheritance, and this time is not trying to fake the way class inheritance works like constructor functions do

    function Polygon(height, width) {
        return {
            height: height,
            width: width
        }
    }

    function Square(height, width){
        return Object.assign(Object.create(Polygon(height, width)),{
            area: function(){
                return this.height * this.width;
            }
        });
    }

    var square = Square(1,2);

    console.log(square.width); // 2
    console.log(square.height); // 1
    console.log(square.area()); // 2

### Should I avoid constructor functions then?

Well, Is not in the scope of this article to persuade you from using constructor functions, but I suggest you [this](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a#.ijpuy2wht "common misconceptions about inheritance in javascript"){:target="_blank"} further reading if you are considering it.

## What about ES6 classes?

ES6 classes are built on top of constructor functions and they produce the exact same structure as constructor functions, here the same example shown with constructor functions:

    class Polygon {
      constructor(height, width) {
        this.height = height;
        this.width = width;
      }
    }

    class Square extends Polygon {
      constructor(sideLength) {
        super(sideLength, sideLength);
      }
      area() {
        return this.height * this.width;
      }
    }

    var square = new Square(2);

    console.log(Object.getPrototypeOf(square) === Square.prototype); //true
    console.log(Object.getPrototypeOf(square).constructor === Square); //true

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(square)) === Polygon.prototype); //true
    console.log(Object.getPrototypeOf(Object.getPrototypeOf(square)).constructor === Polygon); //true

    console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(square))) === Object.prototype); //true
    console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(square))).constructor === Object); //true

### What is different between constructor functions and ES6 classes?

Most of the differences are in teh syntax in fact, the result of a class definition is actually a function

    typeof Polygon //function

With ES6 classes you have:

- A compact syntax for method definitions (no keyword function needed) and a nice syntax for extension
- No commas between the parts of a class, semicolons can be used but are optional
- Function call protection, classes can only be invoked via the "new" operator
- All the non static methods defined in the class body are added to YourClass.prototype and so shared between all the class instances
- All the static methods defined in the class body are added to YourClass (the constructor function) not to YourClass.prototype
- The class body can only contain methods, but not data properties, prototypes having data properties is generally considered an anti-pattern, so this just enforces a best practice
- All parts of a ClassDeclaration or a ClassExpression are strict mode code so expect "this" to be valued with undefined and not with the global object if a method is invoked as a simple function call and not as an object method (see ["This in the Function Context"](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind "“This” in the Function Context"){:target="_blank"} for further details)
- Unless you bind it, "this" will be undefined in static methods
