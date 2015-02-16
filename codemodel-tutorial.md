# Java code generation with CodeModel – Basics

Source: http://blogtech.cardosi.net/2013/03/19/tutorial-java-codemodel-basics/

This part of the tutorial is about basic Java code generation using CodeModel.

My current job is all about ETL – i.e. I have to transform flat formatted data to xml 
and back again… exciting, uh?

Anyway, my first step was to write classes to reflect the different type of “records” 
that could be part of the flat file, but the “specification” for the structure keep 
changing (sounds familiar ?). Being absolutely lazy I realized that I needed something 
to generate that code for me!

Surely there are many other frameworks I am not aware of, maybe even better, but I found 
out CodeModel and decided to give it a try, also because it is used (beyond the scene) 
by JAXB – the library I am using to do xml-related stuff.

As for other stuff from glassfish site, I think its pretty good, but the documentation 
could have been more developer-friendly.

So, I write down this little tutorial for other people that want to use this nice library.

For this guide I will use codemodel-2.4.jar, that could be downloaded from here 
http://download.java.net/maven/2/com/sun/codemodel/codemodel/2.4/.

The very first steps are:

1) instantiate the root of the code DOM;
2) use it to create the class;

This is done in a couple of lines:

```java
JCodeModel codeModel = new JCodeModel();
JDefinedClass definedClass = codeModel._class("ClassName");
```

Keep note that the name of the defined class should be the fully qualified name 
(i.e. with the package declaration).

The actual “java” file will be written calling the method

```java
JCodeModel.build(File destDir);
```

The bare minimum code needed to generate an (absolutely empty) “java” file looks like this:

```java
JCodeModel codeModel = new JCodeModel();
try {
    JDefinedClass definedClass = codeModel._class("net.cardosi.MyNewClass");
    codeModel.build(new File("."));
} catch (JClassAlreadyExistsException e) {
   // ...
} catch (IOException e) {
   // ...
}
```

and here’s the generated file that you will found in the net/cardosi/ folder:

```java
package net.cardosi;

public class MyNewClass {

}
```

Surely we would like to add some field to this class, and here’s the needed method 
provided by JDefinedClass:

```java
JDefinedClass.field(int mods, Class<?> type, String name)
```

The first parameter define the accessor modificator (Public, Private, etc.), 
the second one is to tell the Class of the variable to be generated, and 
the third one is for the field name.

Let’s add an int called `intVar` to our class:

```java
JCodeModel codeModel = new JCodeModel();
try {
    JDefinedClass definedClass = codeModel._class("net.cardosi.MyNewClass");
    codeModel.build(new File("."));
    JFieldVar field = definedClass.field(JMod.PRIVATE, 
                                         int.class, 
                                         "intVar");
} catch (JClassAlreadyExistsException e) {
    // ...
} catch (IOException e) {
    // ...
}
```

and let’s see the result:

```java
package net.cardosi;

public class MyNewClass {
    private int intVar;
}
```

of course it is possible to initialize the field with a *default* value:

```java
JFieldVar.init(JExpression init);
```

and it is possible to create getter and setter for it

```java
// Create the getter method and return the JFieldVar previously defined
JMethod getterMethod = definedClass.method(int mods, 
                                           Class<?> type, 
                                           String name);
JBlock block = getterMethod.body();
block._return(field);

// Create the setter method and set the JFieldVar previously defined 
// with the given parameter
JMethod setterMethod = definedClass.method(int mods, 
                                           Class<?> type, 
                                           String name);
setterMethod.param(int.class, "intParam");		
setterMethod.body().assign(JExpr._this().ref("intVar"), 
                           JExpr.ref("intParam"));
```

While I found most the code-related stuff we have seen until now pretty intuitive 
to understand (JDefinedClass, JFieldVar, JMethod and JBlock), I do not mind to tell you 
I had my problems with JExpression and JExpr, and surely we will give them more attention.

But now let’s review the code we have written until now

```
JCodeModel codeModel = new JCodeModel();
try {
     String className = "net.cardosi.MyNewClass";
     JDefinedClass definedClass = codeModel._class(className);
     String fieldName = "intVar";
     String fieldNameWithFirstLetterToUpperCase = "IntVar";
     JFieldVar field = definedClass.field(JMod.PRIVATE, 
                                          int.class, 
                                          fieldName);
     String getterMethodName = "get" + fieldNameWithFirstLetterToUpperCase;
     JMethod getterMethod = definedClass.method(JMod.PUBLIC,
                                                int.class,
                                                getterMethodName);
     JBlock block = getterMethod.body();
     block._return(field);
     String setterMethodName = "set" + fieldNameWithFirstLetterToUpperCase;
     JMethod setterMethod = definedClass.method(JMod.PUBLIC,
                                                Void.TYPE,
                                                setterMethodName);
     String setterParameter = "intVarParam";
     setterMethod.param(int.class, setterParameter);
    setterMethod.body()
                .assign(JExpr._this().ref(fieldName), 
                        JExpr.ref(setterParameter));
     codeModel.build(new File("."));
} catch (JClassAlreadyExistsException e) {
   // ...
} catch (IOException e) {
   // ...
}
```

And here’s the generated class

```
package net.cardosi;

public class MyNewClass {

    private int intVar;

    public int getIntVar() {
        return intVar;
    }

    public void setIntVar(int intVarParam) {
        this.intVar = intVarParam;
    }

}
```

# Java code generation with CodeModel – Javadoc

This part of the tutorial is about Javadoc generation with CodeModel.

What is Java code without Javadoc? Surely none of you will ever think to write a class or 
a method without a properly formatted and meaningful documentation… or not ?

Well, of course CodeModel does not leave you alone in this task; let’s see how.
Here’s the source of JDocCommentable interface:

```java
package com.sun.codemodel;

/**
 * Program elements that can have Javadoc
 * 
 * @author Jonas von Malottki
 */
public interface JDocCommentable {
    /**
     * @return the JavaDoc of the Element
     */
    JDocComment javadoc();
}
```

As the smartest of you may have already guessed, JDocComment returned by javadoc() method 
is the class to use to create javadocs. Note that this interface is implemented by all 
the classes that generate *javadoc-able* elements (classes, enums, variables, methods, packages).
To write something in the created comment we use the method JDocComment.append(Object).

So, the code needed to comment a class declaration is something like this:

```java
String className = "net.cardosi.MyNewClass";
JDefinedClass definedClass = codeModel._class(className);
JDocComment comment = definedClass.javadoc();
String commentString = "My wonderful class";
comment.append(commentString);
```

and here’s the generated code, in all its splendor:

```java
package net.cardosi;

/**
 * My wonderful class
 * 
 */
public class MyNewClass {
   ...
}
```

The same goes for variables:

```java
String fieldName = "intVar";
String fieldNameWithFirstLetterToUpperCase = "IntVar";
JFieldVar field = definedClass.field(JMod.PRIVATE, int.class, fieldName);
JDocComment comment = field.javadoc();
String commentString = "My int variable";
comment.append(commentString);
```

But, ehy! What about parameters, returned values and stuff like that?
No worry, guys. JDocComment has different methods to retrieve a JCommentPart 
specifically suited for each one.

For example, this is to write a simple setter method documentation, 
with a single parameter description:

```java
JMethod setterMethod = definedClass.method(JMod.PUBLIC, 
                                           Void.TYPE, 
                                           setterMethodName);
JDocComment comment = setterMethod.javadoc();
String commentString = "Method to set " + fieldName + "";
comment.append(commentString);
String setterParameter = "intVarParam";
JCommentPart commentPart = javadoc.addParam(setterParameter);
commentString = "The " + fieldName + " to set.";
commentPart.append(commentString);
```

For return descriptions we have just to retrieve `JCommentPart` 
with `JDocComment.addReturn()`:

```java
JMethod getterMethod = definedClass.method(JMod.PUBLIC, 
                                           int.class, 
                                           getterMethodName);
JDocComment comment = getterMethod.javadoc();
String commentString = "Method to get " + fieldName + "";
comment.append(commentString);
JCommentPart commentPart = javadoc.addReturn();
commentString = "The " + fieldName + ".";
commentPart.append(commentString);
```

`JDocComment` has also methods to retrieve JCommentPart to generate Deprecated, Throws, 
and XDoclet parts, but they works the same way, so I won’t describe them here.

Now, if you followed the tutorial from the beginning, you should have something like
this (well, I know it’s getting ugly…)

```java
JCodeModel codeModel = new JCodeModel();
try {
    String className = "net.cardosi.MyNewClass";
    JDefinedClass definedClass = codeModel._class(className);
    JDocComment comment = definedClass.javadoc();
    String commentString = "My wonderful class";
    comment.append(commentString);
    String fieldName = "intVar";
    String fieldNameWithFirstLetterToUpperCase = "IntVar";
    JFieldVar field = definedClass.field(JMod.PRIVATE, 
                                         int.class,
                                         fieldName);
    comment = field.javadoc();
    commentString = "My int variable";
    comment.append(commentString);
    String getterMethodName = "get" + fieldNameWithFirstLetterToUpperCase;
    JMethod getterMethod = definedClass.method(JMod.PUBLIC, 
                                               int.class,
		                                       getterMethodName);
    comment = getterMethod.javadoc();
    commentString = "Method to get " + fieldName + "";
    comment.append(commentString);
    JCommentPart commentPart = javadoc.addReturn();
    commentString = "The " + fieldName + ".";
    commentPart.append(commentString);
    JBlock block = getterMethod.body();
    block._return(field);
    String setterMethodName = "set" + fieldNameWithFirstLetterToUpperCase;
    JMethod setterMethod = definedClass.method(JMod.PUBLIC, 
                                               Void.TYPE,
				                               setterMethodName);
    comment = setterMethod.javadoc();
    commentString = "Method to set " + fieldName + "";
    comment.append(commentString);
    String setterParameter = "intVarParam";
    commentPart = javadoc.addParam(setterParameter);
    commentString = "The " + fieldName + " to set.";
    commentPart.append(commentString);
    setterMethod.param(int.class, setterParameter);
    setterMethod.body().assign(JExpr._this().ref(fieldName),
			                   JExpr.ref(setterParameter));
    codeModel.build(new File("."));
} catch (JClassAlreadyExistsException e) {
	// ...
} catch (IOException e) {
	// ...
}
```

and here’s the commented class:

```java
package net.cardosi;

/**
 * My wonderful class
 * 
 */
public class MyNewClass {

    /**
     * My int variable
     * 
     */
    private int intVar;

    /**
     * Method to get intVar
     * 
     * @return
     *     The intVar.
     */
    public int getIntVar() {
        return intVar;
    }

    /**
     * Method to set intVar
     * 
     * @param intVarParam
     *     The intVar to set.
     */
    public void setIntVar(int intVarParam) {
        this.intVar = intVarParam;
    }

}
```


# Java code generation with CodeModel – JExpression

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
JExpression newInstance = JExpr._new(arrayListFieldClazz).arg(JExpr.lit(0));
String listFieldString = "genericList";
JFieldVar listFieldVar = concreteClass.field(JMod.PRIVATE, fieldClazz, listFieldString);
listFieldVar.init(newInstance);
javadoc = listFieldVar.javadoc();
commentString = "My Generic List<T>";
javadoc.append(commentString);
String methodName = "addGeneric";
JMethod method = concreteClass.method(JMod.PUBLIC, void.class, methodName);
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


#Java code generation with CodeModel – Inheritance

This part of the tutorial is about Inheritance management with CodeModel.

Now that we have covered basics, javadoc, jexpressions and generics, let’s take a look 
at how to manage inheritance with CodeModel.

First things first, let’s build an Interface. Actually, is not very different from building 
a normal Class: we simply have to pass a specific parameter ClassType.INTERFACE to the usual method. 
As a plus, let’s add some methods and javadocs:

```java  
String interfaceName = "net.cardosi.MyNewInterface";
JDefinedClass definedInterface = codeModel._class(interfaceName, 
                                                  ClassType.INTERFACE);
JDocComment javadoc = definedInterface.javadoc();
String commentString = "My wonderful Interface";
javadoc.append(commentString);
String methodName = "toBeImplemented";
JMethod method = definedInterface.method(JMod.PUBLIC, 
                                         void.class, 
                                         methodName);
javadoc = method.javadoc();
commentString = "Method to be implemented";
javadoc.append(commentString);
codeModel.build(new File("."));
```

And here’s the result:

```java
package net.cardosi;

/**
 * My wonderful Interface
 * 
 */
public interface MyNewInterface {

    /**
     * Method to be implemented
     * 
     */
    public void toBeImplemented();

}
```

Now, we want to create an Abstract class that implements the interface just created. 
In this case, though, we have to use another different form of the usual method:

```java
codeModel._class(int mods, String name, ClassType classTypeVal);
```

“Where this ‘mods’ come from?” the most curios of you will surely asks themselves. 
Well, they are defined inside com.sun.codemodel.JMod and the value actually given 
must be the sum of all the JMods you want to define. So, if you want to generate 
an Abstract Protected class, you should use

```java
int mods = JMod.Abstract + JMod.PROTECTED;
```

Back to our example, we use only the JMod.Abstract modifier, because we want the default access level:

```java
String abstractName = "net.cardosi.MyNewAbstract";
int mods = JMod.ABSTRACT;
JDefinedClass abstractClass = codeModel._class(mods, 
                                               abstractName,
				                               ClassType.CLASS)
				                       ._implements(definedInterface);
JDocComment comment = abstractClass.javadoc();
commentString = "The <code>" + abstractName + "</code> " + 
                "implementing the "  + interfaceName + "interface";
comment.append(commentString);
codeModel.build(new File("."));
```

And here’s the generated Abstract class

```java
package net.cardosi;

/**
 * The < c o d e >net.cardosi.MyNewAbstract< / c o d e > implementing 
 * the < c o d  e>net.cardosi.MyNewInterface< / c o d e> interface
 * 
 */
abstract class MyNewAbstract implements MyNewInterface {

}
```

Now, last but not least, we want to build a new Class that extends the Abstract one 
just created and, since we are getting greedy, also implements another Interface, 
like Iterator<T>. Remember that we have to implement all the methods from MyNewInterface 
and Iterator<T> and that we have to work with Generic as in the previous part:

```java
String concreteName = "net.cardosi.MyNewConcrete";
String genericTypeName = "T";
JClass genericT = codeModel.ref(genericTypeName);
JClass rawInterface = codeModel.ref(Iterator.class);
JClass genericInterface = rawInterface.narrow(genericT);
JDefinedClass concreteClass = codeModel._class(concreteName)
			                           ._implements(genericInterface)
			                           ._extends(abstractClass);
concreteClass.generify(genericTypeName);
javadoc = concreteClass.javadoc();
commentString = "My concrete implementation";
javadoc.append(commentString);
method = concreteClass.method(JMod.PUBLIC, void.class, methodName);
javadoc = method.javadoc();
commentString = "Method implementation";
javadoc.append(commentString);
method.body().directStatement("System.out.println(\"Hello World!\");");
method = concreteClass.method(JMod.PUBLIC, boolean.class, "hasNext");
method.body().directStatement("// TODO To be implemented");
method.body()._return(JExpr.TRUE);
javadoc = method.javadoc();
commentString = "Method implementation";
javadoc.append(commentString);
method = concreteClass.method(JMod.PUBLIC, genericT, "next");
method.body().directStatement("// TODO To be implemented");
method.body()._return(JExpr._null());
javadoc = method.javadoc();
commentString = "Method implementation";
javadoc.append(commentString);
method = concreteClass.method(JMod.PUBLIC, void.class, "remove");
method.body().directStatement("// TODO To be implemented");
javadoc = method.javadoc();
commentString = "Method implementation";
javadoc.append(commentString);
codeModel.build(new File("."));
```
        
And here’s the final result:

```java
package net.cardosi;

import java.util.Iterator;

/**
 * My concrete implementation
 * 
 */
public class MyNewConcrete extends MyNewAbstract implements Iterator {

	/**
	 * Method implementation
	 * 
	 */
	public void toBeImplemented() {
		System.out.println("Hello World!");
	}

	/**
	 * Method implementation
	 * 
	 */
	public boolean hasNext() {
		// TODO To be implemented
		return true;
	}

	/**
	 * Method implementation
	 * 
	 */
	public T next() {
		// TODO To be implemented
		return null;
	}

	/**
	 * Method implementation
	 * 
	 */
	public void remove() {
		// TODO To be implemented
	}
}
```

And that’s all for now. See you soon!!!








