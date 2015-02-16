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
	
Here the result:

```java
package net.cardosi;

public class MyNewClassE {

    private boolean booleanVar;

    public MyNewClassE(boolean booleanVarParameter) {
        this.booleanVar = (!booleanVarParameter);
    }
}
```java

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
JFieldVar intField = (JFieldVar) definedClass.field(JMod.PRIVATE, int.class, intName).init(JExpr.lit(14));
Since JExpr.lit() is overloaded for different kind of literals (boolean, double, float, char, int, long, and String), the corresponding type of variables can be initialized that way.

Maybe I was a little bit pedantic in this section of the tutorial, but in the same time I have the impression to have just scratched the surface of JExpr and JExpression.
Anyway, I hope it will be enough for you as starting point.

Next part will deal with Generics: don’t miss it!!!
