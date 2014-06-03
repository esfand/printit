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


Optional was meant only as a return type in methods that might return null, and not being Serializable is intentional and as per design:

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

All it takes be a Monad is to provide two functions which conform to three laws.

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
  * 
Please see the above gist for a java demonstration of the three laws.

NOTE: One of the key things to understand is the signature of the function to 
apply in monadic context: it takes the raw value type, and returns the monadic type.

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
 
 


