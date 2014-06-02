# The Marvels of Monads#

If the word "continuation" causes eyes to glaze over, then the word "monad" 
induces mental paralysis.  Perhaps, this is why some have begun inventing 
more benign names for monads.

These days, monads are the celebrities of programming language theory.  
They gloss the cover of blogs and have been compared to everything from 
boxes of fruit to love affairs.  Nerds everywhere are exclaiming that 
the experience of understanding monads causes a pleasantly painful mental 
sensation.

Like continuations, monads are simpler than they sound and are very useful 
in many situations.  In fact, programmers write code in a variety of 
languages that implicitly use common monads without even breaking a sweat.

With all of the attention that monads get, why am I writing yet another 
explanation of monads?  Not to compare them to some everyday occurrence or 
to chronicle my journey to understanding.  I explain monads because I need 
monads.  They elegantly solve programming problems in a number of languages 
and contexts.

## Introducing Monads##

Monads come from category theory.  
Moggi introduced them to computer scientists to aid in the analysis of the 
semantics of computations.  
In an excellent paper, The Essence of Functional Programming, Wadler showed 
that monads are generally useful in computer programs to compose together 
functions which operate on **amplified values** rather than **values**.  
Monads became an important part of the programming language Haskell where 
they tackle the awkward squad: IO, concurrency, exceptions, and 
foreign-function calls.

Monads enjoy tremendous success in Haskell, but like an actor who does well in 
a particular role, monads are now stereotyped in the minds of most programmers 
as useful only in pure lazy functional languages.  
This is unfortunate, because monads are more broadly applicable.

## Controlling Complexity ##

Composition is the key to controlling complexity in software.  
In The Structure and Interpretation of Computer Programs, Abelson and Sussman 
argue that composition beautifully expresses complex systems from simple patterns.

In our study of program design, we have seen that expert programmers control 
the complexity of their designs with the same general techniques used by 
designers of all complex systems. They combine primitive elements to form 
compound objects, they abstract compound objects to form higher-level 
building blocks, and they preserve modularity by adopting appropriate 
large-scale views of system structure.

One form of composition, function composition, succinctly describes the dependencies 
between function calls.  
Function composition takes two functions and plumbs the result from the second 
function into the input of the first function, thereby forming one function.

```java
public static Func<T, V> Compose<T, U, V>(this Func<U, V> f, Func<T, U> g) {
    return x => f(g(x));
}
```

For example, instead of applying g to the value x and then applying f to the result, 
compose f with g and then apply the result to the value x.  
The key difference is the abstraction of the dependency between f and g.

```java
var r = f(g(x));         // without function composition
var r = f.Compose(g)(x); // with function composition
```

Given the function Identity, function composition must obey three laws.

```java
public static T Identity<T>(this T value) {
    return value;
}
```

1.  **Left identity**

     `Identity.Compose(f) = f`

2.  **Right identity**

     `f.Compose(Identity) = f`

3.  **Associative**

     `f.Compose(g.Compose(h)) = (f.Compose(g)).Compose(h)`

Often, values are not enough.  Constructed types amplify values.  
The type `IEnumerable<T>` represents a lazily computed list of values of type `T`.  
The type `Nullable<T>` represents a possibly missing value of type `T`.  
The type `Func<Func<T, Answer>, Answer>` represents a function, which returns an 
`Answer` given a continuation, which takes a `T` and returns an `Answer`.  
Each of these types **amplifies** the type `T`.

Suppose that instead of composing functions which return values, 
we compose functions which take values and return amplified values.  
Let `M<T>` denote the type of the **amplified values**.

```java
public static Func<T, M<V>> Compose<T, U, V>(this Func<U, M<V>> f, Func<T, M<U>> g) {
    return x => f(g(x)); // error, g(x) returns M<U> and f takes U
}
```

Function composition fails, because the return and input types do not match.  
Composition with amplified values requires a function that accesses the underlying 
value and feeds it to the next function.  Call that function `Bind` and use it 
to define function composition.

```java
public static Func<T, M<V>> Compose<T, U, V>(this Func<U, M<V>> f, Func<T, M<U>> g) {
    return x => Bind(g(x), f);
}
```

The input and output types determine the signature of `Bind`.  
Therefore, `Bind` takes an **amplified value**, `M<U>`, and a function from `U` to `M<V>`, 
and returns an **amplified value**, `M<V>`.

```java
public static M<V> Bind<U, V>(this M<U> m, Func<U, M<V>> k)
```

The body of Bind depends on the mechanics of the amplified values, `M<T>`.  
Each **amplified type** will need a distinct definition of `Bind`.

In addition to `Bind`, define a function which takes an **unamplified value** 
and amplifies it.  Call this function `Unit`.

```java
public static M<T> Unit<T>(this T value)
```

Together the amplified type, `M<T>`, the function `Bind`, and the function `Unit` 
enable function composition with amplified values.

## Meet the Monads ##

Viola, we have invented monads.

> Monads are a triple consisting of a type, a Unit function, and a Bind function.  
> Furthermore, to be a monad, the triple must satisfy three laws:

1.  **Left Identity**

     Bind(Unit(e), k) = k(e)

2.  **Right Identity**

     Bind(m, Unit) = m

3.  **Associative**

     Bind(m, x => Bind(k(x), y => h(y)) = Bind(Bind(m, x => k(x)), y => h(y))

The laws are similar to those of function composition.  This is not a coincidence.  
They guarantee that the monad is well behaved and composition works properly.

To define a particular monad, the writer supplies the triple, thereby specifying 
the mechanics of the amplified values.

## The Identity Monad ##

The simplest monad is the Identity monad.  The type represents a wrapper containing a value.

```java
class Identity<T> {
    public T Value { get; private set; }
    public Identity(T value) { this.Value = value; }
}
```

The **Unit function** takes a value and returns a new instance of `Identity`, which wraps the value.

```java
static Identity<T> Unit<T>(T value) {
    return new Identity<T>(value);
}
```

The bind function takes an instance of `Identity`, unwraps the value, and invokes the 
**delegate**, `k`, with the unwrapped value.  The result is a new instance of `Identity`.

```java
static Identity<U> Bind<T,U>(Identity<T> id, Func<T,Identity<U>> k) {
    return k(id.Value);
}
```

Consider a simple program that creates two Identity wrappers and performs an operation 
on the wrapped values.  First, bind x to the value within the wrapper containing the 
value five.  Then, bind y to the value within the wrapper containing the value six.  
Finally, add the values, x and y, together.  The result is an instance of Identity 
wrapping the value eleven.

```java
var r = Bind(Unit(5), x => 
            Bind(Unit(6), y => 
                Unit(x + y))); 

Console.WriteLine(r.Value);
```

While this works, it is rather clumsy.  It would be nice to have syntax for dealing 
with the monad.  Fortunately, we do.

C# 3.0 introduced query comprehensions which are actually monad comprehensions in disguise.  
We can rewrite the identity monad to use **LINQ**.  Perhaps, it should have been called 
LINM (**Language INtegrated Monads**), but it just doesn't have the same ring to it.

Rename the method `Unit` to `ToIdentity` and `Bind` to `SelectMany`.  
Then, make them both extension methods.

```java
public static Identity<T> ToIdentity<T>(this T value) {
    return new Identity<T>(value);
}

public static Identity<U> SelectMany<T, U>(this Identity<T> id, Func<T, Identity<U>> k) {
    return k(id.Value);
}
```

The changes impact the calling code.

```
var r = 5.ToIdentity().SelectMany(
            x => 6.ToIdentity().SelectMany(
                y => (x + y).ToIdentity()));

Console.WriteLine(r.Value);
```

Equivalent methods are part of the standard query operators defined for LINQ.  
However, the standard query operators also include a slightly different version 
of SelectMany for performance reasons.  It combines `Bind` with `Unit`, 
so that lambdas are not deeply nested.  The signature is the same except for an 
extra argument that is a delegate which takes two arguments and returns a value.  
The delegate combines the two values together.  
This version of `SelectMany` binds `x` to the wrapped value, applies `k` to `x`, 
binds the result to `y`, and then applies the combining function, `s`, to `x` and `y`.  
The resultant value is wrapped and returned.

```java
public static Identity<V> SelectMany<T, U, V>(this Identity<T>     id, 
                                              Func<T, Identity<U>> k, 
                                              Func<T,U,V>          s) {

    return id.SelectMany(x => k(x).SelectMany(y => s(x, y).ToIdentity()));
}
```

Of course, we can remove some of the code from the generalized solution by using 
our knowledge of the **Identity monad**.

public static Identity<V> SelectMany<T, U, V>(this Identity<T>      id, 
                                              Func<T, Identity<U>>  k, 
                                              Func<T,U,V>           s) {

    return s(id.Value, k(id.Value).Value).ToIdentity();
}
```

The call-site does not need to nest lambdas.

```java
var r = 5.ToIdentity()
         .SelectMany(x => 6.ToIdentity(), (x, y) => x + y);

Console.WriteLine(r.Value);
```

With the new definition of `SelectMany`, programmers can use C#'s **query comprehension syntax**.  
The `from` notation binds the introduced variable to the value wrapped by the expression on the right.  
This allows subsequent expressions to use the wrapped values without directly calling `SelectMany`.

```java
var r = from x in 5.ToIdentity()
        from y in 6.ToIdentity()
        select x + y;
```

Since the original `SelectMany` definition corresponds directly to the 
**monadic Bind function** and because the existence of a generalized transformation 
has been demonstrated, the remainder of the post will use the original signature.  
But, keep in mind that the second definition is the one used by the query syntax.

## The Maybe Monad ##

The Identity monad is an example of a monadic container type where the Identity monad wrapped a value.  
If we change the definition to contain either a value or a missing value then we have the Maybe monad.

Again, we need a type definition.  
The Maybe type is similar to the Identity type but adds a property denoting whether a value is missing.  
It also has a predefined instance, Nothing, representing all instances lacking a value.

```java
class Maybe<T> {
    public readonly static Maybe<T> Nothing = new Maybe<T>();

    public T Value { get; private set; }

    public bool HasValue { get; private set; }

    Maybe() {
        HasValue = false;
    }

    public Maybe(T value) {
        Value = value;
        HasValue = true;
    }
}
```

The Unit function takes a value and constructs a Maybe instance, which wraps the value.

```java
public static Maybe<T> ToMaybe<T>(this T value) {
    return new Maybe<T>(value);
}
```

The *Bind function* takes a `Maybe` instance and if there is a value then it applies the 
delegate to the contained value.  Otherwise, it returns Nothing.

```java
public static Maybe<U> SelectMany<T, U>(this Maybe<T> m, Func<T, Maybe<U>> k) {
    if (!m.HasValue)
        return Maybe<U>.Nothing;
    return k(m.Value);
}
```

The programmer can use the comprehension syntax to work with the Maybe monad.  
For example, create an instance of Maybe containing the value five and add it to Nothing.

```java
var r = from x in 5.ToMaybe()
        from y in Maybe<int>.Nothing
        select x + y;

Console.WriteLine(r.HasValue ? r.Value.ToString() : "Nothing");
```

The result is "Nothing".  
We have implemented the null propagation of nullables without explicit language support.

## The List Monad ##

Another important container type is the `list` type.  
In fact, the list monad is at the heart of LINQ.  
The type `IEnumerable<T>` denotes a lazily computed `list`.

The *Unit function* takes a value and returns a list, which contains only that value.

```java
public static IEnumerable<T> ToList<T>(this T value) {
    yield return value;
}
```

The *Bind function* takes 
an `IEnumerable<T>`, 
a delegate, which takes a `T` and returns an `IEnumerable<U>`, and
returns an `IEnumerable<U>`.

```java
public static IEnumerable<U> SelectMany<T, U>(this IEnumerable<T> m, 
                                              Func<T, IEnumerable<U>> k) {
    foreach (var x in m)
        foreach (var y in k(x))
            yield return y;
}
```

Now, the programmer can write the familiar query expressions with IEnumerable<T>.

```
var r = from x in new[] { 0, 1, 2 }
        from y in new[] { 0, 1, 2 }
        select x + y;

foreach (var i in r)
    Console.WriteLine(i);
```

Remember that it is the monad that enables the magic.

## The Continuation Monad ##

The continuation monad answers the question that was posed at the end of the last post: 
how can a programmer write CPS code in a more palatable way?

The type of the continuation monad, `K`, is a delegate which when given a continuation, 
which takes an argument and returns an answer, will return an answer.

```java
delegate Answer K<T,Answer>(Func<T,Answer> k);
```

The type K fundamentally differs from types `Identity<T>`, `Maybe<T>`, and `IEnumerable<T>`.  
All the other monads represent container types and allow computations to be specified 
in terms of the values rather than the containers, but the continuation monad contains nothing.  
Rather, it composes together continuations the user writes.

**To be a monad**, there must be a Unit function which takes a `T` and returns a `K<T,Answer>` 
for some answer type.

```java
public static K<T, Answer> ToContinuation<T, Answer>(this T value)
```

What should it do?  When in doubt, look to the types.   
The method takes a T and returns a function, which takes a function from T to Answer, 
and returns an Answer.  
Therefore, the method must return a function and the only argument of that function 
must be a function from T to Answer.  Call the argument c.

return (Func<T, Answer> c) => ...
The body of the lambda must return a value of type Answer.  Values of type Func<T,Answer> and a T are available.  Apply c to value and the result is of type Answer.

```java
return (Func<T, Answer> c) => c(value);
```

To be a monad, Bind must take a `K<T,Answer>` and a function from `T` to `K<U, Answer>` and return a `K<U, Answer>`.

```java
public static K<U, Answer> SelectMany<T, U, Answer>(this K<T, Answer> m, Func<T, K<U, Answer>> k)
```

But what about the body?  The result must be of type `K<U, Answer>`, 
but how is a result of the correct type formed?

Expand K's definition to gain some insight.

return type

     Func<Func<U, Answer>, Answer>

m's type

     Func<Func<T, Answer>, Answer>

k's type

     Func<T, Func<Func<U, Answer>, Answer>>

Applying k to a value of type `T` results in a value of type `K<U,Answer>`, 
but no value of type `T` is available.  
Build the return type directly by constructing a function, 
which takes a function from U to Answer.  
Call the parameter c.

```java
return (Func<U,Answer> c) => ...
```

The body must be type of `Answer` so that the return type of Bind is `K<U,Answer>`.  
Perhaps, `m` could be applied to a function from `T` to Answer.  The result is a value of type `Answer`.

```java
return (Func<U,Answer> c) => m(...)
```

The expression inside the invocation of `m` must be of type `Func<T,Answer>`.  
Since there is nothing of that type, construct the function by creating a 
lambda with one parameter, `x`, of type `T`.

```java
return (Func<U,Answer> c) => m((T x) => ...)
```

The body of this lambda must be of type `Answer`.  
Values of type `T`, `Func<U,Answer>`, and `Func<T,Func<Func<U,Answer>, Answer>>` 
haven't been used yet.  Apply `k` to `x`.

```java
return (Func<U,Answer> c) => m((T x) => k(x)...)
```

The result is a value of type `Func<Func<U,Answer>,Answer>`.  Apply the result to `c`.

```java
return (Func<U,Answer> c) => m((T x) => k(x)(c));
```

The continuation monad turns the computation inside out.  
The comprehension syntax can be used to construct continuations.

Construct a computation, which invokes a continuation with the value seven.  
Pass this computation to another computation, which invokes a continuation with the value six.   
Pass this computation to another computation, which invokes a continuation with the result of 
adding the results of the first two continuations together.  
Finally, pass a continuation, which replaces "1"s with "a"s, to the result.

```java
var r = from x in 7.ToContinuation<int,string>()
        from y in 6.ToContinuation<int,string>()
        select x + y;

Console.WriteLine(r(z => z.ToString().Replace('1', 'a'))); // displays a3
```

The continuation monad does the heavy-lifting of constructing the continuations.

## Monadic Magic ##

Beautiful composition of amplified values requires monads.  
The Identity, Maybe, and IEnumerable monads demonstrate the power of monads as container types.  
The continuation monad, K, shows how monads can readily express complex computation.

