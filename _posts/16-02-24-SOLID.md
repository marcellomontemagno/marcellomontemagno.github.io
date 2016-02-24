### What is SOLID?

SOLID is a mnemonic acronym for five programming principles

### What are Software Design Principles?

Software design principles represent a set of guidelines that helps us to avoid having a bad design.

### What are usual characteristics of bad design?

According to Robert Martin there are 3 important characteristics of a bad design that should be avoided:

- Rigidity: It is hard to change because every change affects too many other parts of the system.
- Fragility: When you make a change, unexpected parts of the system break.
- Immobility: It is hard to reuse in another application because it cannot be disentangled from the current application.

### S: Single Responsibility Principle

'A class should have only one reason to change.'

Assume you have a class with two responsibility, and assume a lot o other classes are using this class, if you have to change your class because of a bug related to only one of his responsibility you'll expose all the dependent classes to a possible regression, even it they're interested in the behaviour that was not affected by the bug.

### O: Open Close Principle

'Software entities like classes, modules and functions should be open for extension but closed for modifications.'

When writing your classes, make sure that when you need to extend their behavior you don't have to change the class but to extend it. If you have a library containing a set of classes there are many reasons for which you'll prefer to extend it without changing the code that was already written (e.g backward compatibility, regression testing). This is why we have to make sure our modules follow Open Closed Principle.

### L: Liskov's Substitution Principle

'Derived types must be completely substitutable for their base types.'

We must make sure that new derived classes are extending the base classes without changing their behavior. The new derived classes should be able to replace the base classes without introducing any unexpected behavior and any change in the code. You should feel free to interchange subtypes, hinerithance is the fundament of reuse in OOP.

### I: Interface Segregation Principle

'Clients should not be forced to depend upon interfaces that they don't use.'

If we add methods that should not be there the classes implementing the interface will have to implement those methods as well. For example if we create an interface called Worker and add a method lunch break, all the workers will have to implement it. What if the worker is a robot? probably this method will not be implemented in this case.

### D: Dependency Inversion Principle

'High-level modules or classes should not depend on low-level modules. Both should depend on abstractions.'

In the classical way when a software module (class, framework) need some other module, it initializes and holds a direct reference to it. This will make the 2 modules tight coupled leading to a problem, if we need to change the dependency we have to replace it in every piece of the code it was used. In order to decouple them the first module will provide a hook (usually a property, parameter implementing an abstraction) and an external module controlling the dependencies will have the responsibility to inject the reference to the second one.

By applying the Dependency Inversion the modules can be easily changed just changing one and only one component, the dependency module. 
