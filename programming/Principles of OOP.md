# Principles of OOP

## Introduction

OOP, otherwise known as Object Oriented Programming is a programming
concept of putting data structures inside objects to better isolate functions of
a program and handle large complex relationships between different
abstractions of program features.

Simula was the first Object Oriented language. it was created for building
simulations. This first implementation of Object Oriented programming for
building simulations is a great example of what OO is good at - the idea that
keeping things from being stuck to other things (with the exception of
inheritance) would be useful for this type of task where many objects need to
be handled.

Object Oriented Programming has 4 major principles. they are:

* **Abstraction** - Using a focused representation of an actual item
* **Encapsulation** - Hiding the internals of an object
* **Inheritance** - Reusing code from an existing object
* **Polymorphism** - One name. many forms.

Though separate to the 4 pillars of programming, there are some additional
areas relating to Object Oriented Programming. Namely Coupling and
Cohesion.

### Coupling and Cohesion

Though OOP likes to prevent things from being stuck to other things in ways
that are confusing to a developer. An extension on this idea is Cohesion and
coupling which is used to describe how classes do or do not relate to each
other.

Cohesion describes the quality of a relationship between objects – and how
well each object respects the boundaries of other objects.
Coupling is very similar to Cohesion and describes how much as object can
not rely on other objects. High cohesion is a desirable trait in Object Oriented
Programming where there is a clear boundary between objects. An object is
also cohesive if its purpose can easily be identified and is clear.

An example of these concepts can be seen in some low primitive 2d game
engines where the entire game class is contained within a single object. The
class has a purpose that is unclear and serves more than one easily
understood purpose. This makes Cohesion low. Similarly its not well
decoupled (the opposite of coupled) as there are many different methods
within the ‘game’ class that serve different purposes.

## Abstraction

#### What is Abstraction

"Abstraction denotes the essential characteristics of an object that
distinguishes it from all other kinds of objects. Thus providing a conceptual
boundary relative to the perspective of the viewer" - Grady booch.

In more simple terms. Abstraction takes complex entities and simplifies them
down to a model that can be better understood by a program.

In more functional sense. An abstract class occurs when you don’t have
enough specific information to summarize the same group of objects.

#### Goals of Abstraction

Object oriented programming is an abstraction in itself.
The entire goal of Object Oriented programming is to try and get things into a form where you can work with it, ie. reducing its complexity. The benefits of abstraction is it makes development more modular.

This is the most important aspect of modeling real world objects.
The process in which this is performed involves developing interfaces and
functionality around an object instead of implementing specific details,
separating by the type (the what) then the implementation (the how) – this
makes categorizing fields easier, therefore increasing abstraction.

#### Abstraction vs Encapsulation

Encapsulation is related to abstraction closely because encapsulation is a
loosely related form of abstraction however its referring to how the object
refers to itself rather than how other objects relate to it.
A good comparison between Abstraction and encapsulation is to define
Abstraction as the way your object interfaces out, And encapsulation is the
way that other objects interface in to your object.

## Encapsulation

Attribute accessors are restricted to either accessors or mutators, these fields
are considered state based which means that their attributes store the current
state of the object that are used internally within the class, though these
fields/states can be made available to other classes through the use of
properties to expose them to be pseudo public, it allows the programmer to
decide what data is visible to a program and what should be
hidden/encapsulated within the object.

#### Accessors and Mutators

Accessors are used to exclusively ask an object about itself. As talked about
before they are usually in the form of properties with a get method, another
name for these are ‘read only’ fields.

Mutators are public methods used to alter the state of the object. they still
hide the implementation of the object but make it possible for external classes
to set the otherwise private field.
Through mutators usually use the set property method, both accessors and
mutators can be any public method that alters the state of the object. for
example, regular methods.

## Inheritance

#### What is Inheritance

A good way to think about inheritance is to think of it as a hierarchy.
Ie. If C inherits from B and B inherits from A. then C inherits A as well.
C→B and B→A then C→ A.

#### How Inheritance Works

The way inheritance works as an example:
if you have a vehicle class and need to describe many types of vehicle you
would apply inheritance.
The vehicle class will become abstract, meaning that there is no way to make
a ‘vehicle’, only types of vehicles. Within that class you may have fields for a
make, model, year etc.

The vehicle class then extends that information to other classes, like car,
truck, motorcycle etc. Each of these vehicles have extra information thats
required to propperly describe itself.
relevant information for the car may be if its 2 or 4 door.
relevant information for the truck may be its carrying capacity.
relevant information for the motorcyclce may be its motor CC.

These relevant aspects are extending the class to add additional information
to make it possible to best describe that object, without needing to restate the
info. That means that inside the abstract vehicle class itself, all the base
information is still available to car, truck, and motorcycle.
But its not exclusively related to each vehicle.

#### Drawbacks of Inheritance

Some extra things to consider and to show how inheritance can become
complex is to consider how you would use an abstract Boat class. Think about
what inheritance a hovercraft would have compared to a vehicle and boat, a
boat travels on water but a hovercraft can travel on both, which class should
the hovercraft inherit from? Possible solutions are using an interface, or
abstracting the vehicle class even more in order to properly encompass boats.

This scenario can become a problem when a ‘fragile base class’ can occur. As
a result the program could become over-abstracted. If theres a sudden change
you need to make at the base class level, it will create lots of problems with
your inheriting classes.

#### Inheritance in C\#

In C# you can only have 1 base class, this means you are effectively not
allowed multiple inheritances. But this can be subverted by having 1 base
class and multiple interfaces using it.
Functionally an interface and inheriting class are different, However from a
conceptual standpoint the two concepts are very similar. Ie, if you had an
abstract base class that implements no methods and you inherit from it, its
conceptually the same as an interface.

## Polymorphism

Polymorphism is a greek term meaning an object has the ability to take more
than 1 form.

Multiple methods have the same name, but have different behavior in
different instances. Functionally Polymorphism is the ability to process objects
differently based on their type/class.

#### Example of polymorphism

For example the '+' operator is overloaded in a way in programming where if
given an int on the LHS and RHS it knows to sum them. But if the + operator
has a string on the LHS and RHS it knows to concatenate them.

#### Overriding and Overloading

Overriding is known as runtime polymorphism. the method is determined at
runtime based on the dynamic type of the object. Its limitations are that you
cannot override non virtual or static method. This is a safety measure to limit
scope creep of private functions of a class.

Overloading is a type of compile time polymorphism. The compiler determines
which method will be run at compile time. This is determined by a process of
the compiler reading the method signatures.

### Conclusion

I hope this article helped anyone learning the principles of OOP. glhf

### Disclaimer

All work presented here should NOT be used by Swinburne students enrolled in COS20007, or any swinburne student for that matter without my permission.

COS20007 students should refer to the [Student Academic Misconduct Regulations 2012](https://www.swinburne.edu.au/about/leadership-governance/policies-regulations/statutes-regulations/student-academic-misconduct/) for guidelines about maintaining academic integrity.
Violation will result in sanctions or possibly expulsion for both of us, pls use responsibly.
