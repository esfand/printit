# Monads in Plain English #


> In terms that an OOP programmer would understand (without any functional programming background), what is a monad?

A monad is an *amplifier* of types that obeys certain rules and which has certain operations provided.

First, what is an **amplifier of types**? By that I mean some system which lets you take a 
type and turn it into a more special type. For example, in C# consider `Nullable<T>`. 
This is an amplifier of types. It lets you take a type, say int, and add a new capability 
to that type, namely, that now it can be null when it couldn't before.

As a second example, consider `IEnumerable<T>`. It is an amplifier of types. 
It lets you take a type, say, string, and add a new capability to that type, namely, 
that you can now make a sequence of strings out of any number of single strings.

What are the **certain rules**? Briefly, that there is a sensible way for functions on 
the underlying type to work on the amplified type such that they follow the normal rules 
of functional composition. For example, if you have a function on integers, say

```
int M(int x) { return x + N(x * 2); }
```

then the corresponding function on `Nullable<int>` can make all the operators and 
calls in there work together "in the same way" that they did before.

(That is incredibly vague and imprecise; you asked for an explanation that didn't 
assume anything about knowledge of **functional composition**.)

## What are the "operations"? ##

* That there is a way to take a value of an unamplified type and turn it into a value of the amplified type.
* That there is a way to transform operations on the unamplified type into operations on the amplified type 
  that obeys the rules of functional composition mentioned before
* That there is usually a way to get the unamplified type back out of the amplified type. 
  (This last point isn't strictly necessary for a monad but it is frequently the case that such an operation exists.)

Again, take `Nullable<T>` as an example. You can turn an int into a `Nullable<int> with the constructor. 
The C# compiler takes care of most nullable **lifting** for you, but if it didn't, 
the lifting transformation is straightforward: an operation, say,

```
int M(int x) { whatever }
```

is transformed into

```
Nullable<int> M(Nullable<int> x) { 
    if (x == null) 
        return null; 
    else 
        return new Nullable<int>(whatever);
}
```

And turning a nullable int back into an int is done with the Value property.

It's the function transformation that is the key bit. 
Notice how the actual semantics of the nullable operation -- 
that an operation on a null propagates the null -- is captured in the transformation. 
We can generalize this. Suppose you have a function from int to int, like our original M. 
You can easily make that into a function that takes an int and returns a `Nullable<int>` 
because you can just run the result through the nullable constructor. 
Now suppose you have this higher-order method:

```
Nullable<T> Bind<T>(Nullable<T> amplified, Func<T, Nullable<T>> func) {
    if (amplified == null) 
        return null;
    else
        return func(amplified.Value);
}
```

See what you can do with that? 
Any method that takes an int and returns an int, or takes an int and 
returns a nullable int can now have the nullable semantics applied to it.

Furthermore: suppose you have two methods

```
Nullable<int> X(int q) { ... }
Nullable<int> Y(int r) { ... }
```

and you want to compose them:

```
Nullable<int> Z(int s) { return X(Y(s)); }
```

That is, Z is the composition of X and Y. 
But you cannot do that because X takes an int, and Y returns a nullable int. 
But since you have the "bind" operation, you can make this work:

```
Nullable<int> Z(int s) { return Bind(Y(s), X); }
```

The bind operation on a monad is what makes composition of functions on amplified types work. 
The "rules" I handwaved about above are that the monad preserves the rules of 
normal function composition; that composing with identity functions results in 
the original function, that composition is associative, and so on.

In C#, "Bind" is called "SelectMany". 
Take a look at how it works on the sequence monad. We need to have three things: 
turn a value into a sequence, 
turn a sequence into a value, and 
bind operations on sequences. 
Those operations are:

```
IEnumerable<T> MakeSequence<T>(T item)
{
    yield return item;
}
T Single<T>(IEnumerable<T> sequence)
{
    // let's just take the first one
    foreach(T item in sequence) return item; 
}
IEnumerable<T> SelectMany<T>(IEnumerable<T> seq, Func<T, IEnumerable<T>> func)
{
    foreach(T item in seq)
        foreach(T result in func(item))
            yield return result;            
}
```

The nullable monad rule was "to combine two functions that produce nullables together, 
check to see if the inner one results in null; if it does, produce null, if it does not, 
then call the outer one with the result". That's the desired semantics of nullable. 
The sequence monad rule is "to combine two functions that produce sequences together, 
apply the outer function to every element produced by the inner function, and 
then concatenate all the resulting sequences together". 
The fundamental semantics of the monads are captured in the Bind/SelectMany methods; 
this is the method that tells you what the monad really means.

We can do even better. Suppose you have a sequences of ints, and a method that 
takes ints and results in sequences of strings. 
We could generalize the binding operation to allow composition of functions that 
take and return different amplified types, so long as the inputs of one match 
the outputs of the other:

```
IEnumerable<U> SelectMany<T,U>(IEnumerable<T> seq, Func<T, IEnumerable<U>> func)
{
    foreach(T item in seq)
        foreach(U result in func(item))
            yield return result;            
}
```

So now we can say "amplify this bunch of individual integers into a sequence of integers. 
Transform this particular integer into a bunch of strings, amplified to a sequence of strings. 
Now put both operations together: amplify this bunch of integers into the concatenation of 
all the sequences of strings." Monads allow you to compose your amplifications.

> What problem does it solve and what are the most common places it's used?

That's rather like asking "what problems does the singleton pattern solve?", 
but I'll give it a shot.

Monads are typically used to solve problems like:

* I need to make new capabilities for this type and still combine old functions on this type to use the new capabilities.
* I need to capture a bunch of operations on types and represent those operations as composable objects, building up larger and larger compositions until I have just the right series of operations represented, and then I need to start getting results out of the thing
* I need to represent side-effecting operations cleanly in a language that hates side effects

(Note that these are basically three ways of saying the same thing.)

C# uses monads in its design. As already mentioned, the nullable pattern is highly akin to the "maybe monad". 
LINQ is entirely built out of monads; 
the "SelectMany" method is what does the semantic work of composition of operations. 
(Erik Meijer is fond of pointing out that 
every LINQ function could actually be implemented by SelectMany; 
everything else is just a convenience.)

> To clarify the kind of understanding I was looking for, 
> let's say you were converting an FP application that had monads into an OOP application. 
> What would you do to port the responsibilities of the monads into the OOP app?

Most OOP languages do not have a rich enough type system to represent the monad pattern itself directly; you need a type system that supports types that are higher types than generic types. So I wouldn't try to do that. Rather, I would implement generic types that represent each monad, and implement methods that represent the three operations you need: turning a value into an amplified value, turning an amplified value into a value, and transforming a function on unamplified values into a function on amplified values.

A good place to start is how we implemented LINQ in C#. 
Study the SelectMany method; it is the key to understanding how the sequence monad works in C#. 
It is a very simple method, but very powerful!

For a more in-depth and theoretically sound explanation of monads in C#, 
I highly recommend my colleague Wes Dyer's article on the subject. 
This article is what explained monads to me when they finally "clicked" for me.

The Marvels of Monads
