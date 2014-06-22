# Monads in Java #

A while back Brent Yorgey http://byorgey.wordpress.com posted a menagerie of types. 
A very nice survey of types in Haskell, including, among other things, the monadic types.

A reader posted a question, framed with 'you know, it'd be a lot easier for me 
to understand monads if you provided an implementation in Java.' [1]

Well, that reader really shouldn't have done that.

Really.

Because it got me to thinking, and dangerous things comes out of geophf's thoughts ... 
like monads in Java.

Let's do this.

## Problem Statement ##

Of course, you just don't write monads for Java. You need some motivation. 
What's my motivation?

Well, I kept running up against the Null Object pattern, because 
I kept running up against null in the Java runtime, because there's a whole lot of 
optionality in the data sets I'm dealing with, and instead of using the Null Object, 
null is used to represent absence.

Good choice?

Well, it's expedient, and that's about the best I can say for that.

So I had to keep writing the Null Object pattern for each of these 
optional element types I encountered.

I encounter quite a few of them ... 2,100 of them and counting.

So I generalized the Null Object pattern using generic types, 
and that worked fine ...

But I kept looking at the pattern. 
The generalization of the Null Object pattern is Nothing, and 
a present instance is Just that something.

I had implemented non-monadic Maybe.

And the problem with that is that none of the power of Maybe, 
as a Monad, is available to a generalized Null Object pattern.

Put another way: I wish to do something given that something is present.

Put another-another way: **bind**.

So, after a long hard look ('do I really want to implement monads? 
Can't I just settle for 'good enough' ... that really isn't good enough?), 
I buckled down to do the work of implementing not just Maybe (heh: 'Just Maybe'), 
but Monad itself.

Did it pay off? Yes, but wait and see. 
First the implementation. But even before that, let's review what a monad is.

## What the Hell is a Monad? ##

Everybody seems to ask this question when first encountering monads, 
and everybody seems to come up with their own explanations, 
so we have tutorials that explain Monads as things as far-ranging 
from spacesuits to burritos.

My own explanation is to punt ... to category theory.

Monad is the triple (T, η, μ) where
* T is the Monad Functor
* η is the unit function that lifts an ordinary object (a unit) into the Monad functor; and,
* μ is the join function that takes a composed monad M (M a) and returns the simplified type of the composition, or M a.

And that is (simply) what the hell a Monad is. 
What a monad can do ... well, there are articles galore about that, and, later, 
I will provide examples of what Monad gives us ... in Java, no less.

## The Implementation ##

We need to work our way up to Monad. 
There's a whole framework, a way of thinking, a context, that needs to be in place 
for Monad to be effective or even to exist. 
Monad is 'happy' in the functional world, so we need to implement functions.

Or more correctly, 'functionals' ... in Haskell, they are called **Functors**.

A **functor** is a container, and the function of functors is to take an ordinary function, 
which we write: `f: a → b` (pronounced **the function f from (type) a to (type) b** and 
means that f takes an argument of type a and returns a result of type b), 
and give a function `g: Functor a → Functor b`.

Functor basically is a container for the function `fmap: Functor f ⇒ (a → b) → (f a → f b)`

Well, is Functor the fundamental type? Well, yes and no.

It is, given that we have functions defined. In Java, we don't have that. 
We have methods, yes, but we don't have free-standing functions. 
Let's amend that issue:

```java
public interface Applicable<T1, T2> {
    public T2 apply(T1 arg) throws Failure;
};
```

And what's this Failure thingie? It's simply a functional exception, so:

```java
  public class Failure extends Exception {
      public Failure(String string) { 
          super(string); 
      }
  

      /// and the serialVersionUID for serialization; your IDE can define that
  }
```

I call a Function 'Applicable' because in functional languages that's what you do: 
apply a function on (type) `a` to get (type) `b`. 
And, using Java Generics terminology, type a is T1 and type b is T2. 
So we're using pure Java to write pure Java code and none will be the wiser that 
we're actually doing Category Theory via a Haskell implementation ... in Java, right?

So, now we have functions (Applicable), we can provide the declaration of the Functor type:

```java
public interface Functor<F, T> {
    public <T1> Applicable<? extends Functor<F, T>,
                           ? extends Functor<F, T1>>
        fmap(Applicable<T, T1> f) throws Failure;
  
    public T arg() throws Failure;
}
```

The above definition of fmap is the same declaration (in Java) as its functional declaration: 
it takes a function `f: a → b` and gives a function `g: Functor a → Functor b`

Okay, we're half-way there. A Monad is a Functor, and we have Functor declared.

The unit function, η, comes for free in Java: it's called new.

So that leaves the join function μ.

And defining join for a specific monad is simple. 
Join for the list monad is append. 
Join for the Maybe monad and the ID monad is arg. 
What about join for just Monad?

Well, there is no definition of such, but we can declare its type:

```java
public interface Joinable<F, T> extends Functor<F, T> {

    public Functor<F, ?> join() throws Failure;
}
```

Too easy! you complain.

Well, isn't it supposed to be? Join is simply a function that takes a functor and returns functor.

The trick is that we have to ensure that T is of type Functor<F, ?>, and 
we leave that an implementation detail for each specific monad implementing join.

That was easy!™

## Monad ##

So now that we have all the above, **Functor**, **Join**, and **Unit**, 
we have Monad:

... BUT WAIT!

And here's the interlude of where the power of Monad comes in: **bind**.

What is bind? **Bind** is a function that says: do something in the monadic domain.

Really, that sounds boring, right? But it's a very powerful thing. 
Because once we are in the monadic domain, we can guarantee things that we cannot 
in Java category (if there is such).

(There is. I just declared it.) [2]

So, in the **monadic domain**, a function thus bound can be guaranteed to, 
e.g., be working with an instance and not a null.

Guaranteed.

AND! ...

Well, what is bind?

**Bind** is a function that takes an ordinary value and returns a result in the monadic domain:

```java
Monad m ⇒ bind: a → m b
```

So the 'AND!' of this is that bind can be chained ... meaning we can go from guarantee 
to guarantee, threading a safe computation from start to finish.

Monad gives us bind.

And the beauty of Monad? bind comes for free, for 
once **join** is defined, **bind** can be defined in terms of join:

```java
Monad m ⇒ bind (f: a → m b) (m a) = join ((fmap f) (m a))
```

Do you see what is happening here? We are using fmap to lift the function f 
one step higher into the monadic domain, so fmap f gives the function` Monad m ⇒ g: m a → m (m b)` 
so we are not actually passing in a value of type a, we are passing in the `Monad m a`, and 
then we get back a type of m (m b) which then join does it magic to simplify to the monadic type m b.

So to the outside world, it looks like we are passing in an a to get an m b but 
this definition allows monadic chaining, for a becomes `m a` and then `m (m b)` 
is simplified to `m b` that — and here's the kicker — is passed to the next monadically bound function.

You can chain this as far as you'd like:

```java
m a >>= f >>= g >>= ... >>= h → m z
```

And since the computation occurs in the monadic domain, you are guaranteed the promise of that domain.

There is the power of monadic programming.

Given that explanation, and remembering that bind can be defined in terms of join, let's define Monad:

```java
public abstract class Monad<M, A> implements Joinable<M, A> {
      public <T> Monad<M, T> bind(Applicable<A, Monad<M, T>> f) throws Failure {
          Applicable<Monad<M, A>, Monad<M, Monad<M, T>>> a
              = (Applicable<Monad<M, A> Monad<M, Monad<M, T>>>) fmap(f);
          Monad<M, Monad<M, T>> mmonad = a.apply(this);
          return (Monad<M, T>) mmonad.join();
      }
  

      public Monad<M, A> fail(String ex) throws Failure {
          throw new Failure(ex);
      }
}
```

The bind implementation is exactly as previously discussed: 
bind is the application of the fmap of f with the joined result returned.
So, we simply define join for subclasses, usually a trivial definition, and 
we have the power of bind automagically!

Now fail is an interesting beast in subclasses, because in the monadic domain, 
failure is not always a bad thing, or even an exceptional one, and can be quite, and very, useful. 
Some examples, fail represents 'zero' in the monadic domain, so 
'failure' for list is the empty list, and 
'failure' for Maybe is Nothing. 
So, when failure occurs, the computation can continue gracefully and not always 
automatically abort from the computation, which happens with the try/catch paradigm.

There you have it: Monads in Java!

### The rest of the story: ... Maybe ##

So, we could've ended this article right now, and you have everything you need to do monadic programming in Java. But so what? A programming paradigm isn't useful if there are no practical applications. So let's give you one: Maybe.

Maybe is a binary type, it is either Nothing or Just x where x is whatever value that is what we're really looking for (as opposed to the usual case in Java: null).

So, let's define the Maybe type in Java.

First is the protocol that extends the Monad:

```java
  public abstract class Maybe<A> extends Monad<Maybe, A> {
```
  
Do you see how the Maybe type declares itself as a Monad in its own definition? I just love that.

Anyway.

As mentioned before, fail for Maybe is Nothing:

```java
      public Maybe<A> fail(String ex) throws Failure {
          return (Maybe<A>) NOTHING;
      }
```

(NOTHING is a constant in Maybe to be defined later. Before we define it ('it' being NOTHING), 
recall that a Monad is a Functor so we have to define fmap. Let's do that now)

```java
      protected abstract <T> Maybe<T> 
              mbBind(Applicable<A, Monad<Maybe, T>> arg) throws Failure;
  

      public <T> Applicable<Maybe<A>, Maybe<T>>
                  fmap(final Applicable<A, T> f) throws Failure {
          return new Applicable<Maybe<A>, Maybe<T>>() {
              public Maybe<T> apply(Maybe<A> arg) throws Failure {
                  Applicable<A, Monad<Maybe, T>> liFted =
                          new Applicable<A, Monad<Maybe, T>>() {
                      public Maybe<T> apply(A arg) throws Failure {
                          return Maybe.pure(f.apply(arg));
                      }
                  };
                  return (Maybe<T>)arg.mbBind(liFted);
              }
          };
      }
```

No surprises here. 
fmap lifts the function `f: a → b` to `Maybe m ⇒ g: m a → m b` 
where in this case the Functor is a Monad, and this case, that Monad is Maybe. [3]

There is the small issue(s) of what the static method pure and the method mbBind are, 
but these depend on the definitions of the subclasses NOTHING and Just, so let's define them now.

```java
      public static final Maybe<?> NOTHING = new Maybe() {
          public String toString() {
              return "Nothing";
          }
          public Object arg() throws Failure {
              throw new Failure("Cannot extract a value from Nothing.");
          }
          public Functor join() throws Failure {
              return this;
          }
          protected Maybe mbBind(Applicable f) {
              return this;
          }
      };
```

Trivial definition, as the Null Object pattern is trivial. 
But do note that the bind operation does not perform the f computation, 
but skips it, returning Nothing, and forwarding the computation. 
This is expected behavior, for:

```java
Nothing >>= f → Nothing
```

The definition of the internal class Just is nearly as simple:

```java
      public final static class Just<J> extends Maybe<J> {
          public Just(J obj) {
              _unit = obj;
          }
  

          public Maybe<?> join() throws Failure {
              try {
                  return (Maybe<?>)_unit;
              } catch(ClassCastException ex) {
                  throw new Failure("Joining on a flat structure!");
              }
          }
          public String toString() {
              return "Just " + _unit;
          }
          public Object arg() throws Failure {
              return _unit;
          }
          protected <T> Maybe<T> 
                  mbBind(Applicable<J, Monad<Maybe, T>> f) throws Failure {
              return (Maybe<T>)f.apply(_unit);
          }
          private final J _unit;
      }
```

As you can see, the slight variation to NOTHING for Just is that join returns the _unit value if it's the Maybe type (and throws a Failure if it isn't), and mbBind applies the monadic function f to the Just value.

And with the definition of Just we get the static method pure:

```java
      public static <T> Maybe<T> pure(T x) {
          return new Just<T>(x);
      }
```

And then we close out the Maybe implementation with:

}

Simple.

## Practical Example ##

Maybe is useful for any semideterministic programming, that is: 
where something may be true, or it may not be. 
But the question I keep getting, from Java coders, is this: 
"I have these chains of getter methods in my web interface to my data objects to set a result, but I often have nulls in some values gotten along the chain. How do I set the value from what I've gotten, or do nothing if there's a null along the way."

"Fine, no problems ..." I begin, but then they interrupt me.

"No, I'm not done yet! I have like tons of these chained statements, and I don't want to abort on the first one that throws a NullPointerException, I just want to pass through the statement with the null gracefully and continue onto the next assignment. Can your weirdo stuff do that?"

Weirdo stuff? Excusez moi?

I don't feel it's apropos that functionally pure languages have no (mutable) assignment, as that throws provability out the window and is therefore the root of all evil.

No matter how sorely tempted I am.

So, instead I say: "Yeah, that's a bigger problem [soto voce: so you should switch to Haskell!], but the same solution applies."[4]

So, let's go over the problem and a set of solutions.

The problem is something like:

```java
  try {
      foo.setD(getA().getB().getC().getD());
      foo.setE(getA().getB().getC().getE());
  

      bar.setJ(getF().getG().getH().getJ());
      bar.setM(getF().getG().getK().getM());
  


      // and ... oh, more than 2000 more assignments ... 'hypothetically'
  


  } catch(Exception ex) {
      ex.printStackTrace();
  }
```

And the problem is this: anywhere in any chain of gets, if something goes wrong, the entire computation is stopped from that point, even if there are, oh, let's say, 1000 more valid assignments.

A big, and real, problem. Or 'challenge,' if you prefer.

Now this whole problem would go away if the Null Object pattern was used everywhere. But how to enforce that? In the Java category, you cannot. Even if you make the generic Null Object the base object, the null is still Java's ⊥ — it's 'bottom' — as low as you go in a hierarchy, null is still an allowable argument everywhere.

And if getA(), or getB(), or getC() return null you've just failed out of your entire computation with an access attempt to a null pointer.

## Solutions ##

### Solution 1: test until you puke ###

The first solution is to test for null at every turn. 
And you know what that looks like, but here you go. 
Because why? Because if you think it or code it, you've got to look at it, or I've got to look at it, so here it is:

```java
  A a = getA();
  if(a != null) {
      B b = a.getB();
      if(b != null) {
          C c = b.getC();
          if(c != null) {
              foo.setD(c.getD());
          }
      }
  }
```

What's the problem with this code?

Oh, no problems, just repeat that deep nesting for every single assignment? So you have [oh, let's say 'hypothetically'] 2000 of these deeply nested conditional blocks?

And, besides the fact that it entirely clutters the simple assignment:

```java
foo.setD(getA().getB().getC().getD());
```

in decision logic and algorithms that are totally unnecessary and entirely too much clutter.

All I'm doing is assigning a D to a foo! So why do I have to juggle all this conditional code in my head to reach that eventual assignment.

Solution 1 is bunk.

### Solution 2: narrow the catch ###

So, solution 1 is untenable. But we need to assign where we can and skip where we can't. So how about this?

```java
  try {
      foo.setD(getA().getB().getC().getD());
  } catch(Exception ex) {
      // Intentionally empty; silently don't assign if we cannot;
  }
  

  try {
      foo.setE(getA().getB().getC().getE());
  } catch(Exception ex) {
      // ditto
  }
  


  try {
      bar.setJ(getF().getG().getH().getJ());
  } catch(Exception ex) {
      // ditto
  }
  


  try {
      bar.setM(getF().getG().getK().getM());
  } catch(Exception ex) {
      // ditto
  }
```

And I say to this solution, meh! Granted, it's much better than solution 1, but, again, you made a computational flow described declaratively in the problem to be these set of staccato statements, having the reader of your code switch into and out of context for every statement.

Wouldn't it be nice to have all the statements together in one block, because, computationally, that's what is happening: a set of data is collected from one place and moved to another, and that is what we wish to describe by keep these assignment grouped in one block.

### Solution 3: Maybe Monad ###

And that's what we can do with monads. It's going to look a bit different that how you're used to the usual Java fair, as we have to lift the statements in the Java category into expressions in the monadic one.

So here goes.

First of all, we need to define generic functions (not methods![5]) for getting values from objects and setting values into object in the monadic domain:

```java
  public abstract class SetterM<Receiver, Datum>
          implements Applicable<Datum, Monad<Maybe, Receiver>> {
      protected SetterM(Receiver r) {
          this.receiver = r;
      }
  

      // a monadic set function
      public final Monad<Maybe, Receiver> apply(Datum d) throws Failure {
          set(receiver, d);
          return Maybe.pure(receiver);
      }
  


      // subclasses implement with the particular set method being called
      protected abstract void set(Receiver r, Datum d);
  


      private final Receiver receiver;
  }
```

This declares a generic setter, so to define setD on the foo instance, we would do the following:

```java
      SetterM<Foo, D> setterDinFoo = new SetterM<Foo, D>(foo) {
          protected void set(Foo f, D d) {
              f.setD(d);
          }
      };
```

The getter functional is similarly defined, with the returned value lifted into (or 'wrapped in,' if you prefer) the Maybe type:

```java
  public abstract class GetterM<Container, Returned>
          implements Applicable<Container, Monad<Maybe, Returned>> {
  

      public Monad<Maybe, Returned> apply(Container c) throws Failure {
          Maybe<Returned> ans = (Maybe<Returned>)Maybe.NOTHING;
          Returned result = get(c); 
  


          // c inside the monad is guaranteed to be an instance
          // but result has NO guarantees!
  


          if(result != null) {
              ans = Maybe.pure(result);  
              // and now ans is inside the monad it must have a value.
              // ... even if that value is NOTHING
          }
          return ans;
      }
  


      protected abstract Returned get(Container c);
  }
```

With the above declaration, a getter monadic function (again, not method) is simply defined:

```java
      GetterM<A, B> getterBfromA = new GetterM<A, B>() {
          protected B get(A a) {
              return a.getB();
          }
      };
```

In Haskell, the bind operator has a data-flow look to it: (>>=), so writing one of the 'assignments' is as simple as flowing the data to the setter:

```haskell
getA >>= getB >>= getC >>= getD >>= setD foo
```

But we have no operator definitions, so we must soldier on using the Java syntax:

```java
      try {
          getterA.apply(this).bind(getterBfromA).bind(getterCfromB).bind(getterDfromC).bind(setterDinFoo);
      } catch {
          // a superfluous catch block
      }
```

That's one of the assignment expressions.[6]

Let's walk through what happens with a couple of examples.

Let's say that A has a value of a, but B is null. What happens?

1. getterA forwards Just a to getterBfromA
2. getterBfromA gets a null, converts that into a NOTHING and forwards that to getterCfromB
3. getterCfromB forwards the NOTHING to getterDfromC
4. getterDfromC forwards the NOTHING to setterDinFoo
5. setterDinFoo simply returns NOTHING.

And we're done (that is: we're done doing, literally: NOTHING).
Let's counter with an example where every value has an instance:

1. getterA forwards Just a to getterBfromA
2. getterBfromA gets a b, converts that into Just b and forwards that to getterCfromB
3. getterCfromB gets a c, converts that into Just c and forwards that to getterDfromC
4. getterDfromC gets a d, converts that into Just d and forwards that to setterDinFoo
5. setterDinFoo gets the wrapped d and sets that value in the Foo instance.

Voilà!

The beauty of this methodology is that we can put them all into one block, 
and every expression will be evaluated and in a type-safe manner, too, 
as NOTHING will protect us from a null pointer access:

```java
try {
  getterA.apply(this)
         .bind(getterBfromA)
         .bind(getterCfromB)
         .bind(getterDfromC)
         .bind(setterDinFoo);
  System.out.println("Hey, I've executed the first assignment");

  getterA.apply(this)
         .bind(getterBfromA)
         .bind(getterCfromB)
         .bind(getterEfromC)
         .bind(setterEinFoo);
  System.out.println("...and the second...");

  getterF.apply(this)
         .bind(getterGfromF)
         .bind(getterHfromG)
         .bind(getterJfromH)
         .bind(setterJinBar);
  System.out.println("...and third...");

  getterF.apply(this)
         .bind(getterGfromF)
         .bind(getterKfromG)
         .bind(getterMfromK)
         .bind(setterMinBar);
  System.out.println("...and fourth...");
  
  // another more than 2000 of these expressions
  System.out.println("...and we're done with every assignment we could assign...");

} catch {
  // a superfluous catch block
}
```

And, because the monadic Maybe works it does, every assignment that can occur will, and 
ones that cannot will be 'skipped' (by flowing NOTHING through the rest of the computation, and 
the proof will be in the pudding ... on the standard output will be all the statements printed.

## Summary ##

We presented an implementation of monads in Java, with a concrete example of the Monad type: 
the Maybe type. We then showed that by putting a computational set into the monadic domain, 
the work can be performed in a type-safe manner, even with the possible presence of nulls. 
This is just one example of practical application of monadic programming in Java: 
the framework presented here confers the benefits of research of monadic programming 
in general to projects done in Java. Use the framework to discover for yourself the 
leverage monadic programming gives.

End notes
 	 
[1]	Exact verbage: Phil says: January 14, 2009 at 7:01 pm I think that 
this could all be easily cleared up if one of you FP guys would just show us 
how to write one of these monad thingies in Javaâ To which a semi-mocking response was: 
'you know, objects would be a lot easier for me to understand if you provided 
an implementation in Haskell.' Exact verbage: Cory says: April 23, 2009 at 6:26 
am I may be a bit late to the game here, but Phil, that can be rephrased: 
I think that this could all be easily cleared up if one of you OO guys would 
just show us how to write one of these object thingies in Haskellâ Of course you can, 
but it's a different type of abstraction for a different way of thinking about programming
â I write 'semi-mocking,' because: 1. OOP with message passing has been implemented 
in a Haskell-like language: Common Lisp. 
The Art of the Metaobject Protocol, a wonderful book, covers the 'traditional' 
object-oriented programming methodology, and its implementation, quite thoroughly and simply.

[2]	I leave it as an exercise to the reader to scope the Java category (Consult the Ω-calculus papers, Cardelli, et al;).

[3]	Monad<Maybe, A> and Maybe<A> are type-equivalent (but try telling the poor Java 1.6 compiler that).

[4]	"The same solution applies!" Geddit? Haskell is an applicative language, so the same solution applies! GEDDIT?

[5]	The distinction I raise here between methods and functions is the following: a method is enclosed in and owned by the defining class. A function, on the other hand, is a free-standing object and not necessarily contained in nor attached to an object.

[6]	Again, there is a distinction between an expression and a statement. An expression returns a value; a statement does not. A statement is purely imperative ... 'do something.' On the other hand, an expression can be purely functional ... 'do'ing nothing, as it were, e.g.: 3 + 4. A pure expression has provable properties that can be attached to it, whereas it is extremely difficult to make assertions about statements in the general sense. The upshot is that having pure (non-side-effecting) expressions in code allows us to reason about that code from a firm theoretical foundational basis. Such code is easier to test and to maintain. Side-effecting statement code make reasoning, testing and maintaining such code rather difficult.
