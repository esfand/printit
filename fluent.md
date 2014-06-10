# How to refer to the current type with a type variable? #

Suppose I'm trying to write a function to return an instance of the current type. 
Is there a way to make `T` refer to the exact subtype (so `T` should refer to `B` in class B)?

```java
class A {
    <T extends A> foo();
}
```

```java
class B extends A {
    @Override
    T foo();
}
```

## Answer - Summary ##

You should be able to do this using the **recursive generic definition** 
style that Java uses for `enum`s:

```java
class A<T extends A<T>> {
    T foo();
}
```

```java
class B extends A<B> {
    @Override
    B foo();
}
```

## Solution by Paul Bellora ##


The following pattern would be necessary 
(this is a recipe for a **hierarchical fluent builder** API).


First, a base abstract class (or interface) that lays out the contract for 
returning the runtime type of an instance extending the class:

```java
/**
 * @param <SELF> The runtime type of the implementor.
 */
abstract class SelfTyped<SELF extends SelfTyped<SELF>> {

   /**
    * @return This instance.
    */
   abstract SELF self();
}
```

All intermediate extending classes must be abstract and maintain the **recursive type parameter** `SELF`:

```java
public abstract class MyBaseClass<SELF extends MyBaseClass<SELF>>
                                       extends SelfTyped<SELF> {

    MyBaseClass() { }

    public SELF baseMethod() {

        //logic

        return self();
    }
}
```

Further derived classes can follow in the same manner. 
But, none of these classes can be used directly as types of variables 
without resorting to rawtypes or wildcards (which defeats the purpose of the pattern). 
For example (if `MyClass` wasn't abstract):

```java
//wrong: raw type warning
MyBaseClass mbc = new MyBaseClass().baseMethod();

//wrong: type argument is not within the bounds of SELF
MyBaseClass<MyBaseClass> mbc2 = new MyBaseClass<MyBaseClass>().baseMethod();

//wrong: no way to correctly declare the type, as its parameter is recursive!
MyBaseClass<MyBaseClass<MyBaseClass>> mbc3 =
        new MyBaseClass<MyBaseClass<MyBaseClass>>().baseMethod();
```

This is the reason I refer to these classes as **intermediate**, 
and it's why they should all be marked abstract. In order to close the loop 
and make use of the pattern, **leaf** classes are necessary, which resolve 
the **inherited type parameter** `SELF` with its own type and implement `self()`. 
They should also be marked final to avoid breaking the contract:

```java
public final class MyLeafClass extends MyBaseClass<MyLeafClass> {

    @Override
    MyLeafClass self() {
        return this;
    }

    public MyLeafClass leafMethod() {

        //logic

        return self(); //could also just return this
    }
}
```

Such classes make the pattern usable:

```java
MyLeafClass mlc = new MyLeafClass().baseMethod().leafMethod();
```

```java
AnotherLeafClass alc = new AnotherLeafClass().baseMethod().anotherLeafMethod();
```

The value here being that method calls can be chained up and down the class hierarchy 
while keeping the same specific return type.

## DISCLAIMER ##

The above is an implementation of the curiously recurring template pattern in Java. 
This pattern is not inherently safe and should be reserved for the inner workings 
of one's internal API only. The reason is that there is no guarantee the 
type parameter `SELF` in the above examples will actually be resolved to the correct 
**runtime type**. For example:

```java
public final class EvilLeafClass extends MyBaseClass<AnotherLeafClass> {

    @Override
    AnotherLeafClass self() {
        return getSomeOtherInstanceFromWhoKnowsWhere();
    }
}
```

This example exposes two holes in the pattern:

1. `EvilLeafClass` can *lie* and substitute any other type extending `MyBaseClass` for `SELF`.
2. Independent of that, there's no guarantee `self()` will actually return `this`, 
   which may or may not be an issue, depending on the use of state in the base logic.

For these reasons, this pattern has great potential to be misused or abused. 
To prevent that, allow none of the classes involved to be publicly extended -- 
notice my use of the package-private constructor in `MyBaseClass`, 
which replaces the implicit public constructor:

```java
MyBaseClass() { }
```

If possible, keep `self()` *package-private* too, so it doesn't add noise and 
confusion to the public API.  Unfortunately this is only possible if `SelfTyped` 
is an abstract class, since interface methods are implicitly public.

As zhong.j.yu points out in the comments, the bound on `SELF` might simply be removed, 
since it ultimately fails to ensure the **self type**:

```java
abstract class SelfTyped<SELF> {

   abstract SELF self();
}
```

Yu advises to rely only on the contract, and avoid any confusion or false sense of 
security that comes from the unintuitive recursive bound. 
Personally, I prefer to leave the bound since `SELF extends SelfTyped<SELF>` 
represents the closest possible expression of the self type in Java. 
But Yu's opinion definitely lines up with the precedent set by Comparable.

## CONCLUSION ##

This is a worthy pattern that allows for fluent and expressive calls to your builder API. 
I've used it a handful of times in serious work, most notably to write a custom 
query builder framework, which allowed call sites like this:

```java
List<Foo> foos = QueryBuilder.make(context, Foo.class)
    .where()
        .equals(DBPaths.from_Foo().to_FooParent().endAt_FooParentId(), parentId)
        .or()
            .lessThanOrEqual(DBPaths.from_Foo().endAt_StartDate(), now)
            .isNull(DBPaths.from_Foo().endAt_PublishedDate())
            .or()
                .greaterThan(DBPaths.from_Foo().endAt_EndDate(), now)
            .endOr()
            .or()
                .isNull(DBPaths.from_Foo().endAt_EndDate())
            .endOr()
        .endOr()
        .or()
            .lessThanOrEqual(DBPaths.from_Foo().endAt_EndDate(), now)
            .isNull(DBPaths.from_Foo().endAt_ExpiredDate())
        .endOr()
    .endWhere()
    .havingEvery()
        .equals(DBPaths.from_Foo().to_FooChild().endAt_FooChildId(), childId)
    .endHaving()
    .orderBy(DBPaths.from_Foo().endAt_ExpiredDate(), true)
    .limit(50)
    .offset(5)
    .getResults();
```

The key point being that `QueryBuilder` wasn't just a flat implementation, 
but the *leaf* extending from a complex hierarchy of builder classes. 
The same pattern was used for the helpers like `Where`, `Having`, `Or`, etc. 
all of which needed to share significant code.

However, you shouldn't lose sight of the fact that all this only amounts 
to syntactic sugar in the end. Some experienced programmers take a hard 
stance against the **CRT pattern**, or at least are skeptical of the its 
benefits weighed against the added complexity. Their concerns are legitimate.

Bottom-line, take a hard look at whether it's really necessary before 
implementing it - and if you do, don't make it publicly extendable.

## Comment ##

This can be a good solution if you know that people extending MyClass 
will always follow the rules you've established. There is no way to 
enforce this using compilation rules, though. – StriplingWarrior

	 	
@StriplingWarrior Agreed - it's fit more for the internal workings of one's API, 
not to be publicly extendable. I actually used this construct recently for a 
fluent builder pattern. For example one could call `new MyFinalClass().foo().bar()`
where `foo()` is declared in a superclass and `bar()` is defined in `MyFinalClass`. 
Only the final concrete classes are exposed outside the package so my teammembers 
can't mess with `self()` etc. I hadn't realized until now that 
intermediate classes maintaining the type parameter couldn't be used 
directly though. –  Paul Bellora 
  	 	
That's a comprehensive answer. Now if we just remove the bound of SELF, it looks 
simpler and won't confuse new guys. The bound does not seem to be very useful 
anyway. –  zhong.j.yu
  	 	
@zhong.j.yu The bound is necessary for a hierarchical fluent builder pattern, 
since builder methods (e.g. baseMethod in my walkthrough or lessThanOrEqual 
in my example) must return SELF. –  Paul Bellora
	 	
how do you mean? without the bound for SELF, everything still works 
fine. –  zhong.j.yu
  	 	
@zhong.j.yu You're right actually - thanks for pointing that out. 
Now that you helped jog my memory, the bound only became necessary for my use of nested builders. 
In my bottom snippet this would be for example handing off to an Or object 
with `or()` and then returning to the outer builder with `endOr()`. 
In a nutshell, the bound was needed so that the different builders could talk to each other. 
But that's an extended aspect that I didn't explore in the answer. 
Outside of that I think whether to have the bound comes down preference. 
I'll update the answer to that affect when I get time. –  Paul Bellora
  	 	
This works, with caveats, including not having concrete intermediate classes, 
requiring generics, preventing the top level class from extending an existing (concrete) class, 
exposure to the EvilLeafClass possibility, etc. 
I actually like my solution better, though it requires a call before each 
method invocation. –  GGB667
  	 	
@GGB667 Yep, there are many trade-offs for the fluent prettiness, 
as I hope the answer articulates - it's a tough sell. –  Paul Bellora









