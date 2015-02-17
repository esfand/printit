# CodeModel – Expression

This part of the tutorial is about JExpr and JExpression in CodeModel.

In the previous post I have used the JExpr class to write assignment code inside the body 
of some methods. Well, I must confess that I found JExpr and JExpression really hard to 
understand (let’s say this plainly: documentation did not help a bit, in this case).

From official documentation we could see that

JExpression defines a series of composer methods, which returns a 
complicated expression (by often taking other JExpressions as parameters. 
For example, you can build `5+2` by `JExpr.lit(5).add(JExpr.lit(2))`

and, about JExpr:

Factory methods that generate various JExpressions.

Some example of JExpr (static) methods are:

*  `_null()`;
*  `_super()`: Returns a reference to “super”, an implicit reference to the super class.
*  `_this()`: Returns a reference to “this”, an implicit reference to the current object.

So, easy enough, when you need a reference to one of this, you have to use the 
JExpr method that, in turn, return a JExpression that represent it.

The following snippet creates a constructor method that accept an int as parameter, and 
we assign this parameter to a private field:

```java        
String className = "net.cardosi.MyNewClassD";
JDefinedClass definedClass = codeModel._class(className);
String fieldName = "intVar";
JFieldVar field = definedClass.field(JMod.PRIVATE, int.class, fieldName);
JMethod constructorMethod = definedClass.constructor(JMod.PUBLIC);
String intVarParameter = "intVarParam";
constructorMethod.param(int.class, intVarParameter);
JBlock block = constructorMethod.body();
block.assign(JExpr._this().ref(fieldName), JExpr.ref(intVarParameter));
```

Here’s the resultant code:

```
package net.cardosi;

public class MyNewClassD {

    private int intVar;

    public MyNewClassD(int intVarParam) {
        this.intVar = intVarParam;
    }
}
```

I know I could have avoided the this reference, in this case, but anyway…

Another interesting detail is the use of JExpr.ref(), that’s needed to retrieve 
a reference to a given field (more exactly, it returns a JFieldRef reference for 
a field with a given name). So, the line:

```java
block.assign(JExpr._this().ref(fieldName), JExpr.ref(intVarParameter));
```

assign the value of the field referenced by intVarParameter (in this case, a method parameter) 
to the field referenced by fieldName of the this object.

Another use of JExpr could be the creation of a JExpression representing a 
literal (boolean, double, float, char, int, long, and String):

```java        
String className = "net.cardosi.MyNewClassD";
JDefinedClass definedClass = codeModel._class(className);
String stringName = "stringVar";
JFieldVar stringField = definedClass.field(JMod.PRIVATE, 
                                           String.class, 
                                           stringName);
JMethod constructorMethod = definedClass.constructor(JMod.PUBLIC);
JBlock block = constructorMethod.body();
block.assign(JExpr._this().ref(stringName), 
             JExpr.lit("DEFAULT VALUE"));
```	
	
Here’s the newly generated class, where the “stringVar” is initialized with 
*DEFAULT VALUE* inside the constructor:

```java
package net.cardosi;

public class MyNewClassD {

    private String stringVar;

    public MyNewClassD() {
        this.stringVar = "DEFAULT VALUE";
    }
}
```

Now, it’s time to look at JExpression a little bit closer. 
Like JExpr has static methods to retrieve some specific JExpressions, JExpression 
has instance methods to retrieve specific JExpressions.

Let’s start with one of the simplest:

`JExpression.not();` that returns a “!” (Who may have guessed it ?)

In the following modification of the same beloved code, we create a boolean field and 
inside the constructor we assign to it the negation of a boolean parameter:

```
String className = "net.cardosi.MyNewClassE";
JDefinedClass definedClass = codeModel._class(className);
String booleanName = "booleanVar";
JFieldVar booleanField = definedClass.field(JMod.PRIVATE, 
                                            boolean.class, 
                                            booleanName);
JMethod constructorMethod = definedClass.constructor(JMod.PUBLIC);
String booleanVarParameter = "booleanVarParameter";
constructorMethod.param(boolean.class, booleanVarParameter);
JBlock block = constructorMethod.body();
block.assign(JExpr._this().ref(booleanName), 
             JExpr.ref(booleanVarParameter).not());
codeModel.build(new File("."));
```	
	
Here is the result:

```java
package net.cardosi;

public class MyNewClassE {

    private boolean booleanVar;

    public MyNewClassE(boolean booleanVarParameter) {
        this.booleanVar = (!booleanVarParameter);
    }
}
```

The line:

```java
block.assign(JExpr._this().ref(booleanName), 
             JExpr.ref(booleanVarParameter).not());
```

* retrieve a JExpression referencing the booleanVarParameter
* apply a *negation* to that reference
* assign the resulting value to the field referenced by booleanName.

Ah! Now we are cooking! And, what about creating an integer variable with a default value, and 
then give to it the sum of another value and a number?

```java
String className = "net.cardosi.MyNewClassF";
JDefinedClass definedClass = codeModel._class(className);
String intName = "intVar";
JFieldVar intField = (JFieldVar) definedClass.field(JMod.PRIVATE, 
                                                    int.class, 
                                                    intName)
                                             .init(JExpr.lit(14));
JMethod constructorMethod = definedClass.constructor(JMod.PUBLIC);
String intVarParameter = "intVarParameter";
constructorMethod.param(int.class, intVarParameter);
JBlock block = constructorMethod.body();
block.assign(JExpr._this().ref(intName), 
             JExpr.ref(intVarParameter)
     .plus(JExpr.lit(35)));
codeModel.build(new File("."));
```

Here it is

```java
package net.cardosi;

public class MyNewClassF {

    private int intVar = 14;

    public MyNewClassF(int intVarParameter) {
        this.intVar = (intVarParameter + 35);
    }
}
```

As a plus I have included the initialization of the variable in its declaration:

```java
JFieldVar intField = (JFieldVar) definedClass.field(JMod.PRIVATE, 
                                                    int.class, 
                                                    intName).init(JExpr.lit(14));
```

Since 'JExpr.lit()' is overloaded for different kind of literals (boolean, double, float, 
char, int, long, and String), the corresponding type of variables can be initialized that way.

Maybe I was a little bit pedantic in this section of the tutorial, but in the same time 
I have the impression to have just scratched the surface of JExpr and JExpression.
Anyway, I hope it will be enough for you as starting point.


#Java code generation with CodeModel – Generics

This part of the tutorial is about Generics management with CodeModel.

Now that we have covered basics, javadoc and jexpressions, let’s take a look at how to manage Generics with CodeModel.

First of all, to make a class Generic you have to call the following method to the JDefinedClass created to build it:

```java 
JDefinedClass.generify(String name); 
```

Let’s see actual code and result:

```java 
String concreteName = "net.cardosi.MyNewGenericClass";
String genericTypeName = "T";
JClass genericT = codeModel.ref(genericTypeName);
JDefinedClass concreteClass = codeModel._class(concreteName);
concreteClass.generify(genericTypeName);
```

```java
package net.cardosi;

public class MyNewGenericClass<T> {

}
```

Now, let’s add a “generic” field to it.. let’s see… maybe a List? For this we need a reference to:

* A JClass representing the type parameter (“T” in our example);
* A JClass representing the raw interface (“List” in our example);
* A JClass representing the interface’ generic type (“List<T>” in our example);
* A JClass representing the raw implementation of the interface (“ArrayList” in our example);
* A JClass representing the implementation’ generic type of the interface (“ArrayList<T>” in our example);
* 
With all that, we can define and instantiate a Generic field

```java 
JClass genericT = codeModel.ref(genericTypeName);
JClass rawLLclazz = codeModel.ref(List.class);
JClass fieldClazz = rawLLclazz.narrow(genericT);
JClass rawArrayListClass = codeModel.ref(ArrayList.class);
JClass arrayListFieldClazz = rawArrayListClass.narrow(genericT);
JExpression newInstance = JExpr._new(arrayListFieldClazz).arg(JExpr.lit(0));
String listFieldString = "genericList";
JFieldVar listFieldVar = concreteClass.field(JMod.PRIVATE, 
                                             fieldClazz, 
                                             listFieldString);
listFieldVar.init(newInstance);
```

Here’s what we get (note that the imports are automatically generated by CodeModel:

```java 
package net.cardosi;

import java.util.ArrayList;
import java.util.List;

public class MyNewGenericClass<T>{

    private List<T> genericList = new ArrayList<T>(0);

}
```

Last thing to do, now, is to provide a method to insert something in our List. 
It is not different from what we saw in Basics, but we have to use the JClass
representing the type parameter. More over, we have to “literally” invoke the 
method “add” on the reference to the list field:

```java 
String adderParameterString = "toAddParam";
JVar adderParam = method.param(genericT, adderParameterString);
method.body().invoke(JExpr.ref(listFieldString), "add").arg(adderParam);
```

Generated method:

```java 
public void addGeneric(T toAddParam) {
    genericList.add(toAddParam);
}
```

Here’s all the relevant code (javadoc generation included) and the generated class:

```java 
String concreteName = "net.cardosi.MyNewGenericClass";
String genericTypeName = "T";
JClass genericT = codeModel.ref(genericTypeName);
JDefinedClass concreteClass = codeModel._class(concreteName);
concreteClass.generify(genericTypeName);
JDocComment javadoc = concreteClass.javadoc();
String commentString = "My wonderful Generic class";
javadoc.append(commentString);
JClass rawLLclazz = codeModel.ref(List.class);
JClass fieldClazz = rawLLclazz.narrow(genericT);
JClass rawArrayListClass = codeModel.ref(ArrayList.class);
JClass arrayListFieldClazz = rawArrayListClass.narrow(genericT);
JExpression newInstance = JExpr._new(arrayListFieldClazz)
                            .arg(JExpr.lit(0));
String listFieldString = "genericList";
JFieldVar listFieldVar = concreteClass.field(JMod.PRIVATE, 
                                             fieldClazz, 
                                             listFieldString);
listFieldVar.init(newInstance);
javadoc = listFieldVar.javadoc();
commentString = "My Generic List<T>";
javadoc.append(commentString);
String methodName = "addGeneric";
JMethod method = concreteClass.method(JMod.PUBLIC, 
                                      void.class, 
                                      methodName);
javadoc = method.javadoc();
commentString = "Method to add a Generic T to our List<T>";
javadoc.append(commentString);
String adderParameterString = "toAddParam";
JCommentPart commentPart = javadoc.addParam(adderParameterString);
JVar adderParam = method.param(genericT, adderParameterString);
commentString = "The T to add.";
commentPart.append(commentString);
method.body().invoke(JExpr.ref(listFieldString), "add").arg(adderParam);
codeModel.build(new File("."));
```	

```java	
package net.cardosi;

import java.util.ArrayList;
import java.util.List;

/**
 * My wonderful Generic class
 * 
 */
public class MyNewGenericClass{

    /**
     * My Generic List<T>
     * 
     */
    private List genericList = new ArrayList(0);

    /**
     * Method to add a Generic T to our List<T>
     * 
     * @param toAddParam
     *     The T to add.
     */
    public void addGeneric(T toAddParam) {
        genericList.add(toAddParam);
    }

}
```

