# The Get and Put Principle in bounded wildcard#

* Use extends only when you intend to get values out of a structure or Collection; 
* Use super only when you intend to put values into a structure or Collection.

This also implies: 
* don’t use any wildcards when you intend to both get and put 
  values into and out of a structure.

```java
// Copy all elements, subclasses of T, from source to dest 
// which contains elements that are superclasses of T.
public static <T> void copy(List<? super   T> dest, 
                            List<? extends T> source) {
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
Object o = numbers.get(0);                      // Works fine since object is the upper bound!
```
