# Covariance, Invariance, and Contravariance #

www.thejavageek.com/2013/08/28/generics-the-wildcard-operator/   
thegreyblog.blogspot.fi/2011/03/java-generics-tutorial-part-ii.html

## Q & A ##

### Question: Explain Covariance, Invariance and Contravariance ###


### Answer: ###

> * Some say it is about relationshop between types and subtypes, 
> * others say it is about type conversion, and 
> * others say it is used to decide whether a method is overwritten or overloaded.

All of the above.

At heart, these terms describe how the **subtype relation** is affected by **type transformations**. 
That is, if 
* **A** and **B** are types, 
* **f** a type transformation, and 
* **≤** the subtype relation (i.e. `A ≤ B` means that A is a subtype of B),

we have

* f is **covariant** if `A ≤ B` implies that `f(A) ≤ f(B)`
* f is **contravariant** if `A ≤ B` implies that `f(B) ≤ f(A)`
* f is **invariant** if neither of the above holds

Let's consider an example. Let `f(A) = List<A>` where List is declared by

```java
class List<T> { ... } 
```

Is f covariant, contravariant, or invariant? 
* **Covariant** would mean that a `List<String>` is a subtype of `List<Object>`, 
* **contravariant** that a `List<Object>` is a subtype of `List<String>`, and 
***invariant** that neither is a subtype of the other, 
i.e. `List<String>` and `List<Object>` are **inconvertible** types.

In Java, the latter is true, we say (somewhat informally) that **generics are invariant**.

Another example. Let `f(A) = A[]`. Is f covariant, contravariant, or invariant? 
That is, 
* is String[] a subtype of Object[], 
* Object[] a subtype of String[], or 
* is neither a subtype of the other? 

(Answer: In Java, arrays are covariant)

This was still rather abstract. To make it more concrete, 
let's look at which operations in Java are defined in terms of the subtype relation. 
The simplest example is assignment. The statement

```java
x = y;
```

will compile only if `typeof(y) ≤ typeof(x)`. That is, we have just learned that the statements

```java
ArrayList<String> strings = new ArrayList<Object>();
ArrayList<Object> objects = new ArrayList<String>();
```

will not compile in Java, but

```java
Object[] objects = new String[1];
```

will.

Another example where the subtype relation matters is a **method invocation expression**:

```java
result = method(a);
```

Informally speaking, this statement is evaluated by assigning the value of `a` 
to the method's first parameter, then executing the body of the method, 
and then assigning the methods return value to result. 
Like the plain assignment in the last example, the "right hand side" must be 
a subtype of the "left hand side", i.e. this statement can only be valid if 
`typeof(a) ≤ typeof(parameter(method))` and `returntype(method) ≤ typeof(result)`. 
That is, if method is declared by:

```java
Number[] method(ArrayList<Number> list) { ... }
```

none of the following expressions will compile:

```java
Integer[] result = method(new ArrayList<Integer>());
Number[] result = method(new ArrayList<Integer>());
Object[] result = method(new ArrayList<Object>());
```

but

```java
Number[] result = method(new ArrayList<Number>());
Object[] result = method(new ArrayList<Number>());
```

will.

Another example where subtyping matters is **overriding**. 
Consider:

```java
Super sup = new Sub();
Number n = sup.method(1);
```

where

```java
class Super {
    Number method(Number n) { ... }
}

class Sub extends Super {
    @Override 
    Number method(Number n);
}
```

Informally, the runtime will rewrite this to:

```java
class Super {
    Number method(Number n) {
        if (this instanceof Sub) {
            return ((Sub) this).method(n);  // *
        } else {
            ... 
        }
    }
}
```

For the marked line to compile, 
the **method parameter** of the overriding method must be a supertype of the method parameter of the overridden method, and 
the **return type** a subtype of the overridden method's one. 
Formally speaking, `f(A) = parametertype(method asdeclaredin(A))` must at least 
be contravariant, and if `f(A) = returntype(method asdeclaredin(A))` must at least be covariant.

Note the "at least" above. Those are minimum requirements any reasonable 
statically type safe object oriented programming language will enforce, 
but a programming language may elect to be more strict. 
In the case of Java 1.4, parameter types and method return types must be 
identical (except for type erasure) when overriding methods, i.e.   
`parametertype(method asdeclaredin(A)) = parametertype(method asdeclaredin(B))`   
when overriding. Since Java 1.5, covariant return types are permitted when 
overriding, i.e. the following will compile in Java 1.5, but not in Java 1.4:

```java
class Collection {
    Iterator iterator() { ... }
}

class List extends Collection {
    @Override 
    ListIterator iterator() { ... }
}
```

I hope I covered everything - or rather, scratched the surface. 
Still I hope it will help to understand the abstract, 
but important concept of **type variance**.


## Q & A ##

### Question: Explain Invariance, covariance and contravariance in Java ###

### Answer ###

This is not specific to Java, but has to do with the properties of certain types.

For example, with the function type

```
 A -> B                 // functional notation
 public B meth(A arg)   // how this looks in Java 
```

we have the following:

Let C be a subtype of A, and D be a subtype of B. Then the following is valid:

```
 B b       = meth(new C());  // B >= B, C < A
 Object o  = meth(new C());  // Object > B, C < A
```

but the follwoing are invalid:

```
 D d       = meth(new A());        // because D < B
 B b       = meth(new Object());   // because Object > A
```

hence, to check whether a call of meth is valid, we must check

```
The expected return type is a supertype of the declared return type.
The actual argument type is a subtype of the declared argument type.
```

This is all well known and intuitive. By convention we say that 
the return type of a function is **covariant**, and 
the argument type of a method is **contravariant**.

With parameterized types, like List, we have it that the argument type is 
invariant in languages like Java, where we have mutability. We can't say 
that a list of C's is a list of A's, because, if it were so, we could store 
an A in a list of Cs, much to the surprise of the caller, who assumes only Cs in the list. 
However, in languages where values are immutable, like Haskell, 
this is not a problem. Because the data we pass to functions cannot be mutated, 
a list of C actually is a list of A if C is a subtype of A. 
(Note that Haskell has no real subtyping, but has instead the related notion of 
"more/less polymorphic" types.)

<hr/>

## The Get and Put Principle in bounded wildcard ##

* Use extends only when you intend to get values out of a structure or Collection, 
* use super only when you intend to put values into a structure or Collection.

This also implies: 
* don’t use any wildcards when you intend to both get and put values into and out of a structure.

```java
// Copy all elements, subclasses of T, from source to dest 
// which contains elements that are superclasses of T.
public static <T> void copy(List<? super T> dest, List<? extends T> source) {
   for (int i = 0; i < source.size(); i++) {
       dest.set(i, source.get(i));
   }
}                     
```

```java
// Extends wildcard violation
List<Integer> integers = new LinkedList<Integer>();
List<? extends Number> numbers = integers;   
numbers.get(i);                                 // Works fine!
numbers.add(3);                                 // Won't compile!
```

```java
// Super wildcard violation
List<Number> numbers = new LinkedList<Number>();
List<? super Integer> integers = numbers;
numbers.add(3);                                 // Works fine!
int i = numbers.get(0);                         // Won't' compile!
Object o = numbers.get(0);        // Works fine since object is the upper bound!
```

## Covariance and Contravariance In Java ##

I have found that in order to understand covariance and contravariance a few 
examples with Java arrays are always a good start.

### Arrays Are Covariant ###

Arrays are said to be covariant which basically means that, 
given the subtyping rules of Java, an array of type T[] may 
contain elements of type T or any subtype of T. For instance:

```
Number[] numbers = new Number[3];
numbers[0] = new Integer(10);
numbers[1] = new Double(3.14);
numbers[2] = new Byte(0);
```

But not only that, the subtyping rules of Java also state that an array S[] is a subtype 
of the array T[] if S is a subtype of T, therefore, something like this is also valid:

```
Integer[] myInts = {1,2,3,4};
Number[] myNumber = myInts;
```

Because according to the subtyping rules in Java, an array Integer[] 
is a subtype of an array Number[] because Integer is a subtype of Number.

But this subtyping rule can lead to an interesting question: what would happen if we try to do this?

```
myNumber[0] = 3.14; //attempt of heap pollution
```

This last line would compile just fine, but if we run this code, we would get an ArrayStoreException 
because we’re trying to put a double into an integer array. The fact that we are accessing the array 
through a Number reference is irrelevant here, what matters is that the array is an array of integers.

This means that we can fool the compiler, but we cannot fool the run-time type system. 
And this is so because arrays are what we call a reifiable type. This means that at run-time Java 
knows that this array was actually instantiated as an array of integers which simply happens 
to be accessed through a reference of type Number[].

So, as we can see, one thing is the actual type of the object, an another thing is the type of the 
reference that we use to access it, right?

### The Problem with Java Generics ###

Now, the problem with generic types in Java is that the type information for type parameters is 
discarded by the compiler after the compilation of code is done; therefore this type information 
is not available at run time. This process is called type erasure. There are good reasons for 
implementing generics like this in Java, but that’s a long story, and it has to do with binary 
compatibility with pre-existing code.

The important point here is that since at run-time there is no type information, 
there is no way to ensure that we are not committing heap pollution.

Let’s consider now the following unsafe code:

```
List<Integer> myInts = new ArrayList<Integer>();
myInts.add(1);
myInts.add(2);
List<Number> myNums = myInts; //compiler error
myNums.add(3.14); //heap polution
```

If the Java compiler does not stop us from doing this, the run-time type system cannot stop us either, 
because there is no way, at run time, to determine that this list was supposed to be a list of integers only. 
The Java run-time would let us put whatever we want into this list, when it should only contain integers, 
because when it was created, it was declared as a list of integers. That’s why the compiler rejects 
line number 4 because it is unsafe and if allowed could break the assumptions of the type system.

As such, the designers of Java made sure that we cannot fool the compiler. If we cannot fool the 
compiler (as we can do with arrays) then we cannot fool the run-time type system either.

As such, we say that generic types are non-reifiable, since at run time we cannot determine the 
true nature of the generic type.

Evidently this property of generic types in Java would have a negative impact on polymorphism. 
Let’s consider now the following example:

```
static long sum(Number[] numbers) {
   long summation = 0;
   for(Number number : numbers) {
      summation += number.longValue();
   }
   return summation;
}
```

Now we could use this code as follows:

```
Integer[] myInts = {1,2,3,4,5};
Long[] myLongs = {1L, 2L, 3L, 4L, 5L};
Double[] myDoubles = {1.0, 2.0, 3.0, 4.0, 5.0};
System.out.println(sum(myInts));
System.out.println(sum(myLongs));
System.out.println(sum(myDoubles));
```

But if we attempt to implement the same code with generic collections, we would not succeed:

```
static long sum(List<Number> numbers) {
   long summation = 0;
   for(Number number : numbers) {
      summation += number.longValue();
   }
   return summation;
}
```

Because we we would get compiler errors if you try to do the following:

```
List<Integer> myInts = asList(1,2,3,4,5);
List<Long> myLongs = asList(1L, 2L, 3L, 4L, 5L);
List<Double> myDoubles = asList(1.0, 2.0, 3.0, 4.0, 5.0);
System.out.println(sum(myInts)); //compiler error
System.out.println(sum(myLongs)); //compiler error
System.out.println(sum(myDoubles)); //compiler error
```

The problem is that now we cannot consider a list of integers to be subtype of a list of numbers, 
as we saw above, that would be considered unsafe for the type system and compiler rejects it immediately.

Evidently, this is affecting the power of polymorphism and it needs to be fixed. 
The solution consists in learning how to use two powerful features of Java generics 
known as covariance and contravariance.

### Covariance ###

For this case, instead of using a type T as the type argument of a given generic type, 
we use a wildcard declared as ? extends T, where T is a known base type.

With covariance we can read items from a structure, but we cannot write anything into it. 
All these are valid covariant declarations.

```java
List<? extends Number> myNums = new ArrayList<Integer>();
List<? extends Number> myNums = new ArrayList<Float>();
List<? extends Number> myNums = new ArrayList<Double>();
```

And we can read from our generic structure myNums by doing:

```java
Number n = myNums.get(0);
```

Because we can be sure that whatever the actual list contains, it can be upcasted to a 
Number (after all anything that extends Number is a Number, right?)

However, we are not allowed to put anything into a covariant structure.

```
myNumst.add(45L); //compiler error
```

This would not be allowed because the compiler cannot determine what is the actual type of the object 
in the generic structure. It can be anything that extends Number (like Integer, Double, Long), 
but the compiler cannot be sure what, and therefore any attempt to retrieve a generic value is 
considered an unsafe operation and it is immediately rejected  by the compiler. 
So we can read, but not write.

### Contravariance ###

For contravariance we use a different wildcard called ? super T, where T is our base type. 
With contravariance we can do the opposite. We can put things into a generic structure, 
but we cannot read anything out of it.

```java
List<Object> myObjs = new List<Object();
myObjs.add("Luke");
myObjs.add("Obi-wan");
List<? super Number> myNums = myObjs;
myNums.add(10);
myNums.add(3.14);
```

In this case, the actual nature of the object is  List of Object, and through contravariance, we can put a Number in it, basically because a Number has Object as its common ancestor. As such, all numbers are also objects, and therefore this is valid.

However, we cannot safely read anything from this contravariant structure assuming that we will get a number.

```
Number myNum = myNums.get(0); //compiler-error
```

As we can see, if the compiler allowed us to write this line, we would get a ClassCastException at run time. So, once again, the compiler does not run the risk of allowing this unsafe operation and rejects it immediately.

### Get/Put Principle ###

In summary, we use covariance when we only intend to take generic values out of a structure. We use contravariance when we only intend to put generic values into a structure and we use an invariant when we intend to do both.

The best example I have is the following that copies any kind of numbers from one list into another list. It only gets items from the source, and it only puts items in the destiny.

```
public static void copy(List<? extends Number> source,
                        List<? super Number> destiny) {
   for(Number number : source) {
      destiny.add(number);
   }
}
```

Thanks to the powers of covariance and contravariance this works for a case like this:

```
List<Integer> myInts = asList(1,2,3,4);
List<Integer> myDoubles = asList(3.14, 6.28);
List<Object> myObjs = new ArrayList<Object>();
copy(myInts, myObjs);
copy(myDoubles, myObjs);
```

<hr/>

## Bounded and Unbouded Wildcard ##

### Example of Bounded and Unbounded wildcards in Java Generics ###

Java Collection frameworks has several examples of using bounded and unbounded wildcards in generics. 
Utility method provided in Collections class accepts parametrized arguments. 
`Collections.unmodifiableSet(Set<? extends T> s)` and 
`Collections.unmodifiableMap(Map<? extends K,? extends V> m)` 
are written using  bounded wildcards which allows them to operate on either 
Collection of T or Collection of sub class or super class of T. 

### When to use super and extends wildcards in Generics Java ###

Since there are two kinds of bounded wildcards in generics, **super** and **extends**, 
When should you use super wildcard and when should you extends wildcards. 
Joshua Bloch in Effective Java book has suggested 
> **Producer extends, Consumer super** 
mnemonic regarding use of bounded wildcards. 

So, if type T is used as producer then use <? extends T>  and 
if type T represent consumer then use <? super T> bounded wildcards. 

Bounded wildcards in generics also increase flexibility of any API. 
To me it is a question of requirement, if a method also needs to accept 
any implementation of T then use extends wildcards.

### Difference between ArrayList<? extends T>  and ArrayList<? super T> ###

This is one of popular generics interview question, which is asked to check 
whether you are familiar to bounded wildcards in generics. 
both **<? extends T>** and **<? super T>** represent bounded wildcards, 
one will accept only T or sub class while other will accept T or super class. 
bounded wildcards gives more flexibility to methods which can operate on 
collection of T or its sub class. 
If you look at java.util.Collections class you will find several example 
of bounded wildcards in generics method. e.g. 
Collections.unmodifiableSet(Set<? extends T> s) will accept Set of 
type T or Set of sub class of T.

That's all on what is bounded wildcards in generics. 
both bounded and unbounded wild cards provides lot of flexibility on 
API design specially because Generics is not co-variant and `List<String>` 
can not be used in place of `List<Object>`. Bounded wildcards allows you 
to write methods which can operate on Collection of Type as well as 
Collection of Type subclasses.

<hr/>


## Arrays of Generic types ##

This post examines differences between arrays and generics and finds out 
how we can create arrays of generic types in Java.

### Let’s start with Arrays ###

In Java, Arrays are covariant, which means that if B is a subtype of A, B[] is also subtype of A[].

Is there a problem here

Yes, there is a problem. Suppose, we have array of integers. We can assign these array of 
integers to array of numbers and put a double value as shown in the following program.

```java
package com.code.revisited.generics;
 
public class CovariantArrays {
 
    /**
     * This program throws runtime error. Because arrays are covariant,
     * assigning integer array to number array is successful at compile time
     * But, arrays enforce their element types at runtime. ArrayStoreException
     * is thrown while adding double to integer array
     * 
     * @param args
     */
    public static void main(String[] args) {
 
        Integer[] intArray = new Integer[5];
        Number[] objArray = intArray;
        objArray[0] = 1.0;
 
    }
 
}
```

Though this programs compiles successfully, It throws ArrayStoreException at runtime 
since arrays are reified and enforce their element types at runtime.
Though It is illegal that integer container can’t be assigned to Number container, 
We had to wait till we run this program. It’s hard to find bugs at runtime than compile time.

### Invariant generics ###

The following program perform the same operations as sated above.

```java
package com.code.revisited.generics;
 
import java.util.ArrayList;
import java.util.List;
 
/**
 * 
 * @author sureshsajja
 * 
 */
public class InvariantGenerics {
 
    /**
     * This program throws compile error. Because generics are invariant, List of integers are not
     * compatible with list of numbers
     * 
     * @param args
     */
    public static void main(String[] args) {
        List<Integer> listofInts = new ArrayList<Integer>();
        List<Number> listOfNums = listofInts;
        listOfNums.add(1.0);
 
    }
 
}
```

This program throws compile error. Generics are **invariant** which means
that if B is a subtype of A, `List<B>` is not subtype of `List<A>`. And also, 
Generics are implemented by **erasure**. This means that they enforce their 
type constraints only at compile time and discard (or erase) their 
element type information at runtime. Erasure is what allows generic types 
to interoperate freely with legacy code that does not use generics.
It is preferred to use generic list over arrays because any bugs will be 
surfaced at compile time.

As a consequence of the fact, arrays are covariant but generics are not, 
we are not allowed to create array of generic types unless the type 
argument is an unbounded wildcard.

### Why arrays of generic types ###

Suppose, if you want to implement ArrayList<E> as follows,

```java
public class MyArrayList<E> {
 
    private E[] elements;
    
    public MyArrayList(int size){
        elements = new E[size];
    }
 
}
```

But above code does not work. The compiler doesn’t know what type E really 
represents, so it cannot instantiate an array of type E.

### How can we create arrays of generic types ###

**One workaround** is to create an Object[] and then cast it (generating a warning) to E[]:

```java
elements = (E[])new Object[size];
```

**Second workaround** is through reflection by passing a 
class literal (Foo.class) of element type into the constructor, 
so that the implementation could know, at runtime, the value of E.

```java
public MyArrayList(Class<E> elementType, int size){
    elements = (E[]) Array.newInstance(elementType, size);
}
```

<hr/>


## CO-VARIANCE & CONTRA-VARIANCE ##

Generics/Parametrization is something that has always kept me on the hook 
even after good amount of experience in it. 
There have been some realizations on the go, but to explain that I will first have to explain some basics. 


Before starting, there is one very important theorem called 
**Liskov Substitution principle** in object oriented world. 
This should always be behind your mind while designing API’s. 
It states:

> Let q(x) be a property provable about objects x of type T. 
> Then q(y) should be provable for objects y of type S where S is a subtype of T.

The above means:

> If “A” extends “B” (A <: B), then 
> all the things that one can do with “B” should also be legal with “A”.

### Co-variance & Contra-variance ###

Wiki will give you a formal definition. Crudely:

* **Co-variance** ( **<:** ) is converting type from wider type to narrow. In Java terms  **? extends E**
* **Contra-variance** ( **>:** ) is converting type from narrow to wider. In Java terms: **? super E**
* **In-variant** cannot be converted

Where E is generic type declared and **?** is a wild card which crudely means **Anytype**

So for example:

```java
class Animal { }
 
class Dog extends Animal { }
 
class Example{
 
    void run() {
        Dog dog = new Dog();
        getAnimal(dog);
    }
 
    void getAnimal(Animal animal){
        .....
    }
}
```

In the above example, one is ultimately doing

```java
Animal animal = dog
```

So you are converting a dog type (narrow) to Animal type (wider). 
This is contra-variance. The reverse of it is Co-variance. 
The above paradigm is also followed in C# and python.

For co-variance:

```java
class A {
    Animal getCreature(){
        return new Dog();
    }
}

class B extends A {
    Dog getCreature() {
        return new Dog();
    }
}
```

Check the return type of the getCreature function. getCreature() ultimately returns an Animal. 
But it returns type of Dog in class B. This is co-variance. 
The above is perfectly legal, because the code that invokes getCreature() of class A expects 
an Animal in return; which class B provides a Dog (ultimately dog is an animal. Vice-versa wont work, think!)

Invariance is when the type cannot be converted.

By default, in Java, C#, and python functions:
* **return types** are co-variant, and 
* **argument types** are invariant.

### Points to Remember ###

I will raise some points now and the explanation will be given by the end. 
The whole point of the post are the below points.

* Contravariant types can only appear in places, where you write something 
  (i.e. parameter types) and 
  covariant types can only appear in places, where you read something. [P1]
* Co-variance is only suitable when the data is immutable. More on it below. [P2]

I will first raise the issues caused in Java by not following [P1] i.e Arrays disaster 
for which Generics (type erasure) will be discussed.


### Arrays in Java ###

Arrays in Java are Co-variant. Which basically means if `String` extends `Object`
then `String[]` also extends `Object[]`.

Do you see  any pitfall’s above?

Lets look at the following example

Example – 1
```java
String[] a = new String[]{"Hello","Hey"};
Object[] b = a;                             //line 2
b[0]       = new Integer(23);               //line 3
a[0]       = ???
```

Now because arrays in Java are co-variant, you can do the above. 
`a[0]` now will refer to an Integer which is illegal. 
To prevent this, line3 will throw an **ArrayStoreException** at runtime. 
Truly this should actually be caught at compile time. Thanks to co-variance of arrays for behavior. 
Because for some reason arrays were decided to be **co-variant** and 
they are **reifiable**, i.e. they retain the type information at runtime. 
Which allowed us to do line-2 but also gave us line-3. 
To prevent this, they manually had to check the type info for every access of array 
which again is bad and extra overhead.

But why were arrays made co-variant? 

Java founders wanted to have Generics right from day 0 (Generics were introduced in Java 5).
Generics follow **type erasure**, i.e. the type is checked for compatibility and erased at compile time. 
But due to lack of time, they had to postpone implementing Generics. 
Now without generics a method like the following would had been impossible. 
Unless Arrays were made co-variant.

```java
Arrays.sort(Object[] ob)
```

The above method takes an array of objects and sorts them based on the Comparables. 
Hence a single method will suffice. Without covariance of arrays, they will be a 
need to write a separate method for each of the Data Type. Hence the decision.

Another consequence is that you cannot instantiate an array of a generic type. 
Nothing special for generics.

Example - 2
```java
List<String>[] l  = new ArrayList<String>[10]();     //line 1
Object[]       a  = l;
List<Integer>  l2 = new ArrayList<Integer>();
a[0] = l2;
l[0] = ???   //list of string or integer?
```

Hence line-1 is illegal. Though arrays with wild-cards are allowed i.e. `List<?>`. 
Why is left for user to analyse. 
Similarly, 
why Generics were decided to be invariant is again left to the user 
(Hint: Similar to the above example, try adding any animal `pig` to list of `dogs`) 
So now we have seen that writing onto **co-variant mutable types** always throws one or some other trouble. 
This gives gives rise to 2 questions:

* If Arrays were invariant, would it solve the purpose?
* If Arrays were left co-variant, what change can be made to make it correct? 
  Has mutability has got anything to do with it?

Suppose Arrays were invariant, then the bugs would be detected right at compile time 
as raised in Example -1. Thus Invariance of Arrays looks to be the right way forward. 
Scala gets this right. 

Lets look at the second question. Suppose Generics below were co-variant. 
Lets consider immutable List for example. (By ImmutableList I mean, any addition to 
the list creates a new List by copying the previous elements and adding the latest element. 
Similar to CopyOnWriteArrayList just that addition returns a new List every time)

```java
ImmutableList<String> a = new ImmutableList<String>();
ImmutableList<String> b = a.add("Hey");                 //line2
ImmutableList<Object> x = a;                            //line 3
ImmutableList<Object> y = b.add(new Object);
```

In Line-2, adding a string returned immutable list containing only `Hey`. 
ImmutableList `a` still is empty. 
Now on adding Object in Line-4 gave a new List containing (`Hey` and an `Object`) to become `y`. 
The return type of list is `Object`. Hence it is safe at all the stages. 
As now `y` is a list of Objects containing a `Hey` and an `Object`.

So it turns out that with immutability, co-variance can be used safely. 
Com’on Co-variance is such a lovely tool. 
It lets you do stuff like  if `A <:  B` then it is handy to have `C[A] <: C[B]`. Isn’t it?

So remember [P2] (from last post). Preferably with rights co-variance can be trouble-some. 
Perfect for Reads just like returns in functions.

Interestingly **Contra-variance** is made for writes. More on it in the next post.
