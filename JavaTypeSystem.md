# Java Type System #

## How do parameterized types fit into the Java type system? ##

Instantiations of generic types have certain super-subtype relationship among each other 
and have a type relationship to their respective raw type.  These type relationships are 
relevant for method invocation, assignment and casts.

### Relevance of type relationships and type converstion rules in practice. ###

The type system of a programming language determines which types are convertible to which other types.  
These conversion rules have an impact on various areas of a programming language.  
One area where conversion rules and type relationships play role is casts and instanceof expressions.  
Other area is assignment compatibility and method invocation, where argument and 
return value passing relies on convertibility of the involved types. 

The type conversion rules determine which casts are accepted and which ones are rejected.  
For example, the types String and Integer have no relationship and for this reason the compiler rejects the attempt of a cast from String to Integer , or vice versa.  In contrast, the types Number and Integer have a super-subtype relationship; Integer is a subtype of Number and Number is a supertype of Integer . Thanks to this relationship, the compiler accepts the cast from Number to Integer , or vice versa.  The cast from Integer to Number is not even necessary, because the conversion from a subtype to a supertype is considered an implicit type conversion, which need not be expressed explicitly in terms of a cast; this conversion is automatically performed by the compiler whenever necessary.  The same rules apply to  instanceof expressions. 

The conversion rules define which types are assignment compatible.  Using the examples from above, we see that a String cannot be assigned to an Integer variable, or vice versa, due to the lack of a type relationship.  In contrast, an Integer can be assigned to a Number variable, but not vice versa.  A side effect of the super-subtype relationship is that we can assign a subtype object to a supertype variable, without an explicit cast anywhere.  This is the so-called widening reference conversion ; it is an implicit conversion that the compiler performs automatically whenever it is needed.  The converse, namely assignment of a supertype object to a subtype variable, is not permitted.  This is because the so-called narrowing reference conversion is not an implicit conversion.  It can only be triggered by an explicit cast. 

The rules for assignment compatibility also define which objects can be passed to which method.  An argument can be passed to a method if its type is assignment compatible to the declared type of the method parameter. For instance, we cannot pass an Integer   to a method that asks for String , but we can pass an Integer to a method that asks for a Number .  The same rules apply to the return value of a method.  
 

### Super-subtype relationships of parameterized types. ###

In order to understand how objects of parameterized types can be used in 
assignments, 
method invocations, and 
casts, 
we need an understanding of the relationship that parameterized types have 
among each other and with non-parameterized types. 
And we need to know the related conversion rules. 

We already mentioned super-subtype relationships and the related narrowing and  widening reference conversions .  
They exist since Java was invented, that is, among non-generic types.  
The super-subtype relationship has been extended to include parameterized types.  
In the Java 5.0 type system super-subtype relationships and the related 
narrowing/widening reference conversions exist among parameterized types, too.  
We will explain the details in separate FAQ entries.  
Here are some initial examples to get a first impression of the impact that 
type relationships and conversion rules have on method invocation. 

Consider a method whose declared parameter type is a wildcard parameterized type.  
A wildcard parameterized type acts as supertype of all members of the type family 
that the wildcard type denotes. 

Example (of widening reference conversion from concrete instantiation to wildcard instantiation): 

```java
void printAll( LinkedList<? extends Number> c) { ... }
LinkedList<Long> l = new LinkedList<Long>(); 
... 
printAll(l);  // widening reference conversion
```

We can pass a `List<Long>` as an argument to the printAll method that asks for a `LinkedList<? extends Number>`.  
This is permitted thanks to the super-subtype relationship between a wildcard instantiation and a concrete instantiation.
`LinkedList<Long>` is a member of the type family denoted by `LinkedList<? extends Number>` , and as such a
`LinkedList<Long>` is a subtype of `LinkedList<? extends Number>` .  
The compiler automatically performs a widening conversion from subtype to supertype and thus allows that a
`LinkedList<Long>` can be supplied as argument to the printAll method that asks for a `LinkedList<? extends Number>`.

Note that this super-subtype relationship between a wildcard instantiation and a member of the type family 
that the wildcard denotes is different from inheritance. Inheritance implies a super-subtype relationship as well,
but it is a special case of the more general super-subtype relationship that involves wildcard instantiations. 

We know inheritance relationships from non-generic Java.  
It is the relationship between a superclass and its derived subclasses, or 
a super-interface and its sub-interfaces, or 
the relationship between an interface and its implementing classes.

Equivalent inheritance relationships exists among instantiations of different generic types.  
The prerequisite is that the instantiations must have the same type arguments.  
Note that this situation differs from the super-subtype relationship mentioned above, 
where we discussed the relationship between **wildcard instantiations** and 
**concrete instantiations** of the same generic type, whereas we now talk of 
the relationship between instantiations of different generic types with identical type arguments. 

Example (of widening reference conversion from one concrete parameterized type to another concrete parameterized type): 

```java
void printAll( Collection<Long> c) { ... }
LinkedList<Long> l = new LinkedList<Long>(); 
... 
printAll(l);  // widening reference conversion
```

The raw types Collection and LinkedList have a super-subtype relationship; Collection is a supertype of LinkedList .  This super-subtype relationship among the raw types is extended to the parameterized types, provided the type arguments are identical: Collection<Long> is a supertype of LinkedList<Long> , Collection<String> is a supertype of LinkedList<String> , and so on. 

It is common that programmers believe that the super-subtype relationship among type arguments 
would  extend into the respective parameterized type.  
This is not true. 
Concrete instantiations of the same generic type for different type arguments have no type relationship. 
For instance, Number is a supertype of Integer , but `List<Number>` is not a supertype of `List<Integer>`.  
A type relationship among different instantiations of the same generic type exists only among 
wildcard instantiations and concrete instantiations, but never among concrete instantiations. 

Example (of illegal attempt to convert between different concrete instantiations of the same generic type): 

```java
void printAll( LinkedList<Number> c) { ... }
LinkedList<Long> l = new LinkedList<Long>(); 
... 
printAll(l);  // error; no conversion
```

Due to the lack of a type relationship between `LinkedList<Number>` and `LinkedList<Long>` 
the compiler cannot convert the `LinkedList<Long>` to a `LinkedList<Number>` and the 
method call is rejected with an error message.  
 
## Unchecked conversion of parameterized types. ##

With the advent of parameterized types a novel category of type relationship was added to the Java type system: 
the relationship between a **parameterized type** and the corresponding **raw type**. 
The conversion from a parameterized type to the corresponding raw type is a **widening reference conversion**
like the conversion from a subtype to the supertype. 
It is an implicit conversion.  An example of such a conversion is the conversion from a parameterized type such as `List<String>` or `List<? extends Number>` to the raw type List .  
The counterpart, namely the conversion from the raw type to an instantiation of the respective generic type, 
is the so-called  **unchecked conversion**. It is an automatic conversion, too, but 
the compiler reports an "unchecked conversion" warning.   
Details are explained in separate FAQ entries.  
Here are some initial examples to get a first impression of usefulness of unchecked conversions.  
They are mainly permitted for compatibility between generic and non-generic source code. 

Below is an example of a method whose declared parameter type is a raw type. 
The method might be a pre-Java-5.0 method that was defined before generic and 
parameterized types had been available in Java.  
For this reason it declares List as the argument type.  
Now, in Java 5.0, List is a raw type. 

Example (of a widening reference conversion from parameterized type to raw type): 

```java
void printAll( List c) { ... }
List<String> l = new LinkedList<String>(); 
... 
printAll(l);  // widening reference conversion
```

Source code such as the one above is an example of a fairly common situation, 
where non-generic legacy code meets generic Java 5.0 code.  
The printAll method is an example of legacy code that was developed before Java 5.0 and uses raw types.  
If more recently developed parts of the program use instantiations of the generic type `List`, 
then we end up passing an instantiation such as `List<String>` to the printAll method that declared the raw type List as its parameter type.  Thanks to the type relationship between the raw type and the parameterized type, the method call is permitted. It involves an automatic widening reference conversion from the parameterized type to the raw type. 
Below is an example of the conversion in the opposite direction.  We consider a method has a declared parameter type that is a parameterized type.  We pass a raw type argument to the method and rely on an unchecked conversion to make it work. 

Example (of an unchecked conversion from raw type to parameterized type): 

```java
void printAll( List<Long> c) { ... }
List l = new LinkedList(); 
... 
printAll(l);  // unchecked conversion
```

Like the previous example, this kind of code is common in situation where generic and non-generic code are mixed. 

The subsequent FAQ entries discuss details of the various type relationships and conversions among raw types, 
concrete parameterized types, bounded and unbounded wildcard parameterized types.  
