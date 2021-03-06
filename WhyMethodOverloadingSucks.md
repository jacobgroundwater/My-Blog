Here's the **tl;dr**

> never use method overloading because it sucks

Method overloading sucks in Scala because it actually detracts from your flexibility. 
An implicit conversion is a feature of Scala that let's the compiler 
look up how to convert objects between types at compile time.

Let's say I define a method with the following signature:

    def doSomething( action: Action )

Somewhere else in the code I write:

    val idea: Idea = // some idea
    doSomething( idea )
    

If `Idea` is a subtype of `Action` most languages will have no problem, including Scala. 
If `Idea` is not a subtype of `Action` this would be a comiple-time error in languages like Java. 
The Scala compiler, however, doesn't give up just yet. 
Instead of just throwing a type error, 
the Scala compiler searches for an *implicit conversion* between `Idea` and `Action`.

An implicit conversion is just a method marked `implicit` that has the right signature, in this case

    implicit def anyNameWillDo( idea: Idea ): Action
    

The only caveat is that the compiler must be able to *find* the conversion along a pre-determined search path. 
The compiler will check, in order, the local scopes, followed by the companion objects of `Action` and `Idea`. 
Only if no implicit conversion can be found will the compiler issue a type error.

### Overloading is Not the Answer ###

The problem with operator overloading, and why is sucks, 
is because overloaded methods interfere with implicit conversions. 
Let us overload the `doSomething` method to accept both an `Action` and `Idea`.

    def doSomething( action: Action )
    def doSomething( idea: Idea )
    
Now our implicit conversion, which we had full control over will no longer work. 
We're stuck with whatever conversion the overloaded method uses.

Adding new overloaded methods requires modifying the target class, which is not always possible. 

By using implicit conversions your code will look cleaner. 
The class containing the method `doSomething` need only implement the method that responds to an `Action`. 
All conversion code is located elsewhere in the code base.

Implicit conversions are determined at compile time.
Typically default conversions are in the companion objects or imported from a package.
If you don't like the default conversion, bring another implicit into a nearer scope than the default. 

Methods containing multiple arguments can mix and match conversions. Say we define:

    def doThreeThings( first: Action, second: Action, third: Action )
    

Implicit conversions let us do any of the following:

    doThreeThings( idea, idea, idea )
    doThreeThings( idea, idea, action )
    doThreeThings( idea, action, idea )
    doThreeThings( idea, action, action )
    doThreeThings( action, idea, idea )
    doThreeThings( action, idea, action )
    doThreeThings( action, action, idea )
    doThreeThings( action, action, action )
    

To accomplish the same thing with method overloading, we would require *eight* separate methods.

### Just Say No ###

As Tim points out below, since Scala supports default method arguments, we can avoid the following:

    doSomeThings( action: Action )
    doSomeThings( action: Action, action: Action )
    doSomeThings( action: Action, action: Action, action: Action )

Imagine how many methods we would need to define if we wanted to also accept `Idea` types! 
We would need 14 separate methods. 
