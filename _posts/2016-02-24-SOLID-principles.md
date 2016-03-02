---
layout: post
title: SOLID principles
tags: 
- software design
---

### What is SOLID?

SOLID is a mnemonic acronym for five programming principles originally codified from Robert Martin "Uncle Bob". These principles focus on making systems flexible and adaptable when changes are required.

### What are Software Design Principles?

Software design principles represent a set of guidelines that helps us to avoid having a bad design.

### What are usual characteristics of bad design?

According to Robert Martin "Uncle Bob" there are 3 important characteristics of a bad design that should be avoided:

- Rigidity: It is hard to change because every change affects too many other parts of the system.
- Fragility: When you make a change, unexpected parts of the system break.
- Immobility: It is hard to reuse in another application because it cannot be disentangled from the current application.

### S: Single Responsibility Principle

'A class should have only one reason to change.'

You should have isolated behaviours, in small, cohesive, packages. This allow you to make changes to a functionality safely, you'll not risk to break part of the system that don't need to change.

### O: Open Close Principle

'Software entities like classes, modules and functions should be open for extension but closed for modifications.'

If you have a library containing a set of classes there are many reasons for which you'll prefer to extend it without changing the code that was already written e.g backward compatibility and regression testing. In other words changing code could be dangerous, once you have it written and tested you want to minimize chances to indroduce bugs. 

Pay attention, there is the danger of introducing unnecessary extension points focusing too much on this principle, be careful to create extension points only for part of the system that will really change.

### L: Liskov's Substitution Principle

'Derived types must be completely substitutable for their base types.'

We must make sure that new derived classes are enanching the base classes behaviour without changing it. Polimorphism is a key part of flexible design, you should feel free to interchange subtypes.

### I: Interface Segregation Principle

'Interfaces should be small, focusing on a specific use case, clients should not be forced to depend upon interfaces that they don't use.'

If we add methods that should not be there the classes implementing the interface will have to implement those methods as well. For example if we create an interface called Worker and add a method lunch break, all the workers will have to implement it. What if the worker is a robot? probably this method will not be implemented in this case.

### D: Dependency Inversion Principle

'High-level modules or classes should not depend on low-level modules. Both should depend on abstractions.'

In the classical way when a software module (class, framework) need some other module, it initializes and holds a direct reference to it. This will make the 2 modules tight coupled leading to a problem, if we need to change the dependency we have to replace it in every piece of the code it was used. In order to decouple them the first module will provide a hook (usually a property, parameter implementing an interface) and an external module controlling the dependencies will have the responsibility to inject the reference to the second one.

By applying the Dependency Inversion the modules can be easily changed just changing one and only one component, the dependency module. 
