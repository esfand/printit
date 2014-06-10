# Using Generics To Build Fluent API's In Java #

Normally when creating a fluent interface we want the ability to chain method calls 
together in a variety of different orders. And often in object oriented programming 
we put shared functionality in a base class and extend from that. Well unfortunately, 
in strongly typed languages such as Java these two desires are often at odds with 
each other. 

The problem is that a base class method returning an instance of itself doesn’t know 
anything about its derived classes. Consider the following example:

```java
public abstract class StandardBakedGood {

    public StandardBakedGood prepare() {
        System.out.println("preparing the kitchen");
        return this;
    }

    public StandardBakedGood bake() {
        System.out.println("baking");
        return this;
    }
}
```

```java
public class Cookie extends StandardBakedGood {

    public Cookie addFlour() {
        System.out.println("adding flour");
        return this;
    }
}
```

The base class is only guaranteeing that a StandardBakedGood type will be returned. This means that when chaining calls together, you are effectively blocked from calling a derived class’s methods after a base class method. In the example, we can’t call any Cookie class methods after the first call to prepare().

```java
Cookie = new Cookie()
    .prepare();
    //.addFlour()    // inaccessible!
    //.bake();
```

How can we get around this while working within the type system? 

Well one option is to have the child classes override the base class methods to return instances of their own type. In our example, we would have this:

```java
@Override
public Cookie prepare() {
    return (Cookie) super.prepare();
}

@Override
public Cookie bake() {
    return (Cookie) super.bake();
}
```

This has some obvious downsides. First, you are relinquishing control of the method invocation since the child can do whatever it wants, and you cannot make these methods final. Second, if you want to add functionality to the base class then all of the derived classes need to be updated. All of this can be avoided if we can somehow make the base class aware of the child class’s type, and fortunately generics make that possible.

## Generics to the rescue. ##

Consider this typical example of a derived class passing information about itself to its superclass through a constructor:

```java
public class ChildClass extends BaseClass {
    
    public ChildClass() {
        super(ChildClass.class);
    }
}
```

The basic idea is that the superclass requests information about the child, and the child provides it. 
However this does little for typing, and in fact creates a bad sort of dependency 
where we need to update the base class every time we make a new derived class. 
Not good! Fortunately, we can accomplish the same idea using generics. 
By having the base require information about the child we are able to capture the child class’s type 
and use that as the return type of our chained API methods.

```java
public abstract class BakedGood<CHILD extends BakedGood<CHILD>> {

    @SuppressWarnings("unchecked")
    public CHILD prepare() {
        System.out.println("preparing the kitchen");
        return (CHILD) this;
    }

    @SuppressWarnings("unchecked")
    public CHILD eat() {
        String string = new StringBuilder()
            .append("Mmm, this ").append(this.getClass().getSimpleName())
            .append(" is so tasty!")
            .toString();

        System.out.println(string);
        return (CHILD) this;
    }

    @SuppressWarnings("unchecked")
    public abstract CHILD bake();
}
```

```java
public class Cake extends BakedGood<Cake> {

    public Cake addSugar() {
        System.out.println("adding sugar");
        return this;
    }

    public Cake addFlour() {
        System.out.println("adding flour");
        return this;
    }

    public Cake mix() {
        System.out.println("mixing");
        return this;
    }

    public Cake bake() {
        System.out.println("baking at 325 degrees");
        return this;
    }
}
```

```java
public class Pizza extends BakedGood<Pizza> {

    public Pizza addCheese() {
        System.out.println("adding cheese");
        return this;
    }

    public Pizza addSauce() {
        System.out.println("adding sauce");
        return this;
    }

    public Pizza bake() {
        System.out.println("baking at 450 degrees");
        return this;
    }
}
```

The type parameter is saying **the Child class must extend Base<Child>**, 
forcing the Child class to provide its own type to the type system. 
Now that we can return the derived class in our chained method calls we 
are free to alternately call methods from the base class and the derived class. 
The IDE will happily autocomplete for us because it knows that the objects being returned are 
Pizza and Cake, not just BakedGood. All of the normal polymorphic abilities are retained 
(you can see that we’ve implemented the abstract bake() method required by BakedGood).

```java
Cake  cake;
Pizza pizza;

cake = new Cake()
    .prepare()
    .addFlour()
    .addSugar()
    .mix()
    .bake();

System.out.println();

pizza = new Pizza()
    .prepare()
    .addSauce()
    .addCheese()
    .bake();

System.out.println();

cake.eat();
pizza.eat();
```

The output:

```text
preparing the kitchen
adding flour
adding sugar
mixing
baking at 325 degrees

preparing the kitchen
adding sauce
adding cheese
baking at 450 degrees

Mmm, this Cake is so tasty!
Mmm, this Pizza is so tasty!
```

## Turning up the heat ###

Hopefully you see the power of this application of generics now. 
We’ve managed to create a fluent API while keeping shared code in the base class 
of our object hierarchy. However, it gets even better. We are free to implement 
generic classes using this technique as well. In the following example, 
Pizza and Cake have been implemented as generic classes which take a type parameter. 
The BakedGood class has also been extended to be generic.

```java
public abstract class GenericBakedGood<CHILD extends GenericBakedGood<CHILD, T>, T> 
                                             extends BakedGood<CHILD> {

    @SuppressWarnings("unchecked")
    public CHILD addSecretIngredient(T ingredient) {
        System.out.println("adding some " + ingredient);
        return (CHILD) this;
    }
}
```

```java
public class GenericCake<T> extends GenericBakedGood<GenericCake<T>, T> {

    public GenericCake<T> addSugar() {
        System.out.println("adding sugar");
        return this;
    }

    public GenericCake<T> addFlour() {
        System.out.println("adding flour");
        return this;
    }

    public GenericCake<T> mix() {
        System.out.println("mixing");
        return this;
    }

    public GenericCake<T> bake() {
        System.out.println("baking at 325 degrees");
        return this;
    }
}
```

```java
public class GenericPizza<T> extends GenericBakedGood<GenericPizza<T>, T> {

    public GenericPizza<T> addCheese() {
        System.out.println("adding cheese");
        return this;
    }

    public GenericPizza<T> addSauce() {
        System.out.println("adding sauce");
        return this;
    }

    public GenericPizza<T> addTopping(T topping) {
        System.out.println("putting some " + topping + " on top");
        return this;
    }

    public GenericPizza<T> bake() {
        System.out.println("baking at 450 degrees");
        return this;
    }
}
```

The `addSecretIngredient(...)` method expects a generically typed parameter. 
Combined with the other methods, we have successfully expanded our API 
to make use of generic types:

```java
GenericCake<String>  cake;
GenericPizza<String> pizza;

cake = new GenericCake<String>()
    .prepare()
    .addFlour()
    .addSugar()
    .addSecretIngredient("Yogurt")
    .mix()
    .bake();

System.out.println();

pizza = new GenericPizza<String>()
    .prepare()
    .addSauce()
    .addCheese()
    .addTopping("Bacon")
    .addSecretIngredient("Arugula")
    .addTopping("Clams")
    .bake();

System.out.println();

cake.eat();
pizza.eat();
```

And the corresponding output:

```text
preparing the kitchen
adding flour
adding sugar
adding some Yogurt
mixing
baking at 325 degrees
```

```text
preparing the kitchen
adding sauce
adding cheese
putting some Bacon on top
adding some Arugula
putting some Clams on top
baking at 450 degrees

Mmm, this GenericCake is so tasty!
Mmm, this GenericPizza is so tasty!
```

Looking at the declaration of GenericBakedGood, you can see how the class 
can be expanded to include more type parameters—in the event that more 
parameters are required, you can simply add their type parameters after the ‘T’:

```java
public abstract class MapBase<CHILD extends MapBase<CHILD, K,V>, K,V> { }

public class MyMap<K,V> extends MapBase<MyMap<K,V>, K,V> { }
```

It’s worth pointing out that just because the GenericBakedGood class expects a 
type parameter doesn’t mean that the derived class itself has to be generic 
(this is true of all generics). 
In the following example, the type parameter for the secret ingredient is provided 
in the Muffin class’s declaration, though the Muffin class itself is not generic:

```java
public class Muffin extends GenericBakedGood<Muffin, Muffin.Mixin> {

    public enum Mixin {
        Blueberries, Raspberries, Walnuts
    }


    public Muffin addFlour() {
        System.out.println("adding flour");
        return this;
    }

    public Muffin addEggs() {
        System.out.println("adding eggs");
        return this;
    }

    public Muffin bake() {
        System.out.println("baking at 275 degrees");
        return this;
    }
}
```

When we go to add the secret ingredient, an object of type Muffin.Mixin is required:

```java
Muffin muffin = new Muffin()
    .prepare()
    .addFlour()
    .addEggs()
    .addSecretIngredient(Muffin.Mixin.Blueberries)
    .bake();

System.out.println();

muffin.eat();
```

```text
preparing the kitchen
adding flour
adding eggs
adding some Blueberries
baking at 275 degrees

Mmm, this Muffin is so tasty!
```

## Conclusion ##

Developing a highly useable API is one of the more interesting challenges of 
software architecting. In a strongly typed language such as Java, we must be 
especially clever about how we declare our methods so that we can ensure the 
greatest possible compatibility. 

Some other solutions to this particular problem 
* would require modifying the base class whenever a new derived class is created, or 
* forcing the derived classes to override the base class methods themselves. 

Both of these fly in the face of good design, where 
shared functionality is defined only once in the object hierarchy (the DRY principal), and 
the dependencies only flow in one direction (that is, a parent should not depend on its children).

Java generics have a bad reputation, being added on late in the game, and indeed 
there are many times that I want to pull my hair out when something doesn’t work as 
expected at runtime. However the principals described here should be applicable to any 
strongly typed language with generics as a feature. I think you’ll agree that the gains in readability of your API are worth the little hassles and the extra bulk of the class signatures. So try it out; go forth and create something of your own. Show those programmers working with dynamic languages that you can be beautiful too!

## Notes ##

The best article I was able to find on the topic was this one. 
Some practical applications of fluent API’s can be found here and here. 
The example code detailed in this post is written in Java, is free to use, and can be downloaded here. 

**UPDATE:** It has a name! First pointed out in C++, it is often called the 
**Curiously Recurring Template Pattern**, or **CRTP**.

## Flapi, the Fluent API Builder for Java |##

In a previous post, Using Generics To Build Fluent API’s In Java, I detailed a way 
to create type-safe fluent API’s using generics. Handy, but unfortunately the process 
can be somewhat tedious. Any project wishing to utilize a builder must carefully 
hand-code the various classes and interfaces. 
Modifications made later are oftentimes painful to implement.

Looking at the simplistic examples from the previous article, it is easy to see patterns in the way the classes are constructed. The type parameters act as a sort of stack, storing the memory of where a method came from, while simultaneously providing insight into where it can go. A sort of…state machine, where classes can transition between each other based on the methods invoked. That’s all a program really is in the end anyway, right?

I took that concept and started writing a code generation tool which would build fluent API’s in Java, wrapping the complex maneuvering around the many types required. Flapi is the result of that work. Using a fluent builder itself, it is easy to create a set of classes which form a builder of your design. Lots of great features like nested blocks of methods, chaining methods together, and invocation tracking make writing and using a builder easy and fun. See the Getting Started page on the wiki to start creating your own fluent API’s.

