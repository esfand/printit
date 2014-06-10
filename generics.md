
## Lambdas and Generics ##

Generally, lambdas work well with generic types. 
There are just a couple of issues to keep in mind.

One of the unhappy consequences of type erasure is that you cannot construct a generic array at runtime. 
For example, the toArray() method of Collection<T> and Stream<T> cannot call `T[] result = new T[n]`. 
Therefore, these methods return Object[] arrays. 

In the past, the solution was to provide a second method that accepts an array. 
That array was either filled or used to create a new one via reflection. 
For example, Collection<T> has a method `toArray(T[] a)`. 

With lambdas, you have a new option, namely to pass the constructor. 
That is what you do with streams:

```
String[] result = words.toArray(String[]::new);
```

When you implement such a method, the constructor expression is an `IntFunction<T[]>`, 
since the size of the array is passed to the constructor. In your code, you call 
`T[] result = constr.apply(n)`.

In this regard, lambdas help you overcome a limitation of generic types. 
Unfortunately, in another common situtation lambdas suffer from a different limitation. 
To understand the problem, recall the concept of type variance.

Suppose `Employee` is a subtype of `Person`. 
Is a `List<Employee>` a special case of a `List<Person>`? 
It seems that it should be. But actually, it would be unsound. 
Consider this code:

```
List<Employee> staff = ...;
List<Person> tenants = staff; // Not legal, but suppose it was
tenants.add(new Person("John Q. Public")); // Adds Person to staff!
```

Note that staff and tenants are references to the same list. 
To make this type error impossible, we must disallow the conversion from `List<Employee>` to `List<Person>`. 
We say that the type parameter `T` of `List<T>` is **invariant**.

If List was immutable, as it is in a functional programming language, 
then the problem would disappear, and one could have a covariant list. 
That is what is done in languages such as Scala. 
However, when generics were invented, Java had very few immutable generic classes, and 
the language designers instead embraced a different concept: 
**use-site variance**, or **wildcards**.

A method 
* can decide to accept a `List<? extends Person>` if it only reads from the list. 
  Then you can pass either a `List<Person>` or a `List<Employee>`. Or 
* it can accept a `List<? super Employee>` if it only writes to the list. 
  It is okay to write employees into a List<Person>, so you can pass such a list. 

In general, 
* **reading is covariant** (subtypes are okay), and 
* **writing is contravariant** (supertypes are okay). 

**Use-site variance** is just right for mutable data structures. 
It gives each service the choice which **variance**, if any, is appropriate.

However, for function types, use-site variance is a hassle. 
A function type is always 
* **contravariant in its arguments** and 
* **covariant in its return value**.
 
For example, if you have a `Function<Person, Employee>`, you can safely pass it on 
to someone who needs a `Function<Employee, Person>`. 
They will only call it with employees, whereas your function can handle any person. 
They will expect the function to return a person, and you give them something even better.

In Java, when you declare a generic functional interface, you can’t specify that 
function arguments are always contravariant and return types always covariant. 
Instead, you have to repeat it for each use. For example, look at the javadoc for `Stream<T>`:

```java
void          forEach(Consumer <? super T> action)
Stream<T>     filter (Predicate<? super T> predicate)
<R> Stream<R> map    (Function <? super T, ? extends R> mapper)
```

The general rule is that you 
**use super for argument types**, and 
**use extends for return types**. 
That way, you can pass a Consumer<Object> to forEach on a Stream<String>. 
If it is willing to consume any object, surely it can consume strings.

But the wildcards are not always there. Look at

```
T reduce(T identity, BinaryOperator<T> accumulator)
```

Since T is the argument and return type of BinaryOperator, the type does not vary. 
In effect, the contravariance and covariance cancel each other out.

As the implementor of a method that accepts lambda expressions with generic types, you simply 
* add **? super**   to any argument type that is not also a return type, and 
* add **? extends** to any return type that is not also an argument type.

For example, consider the doInOrderAsync method of the preceding section. Instead of

```
public static <T> void doInOrderAsync(Supplier<T>         first, 
                                      Consumer<T>         second, 
                                      Consumer<Throwable> handler)
```

it should be

```
public static <T> void doInOrderAsync(Supplier<? extends T>       first, 
                                      Consumer<? super T>         second, 
                                      Consumer<? super Throwable> handler)
```


<hr/>

## An introduction to generics in java ##

You may have heard all the fuss about the generics in java. Collections use them. 
But what are they exactly?

Consider you want a data structure which stores some strings in an ordered manner. 
What comes to the mind? arrays and ArrayList . Let’s declare them

```java
String[] stringsArray = new String[50];
List stringsList = new ArrayList();
```

If we add anything other than String into stringsArray, then compiler will stop you, 
complaining that type cannot be added to an array of String.

```java
stringsArray[0] = new Integer(10); // No this does not compile
```

This is good for us, we don’t want anything other than String in stringsArray . 
But we would like to use ArrayList instead of arrays as it provides advantages over it. 
But what happens we add something to stringsList instead of only String

```java
list.add("prasad");
list.add("kharkar");
list.add(new Integer(0)); // sadly, works perfectly fine.
```

This is because the statement List list = new ArrayList(); means that you can 
add ANYTHING into the collection. Any object can be added to it. 
Here comes the benefit of generics into picture.

If you want only a list of String then you can define it as.

```java
List<String> stringsList = new ArrayList<String>();
```
 
After declaring like this, you will get type safety at compile time. 
So if you add anything other than String into stringsList now, 
it will not compile.

```java
list.add("prasad");
list.add("kharkar");
list.add(new Integer(0)); // does not compile
```

This will stop you from adding anything other than String saying 
The method add(String) in the type List<String> is not applicable 
for the arguments (Integer) .

This is the advantage of using generics for compile time safety.

<hr/>




## Java Generic Arrays ##

### Introduction ###

The focus here is not particularly "learning weird Java features" but we use them nonetheless because:

They are an excellent match for describing the sort of data structures and algorithms we are using.
Picking up programming knowledge "along the way" is a vital skill that you will never stop using.
In particular, we use Java's generic types a lot because they capture exactly the idea that many 
of our data structures and algorithms do not rely on the type of data being manipulated.

### Workaround 1: The Basic Generic Array ###

You can declare a field or variable to have a generic array type:

```
class Foo<E> {

  E[] arr;

  void m() {
     E[] localArr;
     . . .
  }
}
```

However, you cannot create a new generic array with:

```
new E[SOME_SIZE];
```

This is because when the code runs, the type E is "not known" and so 
the code has no idea how to identify the type of the elements for 
the array -- and that's important for checking that the array is used properly. 
The same idea works in many other languages, but it doesn't work in Java.

One workaround is to create an Object[] and then 
cast it (generating a warning) to E[]:

```
arr = (E[])new Object[SOME_SIZE]; // WORK-AROUND #1
```

Now when the code runs, the array that arr refers to will allow any Object in it, so there won't be any run-time problems. Be wary of casting from Object[] to E[] though -- this should only be used when creating a new array with new. Otherwise, other code might put items in your array that you are not expecting.

### Workaround 2: The Array of a Parameterized Type ###

It's not just E[] that forbids array creation: we can't create an array where the elements have any parameterized type:

```
class C1<E> {
  E myEfield;
  ...
}

class C2 {
   // Doesn't work: C1<String>[] x = new C1<String>[100];
   C1<String>[] x = new C1[100]; // works fine
   ...
}

class C3<E> {
   C1<E>[] x = new C1<E>[100]; // not allowed
}
```

One natural reaction is to try our first workaround:

```
C1<E>[] x = (C1<E>[]) new Object[100]; // won't work either
```

This "natural reaction" will compile, but it will generate a ClassCastException when executed. 
To understand why, let's digress briefly to "plain old non-generic arrays" and 
Java's problematic treatment of array subtyping. Assume B is a subtype of A, such as with:

```
class A { ... }
class B extends A { ... }
```

Then this works:

```
B[] b_array = new B[100];
A[] a_array = b_array;
b_array = (B[])a_array;
```

Since an array of B objects contains all A objects (thanks to subtyping every B "is also" an A), 
this may seem fine. And it is allowed, provided two things:

You never assign into the array an A that is not a B. If you do this, at run-time you will get 
an ArrayStoreException. This is crucial so that any use of b_array can never see an array element 
that is not a B. But the compiler doesn't catch this.

When you do a cast like `(B[])a_array`, the code checks that a_array actually refers to an array 
that hold elements of type B. If it doesn't, then you get a ClassCastException.
The point is: Arrays "know their element type" and this is used to check that arrays are used 
properly and throw exceptions for misuse.

So now back to generics: While arrays "know their element type", they only know the "raw" type -- 
the type that forgets all about generics. But that's still enough for our "natural reaction" not to work: 
You cannot cast an array that holds elements of type Object to an array that holds elements of 
"raw type" C1. Such an array could have elements that are not of type C1, so this would not be safe 
and you get a ClassCastException.

So here's the workaround you actually need:

```
C1<E>[] x = (C1<E>[]) new C1[100]; // WORK-AROUND #2
```

When we created the array we said it would always hold all C1 elements. C1 here is a "raw type" -- 
we haven't said everything about the type of elements (like C1<String> or C1<Integer>), 
but we said that much. That's enough for the cast to succeed. This is just as safe/dangerous as 
our first work-around: you should only do this for newly created arrays or you can get strange 
errors where arrays don't hold elements of the type you think they do.

### Work-around 3: Arrays of inner classes inside parameterized types ###

The last situation we'll walk through is actually very similar to work-around #2 once you 
understand what inner classes "really are". Consider:

```java
class C<E> {

    class D { // inner class
      ...
    }

    D[] array = new D[100]; // doesn't work
}
```

Now this really seems annoying: D doesn't "look generic" so why can't we create an array of type `D[]`. 
Well, inner classes are convenient and great for indicating "this class D is only used inside class C", 
but the Java language does not treat them particularly specially. The class D is actually the class `C<E>.D` 
here: the class D defined inside the generic class `C<E>`. And so it is like you wrote this harder to read version:

```java
C<E>.D[] array = new C<E>.D[100]; // same thing: doesn't work
```

Now it turns out Java doesn't let you write types like C<E>.D -- it will give a parse error. 
But that's what you "are really saying" when you write D inside class C<E>. 
So we'll use C<E>.D to explain what's going on, even though you can't write it.

The reason this doesn't work is `C<E>.D` mentions a type parameter and we know from work-around #2 
that you can't create such arrays. Fortunately, the same work-around applies: 
use the raw type in the new expression and then cast it to the generic type:

```
C<E>.D[] array = (C<E>.D[]) new C.D[100]; // wordy work-around, not legal Java
```

Now you can't write this, but that's okay: just remember that D means `C<E>.D` and 
you get something that is easier to read anyway:

```
D[] array = (D[]) new C.D[100]; // WORK-AROUND #3
```

So while this 'cleaner' version looks different than work-around #2, the wordier version 
shows it's really the same concept extended to inner classes. 
The whole point is that C.D is a "raw type" but D, which means C<E>.D, is not.

#### Digression: Don't Shadow Type Parameters in Inner Classes ####

Though not related to arrays, another mistake related to inner classes is to write:

```java
class C<E> {
   class D<E> { // inner class
       ...
   }
   ...
}
```

This works but is probably not what you meant: You are making D a generic class too -- 
the type parameter inside D is different than the enclosing type parameter for C. 
It's entirely analogous to the error of having a local variable in a method shadow a field name:

```java
class F {
   int x;
   void m() {
      int x = 17;
      ++x; // why is this not referring to the field x?
   }
}
```

But in the case of generics, the mistake will lead to type errors. 
For example, this won't type-check:

```java
class C<E> {
   E x;
   class D<E> { 
       E y = x; // not the same type!
       ...
   }
}
```
But this works fine:

```java
class C<E> {
   E x;
   class D { 
       E y = x; 
       ...
   }
}
```

<hr/>



## The Dangers of Correlating Subtype Polymorphism with Generic Polymorphism ##

Java 5 has introduced generic polymorphism to the Java ecosystem. 
this has been a great addition to the Java language, even if we’re all aware of 
the numerous caveats due to generic type erasure and the consequences thereof.

**Generic polymorphism** (also known as **parametric polymorphism**) is usually maintained 
orthogonally to possibly pre-existing **subtype polymorphism**. A simple example for 
this is the collections API

```java
List<? extends Number> c = new ArrayList<Integer>();
```

In the above example, the subtype `ArrayList` is assigned to a variable of the super type `List`. 
At the same time `ArrayList` is parameterised with the type `Integer`, which can be assigned to 
the compatible parameter supertype `? extends Number`.  This usage of subtype polymorphism in 
the context of generic polymorphism is also called **covariance**, although covariance can also 
be achieved in non-generic contexts, of course.


### Covariance with Generic Polymorphism ###

Covariance is important with generics.  It allows for creating complex type systems. 
Easy examples involve using covariance with generic methods:

```java
<E extends Serializable> void serialize(Collection<E> collection) {}
```

The above example accepts any Collection type, which can be subtyped at the **call-site** 
with types such as List, ArrayList, Set, and many more.  
At the same time, the generic type argument at the call site is only required to be 
a subtype of Serializable.  I.e. it could be a `List<Integer>` or an `ArrayList<String>`, etc.


### Correlating Subtype Polymorphism with Generic Polymorphism ###

People are then often lured into correlating **the two orthogonal types of polymorphism**. 
A simple example of such a correlation would be to specialise an `IntegerList` or `StringSet` as such:

```java
class IntegerList extends ArrayList<Integer> {}
class StringSet   extends HashSet<String>    {}
```

It is easy to see that the number of explicit types will explode, if you start to span 
the cartesian product of the **subtype** and **generic type** hierarchies, wanting to specialise 
more precisely by creating things like `IntegerArrayList`, `IntegerAbstractList`, `IntegerLinkedList` etc.


### Making the Correlation Generic ###

As seen above, such correlations will often remove the genericity from the type hierarchy, 
although they are not required to do so. This can be seen in the following, more general example:

```java
// AnyContainer can contain AnyObject
class AnyContainer<E extends AnyObject> {}
class AnyObject {}
 
// PhysicalContainer contains only PhysicalObjects
class PhysicalContainer<E extends PhysicalObject> extends AnyContainer<E> {}
class PhysicalObject                              extends AnyObject       {}
 
// FruitContainer contains only Fruit,
// which in turn are PhysicalObjects
class FruitContainer<E extends Fruit> extends PhysicalContainer<E> {}
class Fruit                           extends PhysicalObject       {}
```

The above example is a typical one, where the API designer was lured into 
correlating **subtype polymorphism** (`Fruit extends PhysicalObject extends AnyObject`) 
with **generic polymorphism** (`<E>`), while keeping it generic, allowing to add further 
subtypes below FruitContainer. This gets more interesting when `AnyObject` should 
know its own subtype, generically. This can be achieved with a recursive generic parameter. 
Let’s fix the previous example

```java
// AnyContainer can contain AnyObject
class AnyContainer<E extends AnyObject<E>> {}
class AnyObject   <O extends AnyObject<O>> {}
 
// PhysicalContainer contains only PhysicalObjects
class PhysicalContainer<E extends PhysicalObject<E>> extends AnyContainer<E> {}
class PhysicalObject   <O extends PhysicalObject<O>> extends AnyObject<O>    {}
 
// FruitContainer contains only Fruit,
// which in turn are PhysicalObjects
class FruitContainer<E extends Fruit<E>> extends PhysicalContainer<E> {}
class Fruit         <O extends Fruit<O>> extends PhysicalObject<O>    {}
```

The interesting part here are no longer the containers, but the AnyObject type hierarchy, 
which correlates subtype polymorphism with generic polymorphism on its own type! 
This is also done with java.lang.Enum:

```java
public class Enum<E extends Enum<E>> implements Comparable<E> {
    public final int compareTo(E other) { ... }
    public final Class<E> getDeclaringClass() { ... }
}
 
enum MyEnum {}
 
// Which is syntactic sugar for:
final class MyEnum extends Enum<MyEnum> {}
```


### Where Lies the Danger? ###

The subtle difference between `enums` and our custom `AnyObject` hierarchy is the fact that 
`MyEnum` terminates **recursive self-correlation** of the 
**two orthogonal typing techniques** by being final! 
`AnyObject` subtypes, on the other hand, should not be allowed to remove 
the generic type parameter, unless they are made final as well. 
An example:

```java
// "Dangerous"
class Apple extends Fruit<Apple> {}
 
// "Safe"
final class Apple extends Fruit<Apple> {}
```

Why is final so important, or in other words, why must `AnyObject` subtypes be careful 
when terminating recursive self-correlation, such as Apple did, before? 
It’s simple. Let’s assume the following addition:

```java
class AnyObject<O extends AnyObject<O>> implements Comparable<O> {
 
    @Override
    public int compareTo(O other) { ... }
    
    public AnyContainer<O> container() { ... }
}
```

The above contract on `AnyObject.compareTo()` implies that any subtype of 
`AnyObject` can only ever be compared to the the same subtype. 
The following is not possible:

```java
Fruit<?>     fruit     = // ...
Vegetable<?> vegetable = // ...
 
// Compilation error!
fruit.compareTo(vegetable);
```

The only currently comparable type in the hierarchy is Apple:

```java
Apple a1 = new Apple();
Apple a2 = new Apple();
 
a1.compareTo(a2);
```

But what if we wanted to add `GoldenDelicious` and `Gala` apples?

```java
class GoldenDelicious extends Apple {}
class Gala            extends Apple {}
```

We can now compare them!

```java
GoldenDelicious g1 = new GoldenDelicious();
Gala            g2 = new Gala();
 
g1.compareTo(g2);
```

This was not the intention of the author of `AnyObject`!

The same applies to the `container()` method. Subtypes are allowed to 
**covariantly specialise** the `AnyContainer` type, which is fine:

```java
class Fruit<O extends Fruit<O>> extends PhysicalObject<O> {
 
    @Override
    public FruitContainer<O> container() { ... }
}
```

But what happens to the container() method in GoldenDelicious and Gala?

```java
GoldenDelicious       g = new GoldenDelicious();
FruitContainer<Apple> c = g.container();
```

Yes, it will return an Apple container, not a GoldenDelicious container 
as intended by the AnyObject designer.


### Conclusion ###

**Subtype polymorphism** and **generic polymorphism** span orthogonal type axes. 
Making them correlate can be a design smell in your type system. 
Making them correlate on the same type is dangerous, as it is hard to get right. 
Users will try to terminate the **recursive generic type** definition on a subtype 
of your base type. The reason for this termination is the fact that base types 
with recursive self-bounds are hard to use. But the termination often goes wrong, 
as it should only be done on final classes, not regular classes or interfaces.

In other words:
> if you think you need a recursive generic type definition on 
> a common base type, think again very carefully, if you really need it and 
> if your type users can correctly terminate the recursive generic type definition 
> in a final class.


<hr/>



## Generics: Polymorphism with generics ##

Consider this declarations.

```java
List<String> strings = new ArrayList<String>(); // This is valid
List<Object> objects = new ArrayList<String>(); // This is not valid
```

- List is the base type.
- String is the generic type
- ArrayList is the base type.
- String is the generic type.

There is a simple rule. 
> The type of declaration must match the type of object you are creating.

You can make polymorphic references for the base type NOT for the generic type. 
Hence the first statement is valid while second is not.

```java
ArrayList<String> list = new ArrayList<String>(); 
// Valid. Same base type, same generic type

List<String> list1 = new ArrayList<String>(); 
// Valid. Polymorphic base type but same generic type

ArrayList<Object> list2 = new ArrayList<String>(); 
// Invalid. Same base type but different generic type.

List<Object> list3 = new ArrayList<String>(); 
// Invalid. Polymorphic base type different generic type
```

For the sake of this tutorial, consider classes Vehicle, Car and Bike

* `Bike extends Vehicle`
* `Car extends Vehicle`
* `Every class has service() method`

**Vehicle class**
```java
package com.thejavageek.generics;
 
public class Vehicle {
 
    public void service() {
        System.out.println("Generic vehicle servicing");
    }
 
}
```

**Bike Class**
```java
package com.thejavageek.generics;
 
public class Bike extends Vehicle {
    @Override
    public void service(){
        System.out.println("Bike specific servicing");
    }
}
```

**Car class**
```java
package com.thejavageek.generics;
 
public class Car extends Vehicle {
 
    @Override
    public void service() {
        System.out.println("Car specific servicing");
    }
 
}
```

and we have a **Mechanic class** who can do the servicing of all the vehicles.
```java
package com.thejavageek.generics;
 
import java.util.ArrayList;
import java.util.List;
 
public class Mechanic {
    public void serviceVehicles(List<Vehicle> vehicles){
        for(Vehicle vehicle: vehicles){
                vehicle.service();
           }
    }
    public static void main(String[] args) {
 
        List<Vehicle> vehicles = new ArrayList<Vehicle>();
        vehicles.add(new Vehicle());
        vehicles.add(new Vehicle());
 
        List<Bike> bikes = new ArrayList<Bike>();
        bikes.add(new Bike());
        bikes.add(new Bike());
 
        List<Car> cars = new ArrayList<Car>();
        cars.add(new Car());
        cars.add(new Car());
 
        Mechanic mechanic = new Mechanic();
 
        mechanic.serviceVehicles(vehicles); // This works fine.
        mechanic.serviceVehicles(bikes); // Compiler error.
        mechanic.serviceVehicles(cars); //Compiler error.             
    }
}
```

* `mechanic.serviceVehicles(vehicles)` works fine because 
  we are passing `ArrayList<Vehicle>` to the method that takes `List<Vehicle>`.
* `mechanic.serviceVehicles(bikes)` and `mechanic.serviceVehicles(cars)` does not compile because 
  we are passing ArrayList<Bike>  and ArrayList<Car>  to a method that takes List<Vehicle> .

Now why does this happen? Isn’t it obvious that we use polymorphism and pass 
a subtype list to a method that takes supertype. Why doesn’t generics allow this?

Consider this method
```java
public void addVehicle(List<Vehicle> vehicles){
    vehicles.add(new Car());
}
```

and if you were allowed to pass a `List<Bike>` to `addVehicle(List<Vehicle> vehicles)`  
using  `mechanic.addVehicle(bike)`  this method would have added a Car to the list of 
Bike which is plain wrong. Hence generics does not allow passing a subtype of generic type. 
This is because at runtime, the Collections framework does not have this generic information. 
So even if you declare `List<Vehicle> vehicles = new ArrayList<Vehicle>();`, 
This type safety is only at compile time. At runtime, this becomes `List vehicles = new ArrayList();`. 
That’s right, all the type safety is lost. This is called as **Type Erasure**. 

But there is a workaround for this… We need to use the wildcard operator to be able 
to pass a subtype generic type to super type collection. 
We are going to learn the wildcard operator in the next part of this tutorial.

<hr/>

## Generics: The wildcard operator ##

Now that we learned about the shortcomings of generics while using polymorphism with it,
Let's overcome them using wildcard operator.

Now we want to be able to pass a `ArrayList<Bike>` to `addVehicle(List<Vehicle> vehicles)` s
uccessfully but due to type erasure, as discussed in previous article, it is not possible. 
Here comes the use of wildcard operator i.e. ? .

### Wildcard operator and extends keyword ##

We will change the method to

```java
public void addVehicle(List<? extends Vehicle> vehicles){
       // some code here
}
```

After doing this,

```java
mechanic.addVehicle(vehicles); // compiles fine
mechanic.addVehicle(bikes);    // compiles fine
mechanic.addVehicle(cars);     // compiles fine
```

So what does it mean ?

The statement `List<? extends Vehicle>` means that you can pass any type of 
list that subtype of List and which is typed as Vehicle or some subtype of Vehicle. 
But you cannot add anything into the collection at all. 

Now what is that? Why can’t we add anything into the collection? Lets consider this.

```java
public void addVehicle(List<? extends Vehicle> vehicles){
    vehicles.add(new Car());
}
```

The compiler stops you at the very moment. It doesn't allow you to add anything 
while using **? extends**. Why is that?

**? extends** allows us to pass any collection that is subtype of the method parameter 
which is typed as generic type of subtype of the generic type. i.e. we can pass an 
`ArrayList<Vehicle>`, `ArrayList<Bike>` or `ArrayList<Car>` to it.
But consider a scenario in which we are passing `ArrayList<Bike>` to `addVehicle(List<? extends Vehicle>)`. 
In that case, if compiler didn't stop us from adding into the collection, we would have added a Car into 
`ArrayList<Bike>`. This is the scenario so **? extends** doesn’t allow you to add into collection.
But there is also a workaround for this. We can also pass a subtype collection and still be able to add 
into collection. We have to use the super keyword along with wildcard operator

### The wildcard operator and super keyword ###

Now change as follows

```java
public void addVehicle(List<? super Bike> vehicles){
    vehicles.add(new Car());    // can't add a Car to list of Bike
    vehicles.add(new Bike());   // compiles fine because we are adding a Bike
    vehicles.add(new Vehicle());// can't add a Vehicle to a list of Bike
}
```

and we call this method using

```java
mechanic.addVehicle(vehicles); //compiles fine
mechanic.addVehicle(bikes);    //compiles fine
mechanic.addVehicle(cars);     // does not compile
```

Now what is this?

What does List<? super Bike> mean?

You can pass any collection that is of type Bike or any super type of Bike. i.e. 
Vehicle. and you can add elements into it. So Even if you pass a list of bikes or 
list of vehicles, you will still be able to add a Bike into it. But you cannot add 
a Car into the collection because our method parameter declaration allows only 
a list of bikes or list of supertype of Bike, i.e. Vehicle.

Hope this articles helps you understand the wildcard operator and use of 
extends and super keyword with it.
 
<hr/>


