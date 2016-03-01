---
layout: post
title: The four rules of simple design
---

### Good Design?

The idea of "good design" can often lead to the feeling that THE BEST design exists, unfortunately this is just not true. There are always more than one design that works well, is to look at things from a comparison point of view that lead to discover what makes a design "better" for the problem we have to solve.

### So what make a design "better" than another?

If you think about a constants in software development, one of them is that things are going to change, sooner or later, you fantastic design will have the need to adapt to something new.

- Striving for a simple design, one that is adaptable to changing needs, makes a design a better design.
- Whenever we have a choice to make, look for the easier to change, makes a design a better design.

Is important to not misunderstand the previous tho statements. Is not making everything configurable or build plenty of extension point the key for better a design. Quite the opposite.

As a matter of fact, there is another constant in software development, we don't know what is going to change, and of course, we are not good in predicting the future. Everytime we introduce an extension point we introduce complexity in our design and of course we spend a lot of effort dealing with them.

- Don't write code that guesses the future, write code that will be easy to change when the future arrives, if it will arrive.

As time goes on on our system, we learn about places that are frequently going to change, only once we acquire that knowledge we have a good reason to introduce an extension point in our system.

### The 4 Rules of Simple Design

Originally codified by Kent Beck, these rules are listed below in a simplified form:

1. Tests Pass

If you can't easily verify that your design works it doesn't really matter how great it is. Notice that the rule doesn't state "Automated tests Pass" it is only about correctness and verification. But never forget that, our system is going to change, how much time we will need to verify that our system is still working after a change is a crucial factor in big applications.

2. Expresses Intent

One of the most important qualities of a codebase, when it comes to change, is how quickly you can find the part that should change. Anoter very important quality is how simple could be to understand that part of the system in order to be able to change it. Paying attention on how our code expresses itself is crucial, it is the key to make our lives easy when we'll need. Notice that pay attention to names is only a little part of this process.

3. No duplication (DRY)

This rule doesn't refer to duplication at a code level, it is about knowledge duplication. DRY: Every piece of knowledge should have one and only one representation. Writing new code we could easily introduce abstractions that was already present somewhere in your system, probably our old abstraction could be reused or they could adapt to our new code if we are really dealing with a knowledge duplication.

4. Small

Once applied the other rules, is important to look back and make sure that we don't have something that can be removed, making our code simpler and smaller. You could for example have some vestigial code that seemed like a good idea at the time you wrote the code but that does not give any effective value in the final product.
