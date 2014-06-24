# Monads - Another way to abstract computations in Scala #

You can view **monads** as **containers** or as **computations**. Isn't this confusing enough ?

To a programmer, the biggest scare with the name **monad** is the fact that it originates from 
the constructs of category theory and the rich mathematics that goes along with it. 
The simplest way to make an idea of monads is to look at applications and examples that use them 
to solve real life problems. 
Monads have been described as models of computation, code transformers, state transformers and what not. 
But unfortunately for someone uninitiated to monadic programming, and indoctrinated with the tenets of 
OO programming and design patterns, all these incarnations make little sense. 
Sometime back, after listening to a long dissertation on monads, I asked myself .. 
what the heck do I need monads for? I can very well do the same thing with a **Composite Command** design pattern 
that serializes execution of its composing commands.

I have been playing around with Scala for some time now. Scala is a multiparadigm language. 
The nice thing about Scala is that you never feel out of the way writing either imperative or functional code. 
On one hand, you have the re-assignable vars, explicit IOs, side-effecting constructs. 
And on the other, you can write pure clean functional code without any side-effecting features. 
Coming from a Java / C++ background and seeing enough of OO over the last decade, my interest is in 
exploring the functional features that the language has to offer. 
Ok, I know you will refer me to the backyards of Haskell or OCamL as more pure functional languages. 
But hey, I need to have my appplication deployed on the JVM, and don't ask me why .. 

This post is all about my exposure to monads in Scala. 
This is not meant to be a tutorial on monads, considered by some, to be the stairway to the heaven of Haskell. 
It looks at monads purely as a means to design complex **computational abstractions** 
using the phenomenal powers of **closures** and **higher order functions** in Scala. 
Functional programming is all about **referential transparency**, where you treat your functions algebraically, 
evaluating them in no specific order. 
Monads allow ordered computation within FP that allows us to model **sequencing of actions** in a nice 
structured form, somewhat like a DSL. 
And the greatest power comes with the ability **to compose monads that serve different purposes**, 
into extensible abstractions within an application. 

This sequencing and threading of actions by a monad is done by the language compiler that does 
the transformation through the magic of **closures**. 
Scala provides some built-in support of monads, and offers the machinery to design your own **monadic operations**. 

Consider the following Scala code fragment, which looks quite intuitive to anyone familiar 
with programming in a high level language :

```scala
for {
  x <- List(1, 2)
}
yield(x + 2)
```

The code snippet uses Scala **for-comprehensions** to perform a `+ 2` operation 
on individual elements of a list of integers. 
Scala for-comprehension, being a syntactic sugar, the compiler does the heavy lifting and converts 
it to a more traditional **map operation** ..

```scala
List(1, 2) map {x => x + 2}
```

Quite trivial .. huh ! But how does it relate to monads ? Hang on ..

Now how about this ?

```scala
val first = List(1, 2)
val next  = List(8, 9)

for {
    i <- first
    j <- next
}
yield(i * j)
```

I have added an extra step of **computation** in the **sequence**, that gets resolved as ..

```scala
first flatMap {
    f => next map {
        n => f * n
    }
}
```

and the last one ..

```scala
val first = List(1, 2)
val next = List(8, 9)
val last = List("ab", "cde", "fghi")

for {
    i <- first
    j <- next
    k <- last
}
yield(i * j * k.length)
```

that transforms to :

```scala
first flatMap {
    i => next flatMap {
        j => last map {
            k => i * j * k.length
        }
    }
}
```

The key abstraction is the`flatMap`, which **binds the computation through chaining**. 
Each invocation of flatMap returns the same data structure **type** (but of different **value**), 
that serves as the input to the next command in chain.
In the above snippet, `flatMap` takes as input a **closure** `(SomeType) => List[AnotherType]` and 
returns a `List[AnotherType]`. 
The important point to note is that all `flatMap`s take the same **closure type** as input and 
return the same **type** as output. 
This is what **binds** the computation thread - 
every item of the sequence in the **for-comprehension** 
has to honor this same **type constraint**.

The above is an example of the **List monad** in Scala. 
Lists support filtering, and so does the List monad. 
In the for-comprehension, we can also filter data items through the **if-guards** :

```scala
for {
    i <- 1 until n
    j <- 1 until (i-1)
    if isPrime(i+j)
}
yield (i, j)
```

Consider another different computation involving sequencing of operations (not of the List type though) 
from an example which I mentioned in my previous Scala post :

```scala
case class Order(lineItem: Option[LineItem])
case class LineItem(product: Option[Product])
case class Product(name: String)

for {
    order    <- maybeOrder
    lineItem <- order.lineItem
    product  <- lineItem.product
}
yield product.name
```

This gets me the product name from the **chain** of order, lineItem and product. 
One more sequencing of operations, and the for-comprehension gets converted to :

```scala
maybeOrder flatMap {
    order => order.lineItem flatMap {
        lineItem => lineItem.product map {
            product => product.name
        }
    }
}
```

**flatMap again!**

Once again we have the magic of `flatMap` **binding the thread of computation**. 
And similar to the earlier example of the List monad, 
every `flatMap` in the entire thread is **homogenously typed** - 
input type being `(T => Option[U])` and 
the output type being `Option[U]`. 
The types participating in this sequence of computation is the **Maybe monad** type, 
modeled as `Option[T]` in Scala.

What is the commonality of the above two examples ? 

It is the 
> **sequencing of operations**, that leads to the evolution of higher order functional abstractions.


And what is the variability part ?

It is 
> **the types that actually take part in the operations**. 
In the first case, it is the operations on List type that gets chained. 
While in the second case, it is the Option type.


And what is the secret sauce ?

The `flatMap` (aka bind in Haskell) operation, which works orthogonally across types and 
serves as the generic **binder** of the **sequence of actions**.

In the above examples, the two monad types discussed, `List[T]` and `Option[T]` are **container types**, 
if we consider the latter to be a degenerate version with only 2 elements. 
However, monads can be designed to be of other types as well 
e.g. those that work on **state manipulation** or on **doing IO**. 
And for these monads, possibly it is more intuitive to comprehend **monads as computations**. 
I hope to provide examples of some real life applications of **State monad** and **IO monad** 
using Scala in a future post.

## Whole is bigger than the sum of its parts ##

As I mentioned earlier, the biggest power of monads is 
the ability **to combine diverse monadic operations** to design modular and extensible code. 
The following snippet gives an example that combines a List monad and Maybe monad 
within the same for-comprehension block :

```scala
val list     = List("India", "Japan", "France", "Russia")
val capitals = Map("India" -> "New Delhi", "Japan" -> "Tokyo", "France" -> "Paris")

for {
    i <- list
    j <- capitals get(i) orElse(Some("None"))
}
yield(j)
```

The first operation of the sequence is one on a List monad, while the next one is on a Maybe monad. 
The syntactic sugar of the for-comprehensions abstracts the details nicely enough for the user, 
who is completely oblivious of the underlying machinery of binding monads. 
Here is what comes up after the code transformation :

```scala
list flatMap {
    i => capitals.get(i).orElse(Some("None")) map {
        j => j
    }
}
```

Developing modular software is all about working at the right level of abstraction. 
And monads offer yet another machinery to mix and match the right abstractions within your codebase. 
Consider adding the power of monads to your toolbox - 
they certainly are one of the potent design patterns that you will ever need.

## Comments ##

**Luc Duponcheel said...**

**monads** are just one (maybe the most important) **model of computation**

other models of computation are **arrows** and **applicative functors**.

Scala has binding syntax for **monads**, but not (yet) for **arrows**.  
For **applicative functors** extra syntax makes less sense.


