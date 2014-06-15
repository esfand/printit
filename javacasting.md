## Java Explicit and Implicit Type Casting ##

Java supports Type casting as: 
up casting (widening reference conversion), and 
down casting (narrowing reference conversion). 
Cast operation can be written 
explicitly with the cast operator (T), or 
implicitly with no operator.

In Java, class and interface reference type can be converted from 
one type to another type using the cast operation in two ways:

1) **Widening Reference Conversion** - Type S is converted to type T, where S is a subtype of T. 
Widening reference conversion is also called **up casting**, because it converts a subtype to a supertype. 
Up casting is always allowed, because the reference object of a subtype is always compatible with a supertype. 
For example:

```java
String msg = new String("Hello");

// up casting from String to Object
Object obj = (Object) msg;
```

2) **Narrowing Reference Conversion** - Type T is converted to type S, where T is a supertype of S. 
Narrowing reference conversion is also called **down casting**, because it converts a supertype to a subtype. 
Down casting is always allowed only if the reference object is compatible with the subtype. 
If the reference object is not compatible with the subtype, a compilation error or 
runtime exception will be resulted. For example:

```java
Object obj = new String("Hello");

// down casting from Object to String
String msg = (String) obj;
```

There are 2 syntax formats to write a type casting operation:

1) **Explicit Casting** - Adding (cast-to type) on the left side of the cast-from type. For example,

```java
String msg = new String("Hello");

// explicit casting 
Object obj = (Object) msg;
```

2) **Implicit Casting** - Letting compiler automatically cast the type based on expression context. 
For example,

```java
String msg = new String("Hello");

// explicit casting 
Object obj = msg;
```
