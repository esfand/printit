# Java8 Optional #

Calling `isPresent()` is not the way Optional is intended to be used.

Optional is a **monad** because it has 
* a unit method `Optional.ofNullable(T value)` and 
* a `flatMap(Function<? super T, Optional<U>> mapper)` method. 

It is intended to be used mainly for **composing functions** returning Optional 
by chaining calls such as:

```Java
someOptional.flatMap(SomeClass::someMethodReturningOptional)
            .flatMap(OtherClass::otherMethodReturningOptional)
            .ifPresent(YetAnotherClass::yetAnotherMethod);
```

Methods that always return a value does not need to return Optional and 
must be **composed with** `map()` instead of `flatMap()`.
  
The last method in the chain `yetAnotherMethod` is an **effect** taking the 
enclosed value (if present) and returning void. 
In Java 8, **effects** are called `Consumer`.

`isPresent()` is there only to allow non functional developers to use Optional 
with the imperative paradigm.

Optional is only meant to be used when the absence of value is not an error.
Optional should NEVER be used to replace throwing an exception.
Optional is meant to allow composing functions that may or may not return a value.

To compose functions that may return a value or throw an exception, we need an somethings else. 
In Scala, it is called Try, and like Optional, it is a monad.

For more information about the (absence of) Try monad in Java 8 
(along with how to use the Optional monad), see my post:
http://java.dzone.com/articles/whats-wrong-java-8-part-iv

Optional is one of the many things that were made wrong in Java 8.

- It should of course be Serializable.

- Method of() is misleading. 
  It does two things: 
  * Throwing an exception (if null) or 
  * returning an Optional (if not null). Use ofNullable() instead. 
    But this is misleading, because it makes people think that 
    the normal case is of() and ofNullable() is a special case.

- The real good with Optional is that it allows composing functions 
  not only when they return a value, but also when they would throw 
  an exception or return a null pointer. Instead, the function returns 
  an empty Option and this result may be composed with a another function. 
  In other word, it allows transforming partial functions into total ones. 
  However, this is not the only way to go. A better type system would also 
  allow transforming partial functions into total ones by shrinking their domain. 
  For example the square root function is partial on integers and would be total 
  if returning an Optional (empty if the argument is negative). 
  But the function would also be total if applied to non negative integers.

- The bad thing is we seldom need Optional, because Option is good for cases 
  where we should do nothing if there is no value. Much more useful is the 
  Try monad that holds either a value or an exception, allowing composing 
  function and carrying any exception until the end of the calculation.
<br/>
  It is very easy to write our own Try monad and use it in our programs. 
  But the big problem is that we can't use it in public APIs because we would end 
  with as many incompatible Try implementations as there are APIs!


Optional was meant only as a return type in methods that might return null, 
and not being Serializable is intentional and as per design:

> The JSR-335 EG felt fairly strongly that Optional should not be on any 
> more than needed to support the optional-return idiom only.

If used as intended it's still a useful thing to have, just not as powerful as in other languages.
Maybe the name OptionalReturn would have been a better choice.

You should use Optionals in your data model: to leverage the type system to 
clearly differentiate 
* the values that can be missing (the car in the Person class and 
  the insurance in the Car class in my example), from 
* the ones that must be there (the name of the insurace company). 

In this way you know, through the type system, that 
* a person without a car is acceptable in your data model while 
* an insurance company without a name isn't.


http://stackoverflow.com/questions/13454347/monads-with-java-8

Just FYI:

The proposed JDK8 Optional class does satisfy the three Monad laws. 
https://gist.github.com/esfand/66cda4876cb04c132901 is a gist demonstrating that.

All it takes to be a Monad is to provide two functions which conform to three laws.

The two functions:

1. Place a value into monadic context

  * Haskell's Maybe: return / Just
  * Scala's Option: Some
  * Functional Java's Option: Option.some
  * JDK8's Optional: Optional.of

2. Apply a function in monadic context

  * Haskell's Maybe: >>= (aka bind)
  * Scala's Option: flatMap
  * Functional Java's Option: flatMap
  * JDK8's Optional: flatMap

Please see the above gist for a java demonstration of the three laws.

NOTE: One of the key things to understand is the signature of the function to 
apply in monadic context: it takes the **raw value type**, and returns the **monadic type**.

In other words, if you have `Optional<Integer>`, the functions you can pass to 
`flatMap` will have the signature `(Integer) -> Optional<U>`, where U is a value type 
which does not have to be Integer, for example String:

```java
Optional<Integer> maybeInteger = Optional.of(1);

// Function that takes Integer and returns Optional<Integer>
Optional<Integer> maybePlusOne 
        = maybeInteger.flatMap(n -> Optional.of(n + 1));

// Function that takes Integer and returns Optional<String>
Optional<String> maybeString 
        = maybePlusOne.flatMap(n -> Optional.of(n.toString));
```

You don't need any sort of Monad Interface to code this way, or to think this way. 
In Scala, you don't code to a Monad Interface (unless you are using Scalaz library...).
It appears that JDK8 will empower Java folks to use this style of 
**chained monadic computations** as well.

Hope this is helpful!

In theory Monad is an abstraction of a computation process in some context. 
For example, Option Monad is a context for abstracting over some possible 
computation, i.e you can have a couple of functions which can return either 
Some value or None and you can build a computation where each step depends 
on the previously computed value, or there are a Future monad, 
which describes a context of some defered computations, IO monad, 
State monad, etc =) – 
 
 
<hr/>


This is pretty cool. It's a bit abstract though. I can imagine people who don't know what 
monads are already get confused due to the lack of real examples.

So let me try to comply, and just to be really clear I'll do an example in C#, even though 
it will look ugly. I'll add the equivalent Haskell at the end and show you the cool Haskell 
syntactic sugar which is where, IMO, monads really start getting useful.

Okay, so one of the easiest Monads it called the "Maybe monad" in Haskell. 
In C# the Maybe type is called Nullable<T>. It's basically a tiny class that just encapsulates 
the concept of a value that is either valid and has a value, or is "null" and has no value.

A useful thing to stick inside a monad for combining values of this type is the notion of failure. 
I.e. we want to be able to look at multiple nullable values and return "null" as soon as any one 
of them is null. This could be useful if you, for example, look up lots of keys in a dictionary 
or something, and at the end you want to process all of the results and combine them somehow, 
but if any of the keys are not in the dictionary, you want to return "null" for the whole thing. 
It would be tedious to manually have to check each lookup for "null" and return, so we can hide 
this checking inside the bind operator (which is sort of the point of monads, 
we hide book-keeping in the bind operator which makes the code easier to use since we can forget 
about the details).


Here's the program that motivates the whole thing (I'll define the Bind later, this just to show you why it's nice).


```java
class Program {
 
        static Nullable<int> f(){ return 4; }        
        static Nullable<int> g(){ return 7; }
        static Nullable<int> h(){ return 9; }
        

        static void Main(string[] args) {
      
            Nullable<int> z = f().Bind( fval => 
                              g().Bind( gval => 
                              h().Bind( hval =>
                              new Nullable<int>( fval + gval + hval ))));

            Console.WriteLine("z = {0}", z.HasValue ? z.Value.ToString() : "null" );
            Console.WriteLine("Press any key to continue...");
            Console.ReadKey();
        }
}

Now, ignore for a moment that there already is support for doing this for Nullable in C# 
(you can add nullable ints together and you get null if either is null). Let's pretend 
t there is no such feature, and it's just a user-defined class with no special magic. 
The point is that we can use the Bind function to bind a variable to the contents of 
our Nullable value and then pretend that there's nothing strange going on, and use them 
like normal ints and just add them together. We wrap the result in a nullable at the end, 
and that nullable will either be null (if any of f, g or h returns null) or it will be 
the result of summing f, g, and h together. (this is analogous of how we can bind a row 
in a database to a variable in LINQ, and do stuff with it, safe in the knowledge that 
the Bind operator will make sure that the variable will only ever be passed valid row values).

You can play with this and change any of f, g, and h to return null and you will see that
the whole thing will return null.

So clearly the bind operator has to do this checking for us, and bail out returning null 
if it encounters a null value, and otherwise pass along the value inside 
the Nullable structure into the lambda.

Here's the Bind operator:

```java
        public static Nullable<B> Bind<A,B>( this Nullable<A> a, Func<A,Nullable<B>> f ) 
            where B : struct 
            where A : struct
        {
            return a.HasValue ? f(a.Value) : null;
        }
```

The types here are just like in the video. It takes an `M a` (`Nullable<A>` in C# syntax for this case), 
and a function from `a` to `M b` (`Func<A, Nullable<B>>` in C# syntax), and it returns an `M b` (`Nullable<B>`).
The code simply checks if the nullable contains a value and if so extracts it and passes it onto the function, 
else it just returns `null`. This means that the Bind operator will handle all the null-checking logic for us.
If and only if the value that we call Bind on is non-null then that value will be **passed along** to 
the lambda function, else we bail out early and the whole expression is null.
This allows the code that we write using the monad to be entirely free of this null-checking behaviour, 
we just use Bind and get a variable bound to the value inside the **monadic value**  
(fval, gval and hval in the example code) and we can use them safe in the knowledge that 
Bind will take care of checking them for null before passing them along.


There are other examples of things you can do with a monad. For example you can make the Bind operator 
take care of an input stream of characters, and use it to write parser combinators. Each parser combinator 
can then be completely oblivious to things like back-tracking, parser failures etc., and just combine 
smaller parsers together as if things would never go wrong, safe in the knowledge that a clever 
implmenetation of Bind sorts out all the logic behind the difficult bits. 
Then later on maybe someone adds logging to the monad, but the code using the monad doesn't change, 
because all the magic happens in the definition of the Bind operator, the rest of the code is unchanged.

Finally, here's the implemenation of the same code in Haskell (-- begins a comment line).

```haskell
-- Here's the data type, it's either nothing, or "Just" a value
-- this is in the standard library
data Maybe a = Nothing | Just a

-- The bind operator for Nothing
Nothing >>= f = Nothing
-- The bind operator for Just x
Just x >>= f = f x

-- the "unit", called "return"
return = Just

-- The sample code using the lambda syntax
-- that Brian showed
z = f >>= ( \fval ->
     g >>= ( \gval ->  
     h >>= ( \hval -> return (fval+gval+hval ) ) ) )

-- The following is exactly the same as the three lines above
z2 = do 
   fval <- f
   gval <- g
   hval <- h
   return (fval+gval+hval)
```

As you can see the nice "do" notation at the end makes it look like straight imperative code. 
And indeed this is by design. Monads can be used to encapsulate all the useful stuff 
in imperative programming (mutable state, IO etc.) and used using this nice imperative-like syntax, 
but behind the curtains, it's all just monads and a clever implementation of the bind operator!
The cool thing is that you can implement your own monads by implemnting >>= and return. 
And if you do so those monads will also be able to use the do notation, 
which means you can basically write your own little languages by just defining two functions!


