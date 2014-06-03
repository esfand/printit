# Java8 Optional #

Calling `isPresent()` is not the way Optional is intended to be used.

Optional is a **monad** because it has 
* a unit method (Optional.of()) and 
* a flatMap() method. 

It is intended to be used mainly for composing functions returning Optional 
by chaining calls such as:

```Java
someOptional.flatMap(SomeClass::someMethodReturningOptional)
            .flatMap(OtherClass::otherMethodReturningOptional)
            .ifPresent(YetAnotherClass::yetAnotherMethod);
```

Methods that always return a value does not need to return Optional and 
must be **composed with** `map()` instead of `flatMap()`.
  
The last method in the chain (yetAnotherMethod) is an "effect" taking the 
enclosed value (if present) and returning void. 
In Java 8, **effects** are called `Consumer`.

`isPresent()` is there only to allow non functional developers to use Optional 
with the imperative paradigm.
