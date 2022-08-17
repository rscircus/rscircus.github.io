---
layout: post
title: 'Working with Legacy Code'
sub_title: 'All code is legacy code.'
excerpt_separator: <!-- more -->
categories:
    - Code
tags:
    - legacy_code
    - languages
    - refactoring
    - complexity
---

All code is legacy code to some degree. There are various definitions. One of the probably most famous ones is Michael Feather's definition of Legacy Code: _Code without tests_.

<!-- more -->

These four books are mainly dealing with two topics. Designing and working with software (top row) and releasing reliable software which works™ (bottom row)

![](https://rscircus.github.io/assets/img/20220801_legacy_code.png)

as this post will. A few years back I needed to deal with an old [FORTRAN](https://en.wikipedia.org/wiki/Fortran) codebase. This started the journey which over the years led me to cover the books above.

## Working with Legacy Code

According to the definition above untested code is bad code no matter how good it is. Why? Because it is impossible to see if the code gets better or worse upon changes.

### Working with Feedback

Having tested code gives us feedback about our changes. So, given we have to change a system, i.e., codebase. There are two fundamentally two ways of doing it:

- Edit and Pray
- Cover and Modify

Given a (part of a) codebase does not have tests in place, we run into the "Legacy Code Dilemma":

> When we change code, we should have tests in place. To put tests in place, we often have to change code.

To solve this dilemma Feathers suggests the following algorithm:

1. Identify change points
2. Find test points
3. Break dependencies
4. Write tests
5. Make changes and refactor

**Identifying change points** benefits from behavioral approaches. That is, what are the _most_ touched or _least_ touched places in a codebase one can identify using a version control system. Treat your codebase like a crime scene. [TODO: Add reference to Code as a crime scene].

**Finding test points** usually results in **breaking dependencies**, because:

- Sensing: We can't access values our code computes.
- Separation: We can't even get a piece of code into a test harness to run.

Considering the various strategies to implement the above, I can highly recommend Feather's book which is built around questions and situations one runs into while reworking a legacy code base. That is, each chapter has a fundamental question or situation and explains the solution to solve it.

It's impossible to cover all of this, so let's get an overview:

- Chapter 6: I Don’t Have Much Time and I Have to Change It
- Chapter 7: It Takes Forever to Make a Change
- Chapter 8: How Do I Add a Feature?
- Chapter 9: I Can’t Get This Class into a Test Harness
- Chapter 10: I Can’t Run This Method in a Test Harness
- Chapter 11: I Need to Make a Change. What Methods Should I Test?
- Chapter 12: I Need to Make Many Changes in One Area. Do I Have to Break Dependencies for All the Classes Involved?
- Chapter 13: I Need to Make a Change, but I Don’t Know What Tests to Write
- Chapter 14: Dependencies on Libraries Are Killing Me
- Chapter 15: My Application Is All API Calls
- Chapter 16: I Don’t Understand the Code Well Enough to Change It
- Chapter 17: My Application Has No Structure
- Chapter 18: My Test Code Is in the Way
- Chapter 19: My Project Is Not Object Oriented. How Do I Make Safe Changes?
- Chapter 20: This Class Is Too Big and I Don’t Want It to Get Any Bigger
- Chapter 21: I’m Changing the Same Code All Over the Place
- Chapter 22: I Need to Change a Monster Method and I Can’t Write Tests for It
- Chapter 23: How Do I Know That I’m Not Breaking Anything?
- Chapter 24: We Feel Overwhelmed. It Isn’t Going to Get Any Better

The book closes with a number of dependency-breaking techniques. These are techniques to enable testing a piece of code. Again in a similar fashion. This time with a quick summary for my own reference. :)

When it is annotated with 'does what it says', there is usually a small algorithm behind it in the book.

 - **Safety first**. Write tests first :)
 - **Adapt Parameter**: Extracting interfaces fails...
 - **Break Out Method Object**. Method goes into a new class.
 - **Definition Completion**. Separate declaration and definition when the language allows it.
 - **Encapsulate Global References**. Does what it says. :)
 - **Expose Static Method**. If something is a utility mark it as such.
 - **Extract and Override Call**. Extract a call. Perfect for global objects and static methods.
 - **Extract and Override Factory Method**. This and most of the following ones are meanwhile parts of most IDEs. Similar to above except with a factory method.
 - **Extract and Override Getter**. Sometimes the best solution on the problems above.
 - **Extract Implementer**. Create an interface from the class definition and let the class implement that interface.
 - **Extract Interface**. Most refactoring tools are able to do this. The compiler helps.
 - **Introduce Instance Delegator**. Move a static class into an instance class step by step to replace behavior.
 - **Introduce Static Setter**. Add a static setter to the singleton to replace the instance. Followed by setting the constructor to be protected.
 - **Link Substitution**. Make use of the linker/classpath.
 - **Parameterize Constructor**. Create the object outside a class and make clients pass it into the constructor as a parameter.
 - **Parameterize Method**. Pass an object from the outside as a parmaeter of a method when it is created by a function.
 - **Primitivize Parameter**. Create a free function doing whatever a given class function does. Create a representation of the functions input and pass it to it.
 - **Pull Up Feature**. Create an abstract superclass for the class that contains the methods and copy them to the superclass.
 - **Push Down Dependency**. Make class abstract and current class a subclass.
 - **Replace Function with Function Pointer**. If the language allows it.
 - **Replace Global Reference with Getter**. Does what it says.
 - **Subclass and Override Method**. Use inheritance in the context of a test to nullify behavior that you do not care about.
 - **Supersede Instance Variable**. Does what it says.
 - **Text Redefinition**. Ruby specific. Ignored that.

## Building software reliably
