# Java8 Monads #

Monads are central to Functional programming. 
Although this has not been much advertised, Java 8 brings monads.

In a recent post, (http://java.dzone.com/articles/java-8-optional-whats-point) 
Hugues Johnson asks about Java 8 Optional: Whats the point?

The example he gives is close to the following:

```java
String getString() {
    //method returning a String or null, such as get on a Map<String, String>
}

Option<String> optionalString = Optional.ofNullable(getString());
optionalString.ifPresent(System.out::toString);
```

He then asks what the point of using Optional since we can do the same without it:

```java
String getString() {
    //method returning a String or null, such as get on a Map<String, String>
}

String string = getString();
if (string != null) System.out.println(string);
```

This is missing the point. The important point is that 
Optional is a **monad** and is supposed to be used as such.

## What is a monad? ##

Without entering the details of Category theory, a monad is a very simple, 
yet extremely powerful thing. 

A monad a set of three things:
* a **parameterized type** M<T>
* a **unit** function T -> M<T>
* a **bind** operation: M<T> bind T -> M<U> = M<U>

These may seems over-complicated, but it is really simple considering an example, 
such as the Optional monad:
* **Parameterized type**: Optional<T>
* **unit**: Optional.ofNullable()
* **bind**: Optional.flatMap()

# How to use the Optional monad #

The Optional monad is meant to allow composing functions that may or may not return a value, 
when the absence of value is not an error. 
The most common example is searching for a key in a map. Suppose we have the following:

```java
Person person = personMap.get("Name");
process(person.getAddress().getCity());
```

Here, we are searching a person by its name in a map, 
then we get the person address, 
then the city in the address, and 
we pass the result to the process method. 
Lot of things may happen:
* The "Name" key may not be present in the map resulting in person being null.
* If the key is present, person.getAdress() may return null.
* If the address is not null, the city may be null.
* Eventually all may run fine, with nothing being null.

(We will not consider the case were the null value is bind to the “Name” key in the map.)

In the first three cases, we get an NPE. To handle this, we must do the following:

```java
Person person = personMap.get("Name");
if (person != null) {
  Adress address = person.getAddress();
  if (address != null) {
    City city = address.getCity();
    if (city != null) {
      process(city)
    }
  }
}
```

How can we use Optional in this code to clear up this mess? Well, we can't. To use Optional, we have first to modify the Map, Person, and Address classes so that all their methods return an Optional. Then, we can change our code to:

```java
Optional<Person> person = personMap.get("Name");
if (person.isPresent()) {
  Optional<Adress> address = person.getAddress();
  if (address.isPresent()) {
    Optiona<City> city = address.getCity();
    if (city.isPresent()) {
      process(city)
    }
  }
}
```

But this is NOT how Optional is intended to be used. 
Optional is a monad and is intended to be use as such:

```java
personMap.find("Name")
         .flatMap(Person::getAddress)
         .flatMap(Address::getCity)
         .ifPresent(ThisClass::process);
```

This implies that we use a modified version of Map:

```java
public static class Map<T, U> extends HashMap<T, U> {
    public Optional<U> find(T key) {
        return Optional.ofNullable(super.get(key));
    }
}
```

and that methods `getAddress` and `getCity` return `Optional<Address>` and `Optiona<City>`.

**Note 1**: we have used method references in this example to clarify the code, and 
one could argue that method references are not functions. 
In fact, this is only syntactic sugar over:

```java
personMap.find("Name")
         .flatMap(x -> x.getAddress())
         .flatMap(x -> x.getCity())
         .ifPresent(() -> process(x));
```

Here, `x -> x.getAddress()` is a function of type `T -> M<U>`.

**Note 2**: remark that `ifPresent` is exactly the same as `forEach` for collections and streams. 
It should have been called forEach although there is at most one element. 
This should have make clearer the relation between the Optional monad and the List monad. 
Unfortunately, List is not a monad in Java 8, but it can be transformed easily to a Stream.

Of course, some of these methods may always return a value. For example, given the following class:

```java
public static class Address {

  public final City city;

  public Address(City city) {
    this.city = city;
  }

  public City getCity() {
    return city;
  }
}
```

we may change our code to:

```java
personMap.find("Name")
         .flatMap(Person::getAddress)
         .map(Address::getCity)
         .ifPresent(ThisClass::process);
```

This is how Optional is meant to be used.

## The (missing) Try monad ##

So Java 8 has monads. Stream is also a monad. 
We should use monads for what they were intended.

Then all is good? Nope. The main problem is that there are many things missing 
in order to promote the use of monads. 
In this example, we can see that the `get` method in **Map** should have been updated, 
as is the case for all methods that would possibly return null. 
As it is no possible to break backward compatibility, they should have been deprecated 
and new methods should have been added like the find method in the above map. 
(I will not speak about methods taking null as an argument, since these methods should never have existed.)

Another major concern is that the Optional monad is only good for cases where 
the absence of value is not an error. In our case, if all methods should 
**return a value** or **throw an exception**, we would be in trouble 
because we have no monad for this.

Here is what we could do in imperative programming:

```java
Person person = personMap.get("Name");
if (person != null) {
    Adress address = person.getAddress();
    if (address != null) {
        City city = address.getCity();
        if (city != null) {
            process(city)
        } else {
            throw new IllegalStateException("Address as no city");
        }
    } else {
        throw new IllegalStateException("Person has no address");
    }
} else {
    throw new IllegalStateException("Name not found in map");
}
```

Note that throwing exceptions like this is a modern form of goto, 
with the difference that we do not know where we are going.

To apply the same programming style we have used with Optional, we need another 
monad that is often called **Try**. But Java 8 does not have it. 
We may write it ourselves, which is not very difficult:

```java
public abstract class Try<V> {

  private Try() {
  }

  public abstract Boolean isSuccess();

  public abstract Boolean isFailure();

  public abstract void throwException();

  public static <V> Try<V> failure(String message) {
    return new Failure<>(message);
  }

  public static <V> Try<V> failure(String message, Exception e) {
    return new Failure<>(message, e);
  }

  public static <V> Try<V> failure(Exception e) {
    return new Failure<>(e);
  }

  public static <V> Try<V> success(V value) {
    return new Success<>(value);
  }

  private static class Failure<V> extends Try<V> {

    private RuntimeException exception;

    public Failure(String message) {
      super();
      this.exception = new IllegalStateException(message);
    }

    public Failure(String message, Exception e) {
      super();
      this.exception = new IllegalStateException(message, e);
    }

    public Failure(Exception e) {
      super();
      this.exception = new IllegalStateException(e);
    }

    @Override
    public Boolean isSuccess() {
      return false;
    }

    @Override
    public Boolean isFailure() {
      return true;
    }

    @Override
    public void throwException() {
      throw this.exception;
    }

  }

  private static class Success<V> extends Try<V> {

    private V value;

    public Success(V value) {
      super();
      this.value = value;
    }

    @Override
    public Boolean isSuccess() {
      return true;
    }

    @Override
    public Boolean isFailure() {
      return false;
    }

    @Override
    public void throwException() {
      //log.error("Method throwException() called on a Success instance");
    }
  }

  // various method such as map an flatMap
}
```

Such a class may be used to replace Optional. 
The main difference is that if at some stage in the program some component 
returns a **Failure** instead of a **Success**, it will still compose with the next method call. 
For example, given that `Map.find()`, `Person.getAddress()`, and `Address.getCity()` 
all return a `Try<Something>` our previous example may be rewritten as:

```java
personMap.find("Name")
         .flatMap(Person::getAddress)
         .flatMap(Address::getCity)
         .ifPresent(This.class::process);
```

Yes, this is exactly the same code as what we wrote with Optional. 
The differences are in the classes used, for example:

```java
public static class Map<T, U> extends HashMap<T, U> {
  public Try<U> find(T key) {
    U value = super.get(key);
    if (value == null) {
      return Try.failure("Key " + key + " not found in map");
    }
    else {
      return Try.success(value);
    }
  }
}
```


As Java does not allow overriding methods which differ only by the return type, 
we would have to choose different names for method returning Optional and Try. 
But this is not even necessary, since Try may be used to replace Optional. 
The only difference is that if we want to process the exception, 
we can have special methods in the Try class such as:

```java
public void ifPresent(Consumer c) {
  if (isSuccess()) {
    c.accept(successValue());
  }
}

public void ifPresentOrThrow(Consumer<V> c) {
  if (isSuccess()) {
    c.accept(successValue());
  } else {
    throw ((Failure<V>) this).exception;
  }
}

public Try<RuntimeException> ifPresentOrFail(Consumer<V> c) {
  if (isSuccess()) {
    c.accept(successValue());
    return failure("Failed to fail!");
  } else {
    return success(failureValue());
  }
}
```

This gives the user (the business programmer, as opposed to the API designer) the choice of what to do. 
He can use `ifPresent` to obtain with **Try** the same result as with **Optional** and ignore any exception, or 
he can use `ifPresentOrThrow` to throw the exception if there is one, or 
he can use `ifPresentOrFail` if he wants to handle the exception in any other way, as in the following example:

```java
personMap.find("Name")
         .flatMap(Person::getAddress)
         .flatMap(Address::getCity)
         .ifPresentOrFail(TryTest::process)
         .ifPresent(e -> Logger.getGlobal().info(e.getMessage()));
```

Note that the exception we get at the end of the chain may have occurred in any of the methods. 
It is simply transmitted from one function to the next. 
So the API designer does not have to care about what to do with the exception. 
The result is the same as if the exception had been thrown and caught by the user, 
without the need of a try/catch block, and 
without the risk of forgetting to catch it in case of an unchecked exception.

##Other useful monads ##

Many other monads may be very useful. 
* We may use monads **to handle randomness** 
  (function that usually return a changing value, such as date and random generators). 
* We may use monads **to handle functions returning multiple values**. 
  List is not a monad in Java, but we can easily create it, or 
  we can get a Stream from a list, and Stream is a monad. 
* We may also use monads **to deal with future values**, and 
* even **for primitives wrappers**.

In Java 8, the `Collection.stream()` method may be used to transform a collection into a monad.

## So what is wrong? ##

Since it is so simple to write our own monads, is there something wrong? In fact there are at least three.

* The first one is that although we can create the missing monads, 
  we will not be able to use them in a public API, because each API would have 
a different, incompatible implementation. 
We need a standard implementation we can share.

* The second is that Java API should have been retrofitted to use these monads. 
  It should have been done at least for Optional.

* Another recurrent problem is due to primitives. 
  Optional won't work with primitives, so there are special versions for int, double and long 
(OptionalInt, OptionalDouble and OptionalLong). 
We really need value types! 
(but this might be coming: see http://www.dzone.com/links/r/brian_goetz_value_types_big_on_the_agenda_for_fut.html)


## Comments ##
 
David Gates replied on Wed, 2014/05/21 - 9:00am  
The lack of a way to code to monads in general stood out as a problem 
since I first learned about Java 8's implementation of Stream.  
I should be able to write code once and have it work unmodified with 
collections, Optional, and any user-defined monads.

The other issue is lack of syntax to streamline monadic computations; 
something like **comprehensions** syntax in Haskell or Scala, or **Linq query** syntax in C#.  
Without this, combining multiple sources has more conceptual clunkiness than necessary:

```java
// Imperitive style
List<Integer> resultList = new ArrayList<Integer>();
for (Integer i : list1) {
	for (Integer j : list2) {
		for (Integer k : list3) {
			resultList.add(i + j + k);
		}
	}
}
```

```java
// Java 8
list1.stream()
	   .flatMap(i -> list2.stream().flatMap(
				   j -> list3.stream().map(
				   k -> i + j + k)))
	   .collect(Collectors.toList());
```

```scala
// Scala comprehension
// Translates to calls to flatMap and map
for { i <- list1
		j <- list2
		k <- list3 } yield i + j + k
```


Lukas Eder replied on Thu, 2014/05/22 - 8:46am in response to: David Gates  
Agh. I just find all of these idioms quite hard to read and write 
(except for the imperative one, in fact, which is about OK).

I wish we had a bit more SQL in our languages:

```java
SELECT i.value + j.value + k.value
       FROM i, j, k
```
 
David Gates replied on Thu, 2014/05/22 - 10:51am in response to: Lukas Eder   
 The Java 8 version IS hard to understand, which is why I wish they had specialized syntax.  
If the Scala comprehension is hard for you to read, it's because it's unfamiliar.  
C# based its syntax on SQL rather than for loops:

```scala
from i in list1
from j in list2
from k in list3
select i + j + k;
```

This compiles to the Linq equivalents of `map` and `flatMap`: `Select` and `SelectMany`.

 
Pierre-yves Saumont replied on Thu, 2014/05/22 - 11:34am in response to: David Gates  
I agree, but I would say that if the Java 8 version is hard to understand, it is also because it is unfamiliar. 

I personally find the Scala for comprehension syntax more difficult to read, and 
this is probably BECAUSE I do not use it,  and not WHY I do not use it; -)
 
Lukas Eder replied on Thu, 2014/05/22 - 12:44pm in response to: David Gates
If the Scala comprehension is hard for you to read, it's because it's unfamiliar. 

You're right, but Scala syntax is a device whose mystery is only exceeded by its power. 
I'm not sure if a single developer can ever master all idioms :-) 
Although, I frequently say the same about SQL, considering things like 
ordered aggregate functions (WITHIN GROUP (ORDER BY ...)) or Oracle's MODEL clause

reply
 
David Gates replied on Thu, 2014/05/22 - 12:46pm in response to: Pierre-yves Saumont  
I'm guessing the yield throws people with the Scala syntax; it just means we're building 
a new list instead of calling foreach.  The above example translates to:

```scala
list1.flatMap(i => list2.flatMap(
              j => list3.map(
              k => i + j + k)))
```

I went through a period where I'd write code that way then realize I could clean it up with the comprehension syntax.  
Now I jump straight to comprehensions if I'm working with multiple sources, but it's the only time I use them.  
If you're more comfortable in Haskell (which I'm guessing you are, as you use Haskell's terminology and monad definition), 
here's the equivalent comprehension:

```haskell
do i <- list1
   j <- list2
   k <- list3
   return (i + j + k)
```
 
David Gates replied on Thu, 2014/05/22 - 1:12pm in response to: Lukas Eder  
There's a paper titled A Co-Relational Model of Data for Large Shared Data Banks 
that demonstrates how relational algebra is mathematically equivalent to monadic computations.  
If you love the power of SQL, then you already have a picture of why functional programmers gush about monads.  
We're used to having that power all the time in normal code.

 
Lukas Eder replied on Thu, 2014/05/22 - 2:11pm in response to: David Gates  
Yes, I've read that paper by Erik Meijer and Gavin Bierman, although I somewhat doubt that 
there was "proof" in the paper rather than "evidence", if somewhat esoteric at times - 
specifically when trying to coerce the whole theory upon the hype-term "NoSQL". 
I do think, though, that the interesting aspect of the paper is the study of 
inversion of the "direction" of arrows when comparing 
relations (child->parent) with objects (parent->child).
 However, this is nothing new.

In any case, SQL is a bit more than that. 
I have yet to see an elegant functional way to express aforementioned ordered aggregate functions, 
or window functions, grouping sets, and many other OLAP features. Also, I believe that it is 
far easier to express very complex relationships between transformed data sets 
in SQL using common table expressions or derived tables, than with monads.

Other expressions, of course, are easier to express in a functional way, e.g. recursion.

 
David Gates replied on Thu, 2014/05/22 - 4:37pm in response to: Lukas Eder  
The mathematical portion of the paper shows how the monadic primitive operations map to relational algebra concepts.  The primitive operations let you work with a single value (map), work with single values from multiple monads (flatMap), and operate conditionally on a value (filter).  Individual monad types (such as lists) can of course have additional functionality, just as SQL isn't limited to basic set theory operations.

Here's some sample SQL code I found with ordered aggregation:

```sql
SELECT LISTAGG(product_name, ', ') WITHIN GROUP (ORDER BY product_cost) "Product_Listing"
FROM products;
```

I'm not a SQL guru, so I might be misinterpreting the code or your question, but here's my Scala version:

```scala
products.sortBy(p => p.cost)
        .map(p => p.name)
        .mkString(", ")
```

C#'s query syntax has dedicated keywords for grouping and ordering, which might be more comfortable coming from SQL.

 
Lukas Eder replied on Fri, 2014/05/23 - 12:51am in response to: David Gates  
I'm not a SQL guru, so I might be misinterpreting the code or your question, but here's my Scala version

That's probably a correct translation, but have you considered what it means to project tuples in a similar way with monads.? Your mkString method seems to consume the entire Stream (or collection in Scala?). You cannot have more than one aggregation in this case

What if every column of a tuple has different PARTITION BY, ORDER BY, and frame criteria (if it's a window function)? In fact, how would you implement the frame in a window with Scala functions? (I'm really curious, it would be awesome if it worked).

And what if you express complex tuple-sources (i.e. tables), e.g. with LEFT OUTER JOIN or more complex joins. I know that LINQ provides quite a bit of abstraction, but this is because of LINQ providers implementing a lot of functionality. But Scala's Slick seems to be struggling a lot with this kind of mapping of actual SQL features to Scala collection querying.

reply
 
David Gates replied on Fri, 2014/05/23 - 6:18am
Scala's stock tuples are flawed, as they don't provide a way to abstract over arity.  For example, here's Wikipedia's LEFT OUTER JOIN sample:

```sql
SELECT *
FROM employee LEFT OUTER JOIN department
ON employee.DepartmentID = department.DepartmentID;
```

where employees have a name and may have a department ID.  
ID lookups like this are typically a bad code smell, and the employees would instead 
have a direct reference to the department:

```java
type Employee = (String, Optional[Department])
```

If we have a data store that relies on ID lookups (such as, say, a relational database), 
we run into the limitations of stock tuples.  
We need a function from `(String, Option[Int])` to 
`(String, Option[Int], Option[String], Option[Int])`, which is easy enough.  
The problem is, we'd have to code a separate function for every combination of left table width to right table width.  
It also doesn't convey that all three optional values are either there or not, though neither does the SQL version.

There is a library called Shapeless that adds some functional goodies missing in stock Scala; 
this particular problem wants heterogeneous lists and union types.  
Heterogeneous lists maintain the type information of each element as tuples do:

```
List(1, "Two", 3.0)   // List[Any]
HList(1, "Two", 3.0)  // Int :: String :: Double :: HNil
```

but because they're recursively defined rather than fixed-length, 
it's easy to write operations on two arbitrary-length HLists.  
Using union types, we can define our Employee records to be either 
String :: HNil or String :: Int :: HNil.  
After the join, we have records of either String :: HNil or String :: Int :: String :: Int :: HNil.

If I were writing a library like Slick specifically to map onto relational databases, 
I'd also use Shapeless's ability to encode strings in the type information.  
My Employee records wouldn't just have a String; they would know that the String is an 
"Employee.Name" without any runtime overhead or having to define an Employee class.

That's a quick, top-of-my-head answer; I'd have to do more research to see if 
it applies to the other use cases you mentioned, but a lot of the limitations of 
Scala's tuples don't apply to heterogeneous lists.


Riccardo Muzzì replied on Tue, 2014/06/10 - 7:19am  
Nice article, just a few points:

At the beginning you say: 
"He then asks what the point of using Optional since we can do the same without it". 
Well, if by "same" you mean the the 2 examples do the same thing I agree, however 
take a look to the cyclomatic complexity of the first (which is 1) and the second(which is 2). 
That may be subjective but I think it increases a bit code maintainability.

In the end you state:
"The second is that Java API should have been retrofitted to use these monads. It should have been done at least for Optional."
My question is: How can you do that without breaking backward compatibility? 
Anyway, I don't see this as a real problem because you can always do something like:

```java
Optional.of( oldMethodOfJavaNotRetrofittedForUsingMonads())
        .flatMap(...)
        .orElse(...)

In the end: Can you please expand a bit the point of using Optional for primitive type? 
The point of using optional is mostly to avoid null check in that object or in a nested object (with flatMap), but 
in primitive types you have no null values nor nested values/objects... what am I missing?

 
Pierre-yves Saumont replied on Tue, 2014/06/10 - 8:36am in response to: Riccardo Muzzì  
 Hi Riccardo,

You are perfectly right that cyclomatic complexity in the exposed code is lower. 
In fact, Using Optional is abstracting the (cyclomatic) complexity. 
We get the same result when using a terminal operation on a stream. 
This abstracts iteration, which is why it is sometimes called "internal iteration", although 
this is a particularity of terminal operations, not Streams by themselves.

To retrofit existing classes in order to return Optional, one could:
- add methods for classes not implementing interfaces
- add default methods to interfaces and implementations to classes for the classes implementing interfaces. 
That's the reason why default methods were added in Java 8.

One can create Optional from a returned value as you show, but it is better, in my opinion, 
to abstract this (complexity again!) in the APIs.

The reason for using Optional with primitives is exactly the same as with objects, although 
most often, we would need Try instead of Optional. 
Optional may be used each time no result is not an error. 
Try (if it existed) should be used each time no result is an error.

You may for example have an object with a field of type int which value is optional. 
How can you handle this? 
I saw many tricks for this, such as negative values (when the correct value should be positive).

This is exactly what String.indexOf does. 
It returns -1 if there is no value. So positive values mean one thing and 
a negative value means something else, leading to errors such as using the return value 
as an index to access array elements. This could not happen with Optional.
