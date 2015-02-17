# Java code generation with CodeModel – Inheritance

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
