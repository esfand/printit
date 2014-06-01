# Programming with lambda expressions #

**A mathematical example demonstrates the power of lambdas in Java 8**

source: www.javaworld.com/article/2092260/java-se/java-programming-with-lambda-expressions.html

While there are many applications for 
lambda expressions, this article focuses on a specific example that occurs frequently 
in mathematical applications; namely, the need to pass a function to an algorithm.

Many applications in mathematics require that a function be passed as a parameter 
to an algorithm. Examples from college algebra and basic calculus include solving 
an equation or computing the integral of a function. For over 15 years Java has 
been my programming language of choice for most applications, but it was the first 
language that I used on a frequent basis that did not allow me to pass a function 
(technically a pointer or reference to a function) as a parameter in a simple, 
straightforward manner. That shortcoming is changed with the release of Java 8.

The power of lambda expressions extends well beyond a single use case, but 
studying various implementations of the same example should leave you with a solid 
sense of how lambdas will benefit your Java programs. In this article I will use a 
common example to help describe the problem, then provide solutions written in 
**C++**, **Java before lambda expressions**, and **Java with lambda expressions**. 

## The Simplson's Rule ##

The example used throughout this article is **Simpson's Rule** from basic calculus. 
Simpson's Rule, or more specifically Composite Simpson's Rule, is a numerical 
integration technique to approximate a definite integral.  Don't worry if you 
are unfamiliar with the concept of a definite integral; what you really need 
to understand is that Simpson's Rule is **an algorithm that computes a real 
number based on four parameters**:

* A function that we want to integrate.
* Two real numbers a and b that represent the endpoints of an interval [a,b] 
  on the real number line. (Note that the function referred to above should be 
  continuous on this interval.)
* An even integer n that specifies a number of subintervals. 
  In implementing Simpson's Rule we divide the interval [a,b] into n subintervals.

To simplify the presentation, let's focus on the programming interface and not on 
the implementation details. We will use type double for parameters a and b, 
and we will use type int for parameter n. The function to be integrated will take a 
single parameter of type double and a return a value of type double.
   
## Function parameters in C++ ## 

**Download the C++ source code**.

To provide a basis for comparison, let's start with a C++ specification. 
When passing a function as a parameter in C++, I usually prefer to specify 
the signature of the function parameter using a typedef. Listing 1 shows a 
C++ header file named simpson.h that specifies both the typedef for the 
function parameter and the programming interface for a C++ function named integrate. 
The function body for integrate is contained in a C++ source code file named 
simpson.cpp (not shown) and provides the implementation for Simpson's Rule.

**Listing 1. C++ header file for Simpson's Rule**
```c++
#if !defined(SIMPSON_H)
#define SIMPSON_H
#include <stdexcept>
using namespace std;
typedef double DoubleFunction(double x);
double integrate(DoubleFunction f, double a, double b, int n)
    throw(invalid_argument);
#endif
```

Calling integrate is straightforward in C++. As a simple example, suppose 
that you wanted to use Simpson's Rule to approximate the integral of the sine 
function from 0 to π (PI) using 30 subintervals. (Anyone who has completed 
Calculus I should be able to compute the answer exactly without the help of a 
calculator, making this a good test case for the integrate function.) 
Assuming that you had included the proper header files such as <cmath> 
and "simpson.h", you would be able to call function integrate as shown in Listing 2.

**Listing 2. C++ call to function integrate**
```c++
double result = integrate(sin, 0, M_PI, 30);
```

That's all there is to it. In C++ you pass the sine function as easily as you pass the other three parameters.

## Another example ##

Instead of Simpson's Rule I could have just as easily used the Bisection Method (aka the Bisection Algorithm) for solving an equation of the form f(x) = 0. In fact, the source code for this article includes simple implementations of both Simpson's Rule and the Bisection Method.

  
## Java without lambda expressions ##

**Download the Java source code.**

Now let's look at how Simpson's Rule might be specified in Java. 
Regardless of whether or not we are using lambda expressions, 
we use the Java interface shown in Listing 3 in place of the C++ typedef 
to specify the signature of the function parameter.

**Listing 3. Java interface for the function parameter**
```java
public interface DoubleFunction {
    public double f(double x);
}
```

To implement Simpson's Rule in Java we create a class named Simpson that contains a method, 
integrate, with four parameters similar to what we did in C++. As with a lot of 
self-contained mathematical methods (see, for example, java.lang.Math), we will make 
integrate a static method. Method integrate is specified as follows:

**Listing 4. Java signature for method integrate in class Simpson**
```java
public static double integrate(DoubleFunction df, double a, double b, int n) {
    // . . .
}
```

Everything that we've done thus far in Java is independent of whether or not we will use 
lambda expressions. The primary difference with lambda expressions is in how we pass 
parameters (more specifically, how we pass the function parameter) in a call to 
method integrate. First I'll illustrate how this would be done in versions of 
Java prior to version 8; i.e., without lambda expressions. As with the C++ example, 
assume that we want to approximate the integral of the sine function from 0 to π (PI) 
using 30 subintervals.

### Using the Adapter pattern for the sine function ###

In Java we have an implementation of the sine function available in java.lang.Math, 
but with versions of Java prior to Java 8, there is no simple, direct way 
to pass this sine function to the method integrate in class Simpson. 
One approach is to use the Adapter pattern. In this case we would write a simple 
adapter class that implements the DoubleFunction interface and adapts it to call 
the sine function, as shown in Listing 5.

**Listing 5. Adapter class for method Math.sin**
```java
import com.softmoore.math.DoubleFunction;

public class DoubleFunctionSineAdapter implements DoubleFunction {
    public double f(double x) {
        return Math.sin(x);
      }
}
```

Using this adapter class we can now call the integrate method of class Simpson as shown in Listing 6.

**Listing 6. Using the adapter class to call method Simpson.integrate**
```java
DoubleFunctionSineAdapter sine = new DoubleFunctionSineAdapter();
double result = Simpson.integrate(sine, 0, Math.PI, 30);
```

Let's stop a moment and compare what was required to make the call to integrate in C++ 
versus what was required in earlier versions of Java. With C++, we simply called integrate, 
passing in the four parameters. With Java, we had to create a new adapter class and then 
instantiate this class in order to make the call. If we wanted to integrate several 
functions, we would need to write an adapter class for each of them.

We could shorten the code needed to call integrate slightly from two Java statements to one by creating the new instance of the adapter class within the call to integrate. Using an anonymous class rather than creating a separate adapter class would be another way to slightly reduce the overall effort, as shown in Listing 7.

**Listing 7. Using an anonymous class to call method Simpson.integrate**
```java
DoubleFunction sineAdapter = new DoubleFunction() {
    public double f(double x)
      {
        return Math.sin(x);
      }
};

double result = Simpson.integrate(sineAdapter, 0, Math.PI, 30);
```

Without lambda expressions, what you see in Listing 7 is about the least amount of code that you could write in Java to call the integrate method, but it is still much more cumbersome than what was required for C++. I am also not that happy with using anonymous classes, although I have used them a lot in the past. I dislike the syntax and have always considered it to be a clumsy but necessary hack in the Java language.

## Java with lambda and functional interfaces ##

Now let's look at how we could use lambda expressions in Java 8 to simplify the 
call to integrate in Java. Because the interface DoubleFunction requires the 
implementation of only a single method it is a candidate for lambda expressions. 
If we know in advance that we are going to use lambda expressions, we can annotate 
the interface with @FunctionalInterface, a new annotation for Java 8 that says we 
have a functional interface. Note that this annotation is not required, 
but it gives us an extra check that everything is consistent, 
similar to the @Override annotation in earlier versions of Java.

The syntax of a lambda expression is an argument list enclosed in parentheses, 
an arrow token (->), and a function body. The body can be either a 
statement block (enclosed in braces) or a single expression. Listing 8 shows 
a lambda expression that implements the interface DoubleFunction and is then 
passed to method integrate.

**Listing 8. Using a lambda expression to call method Simpson.integrate**
```java
DoubleFunction sine = (double x) -> Math.sin(x);
double result = Simpson.integrate(sine, 0, Math.PI, 30);
```

Note that we did not have to write the adapter class or create an instance of an anonymous class. 
Also note that we could have written the above in a single statement by substituting the lambda expression itself, (double x) -> Math.sin(x), for the parameter sine in the second statement above, eliminating the first statement. Now we are getting much closer to the simple syntax that we had in C++. But wait! There's more!

The name of the functional interface is not part of the lambda expression but can be inferred based on the context. The type double for the parameter of the lambda expression can also be inferred from the context. Finally, if there is only one parameter in the lambda expression, then we can omit the parentheses. Thus we can abbreviate the code to call method integrate to a single line of code, as shown in Listing 9.

**Listing 9. An alternate format for lambda expression in call to Simpson.integrate**
```java
double result = Simpson.integrate(x -> Math.sin(x), 0, Math.PI, 30);
```

But wait! There's even more!

### Method references in Java 8 ###

Another related feature in Java 8 is something called a method reference, which allows us to refer to an existing method by name. Method references can be used in place of lambda expressions as long as they satisfy the requirements of the functional interface. As described in the resources, there are several different kinds of method references, each with a slightly different syntax. For static methods the syntax is Classname::methodName. Therefore, using a method reference, we can call the integrate method in Java as simply as we could in C++. Compare Java 8 call shown in Listing 10 below with original C++ call shown in Listing 2 above.

**Listing 10. Using a method reference to call Simpson.integrate**
```java
double result = Simpson.integrate(Math::sin, 0, Math.PI, 30);
```

As a final thought, although I prefer writing the interface DoubleFunction 
because I think that it makes the code easier to understand, even that 
interface can be eliminated. Java 8 has a new package, java.util.function, 
that contains a number of commonly used functional interfaces. 
Many are expressed using generics, but there are specializations for 
primitive types. I'll leave the use of one of these interfaces in place of 
DoubleFunction as the proverbial exercise for the reader. (For a hint, 
look closely at DoubleUnaryOperator in java.util.function.)

## In conclusion ##

Overall I have found it a pleasure to code in Java, and it has been my preferred 
programming language for more than 15 years. There are (still) a few places where 
Java's syntax seems awkward, however. With the addition of lambda expressions, 
Java 8 will correct one of them. 

