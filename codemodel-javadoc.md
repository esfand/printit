# CodeModel – Javadoc

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
