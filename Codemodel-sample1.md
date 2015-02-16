# Java CodeModel – Java Code Generation Library

## What Is CodeModel ?

CodeModel is a Java library for code generators; it provides a way to generate Java programs.Now a days the library used in many areas, like IDE and FreeMarkers.This is the simplest method to create Java class from any data model or from files.
Most of the template Enginnes work on this librarys and its capable of developing classes without bug.

## How to Use CodeModel ?

firsly download or add depentency to your project

```xml
<dependency>
    <groupId>com.sun.codemodel</groupId>
    <artifactId>codemodel</artifactId>
    <version>2.6</version>
</dependency>
```
            
Then create first class like this,

```java
JCodeModel codeModel = new JCodeModel();
JDefinedClass sample= codeModel._class( "org.letmeshare.sample.Sample" );
```

This will create a class structure like normal Sample.java,
If your class impliments or Extends some class or Interface then class ,

    sample._implements(Class<?> iface);//it impliments interface

or

    sample._extends(JClass superClass)//it exends superClass

## How to create Methods, in our Sample class.

```java
JMethod method = sample.method( JMod.PUBLIC, String.class, "doPrint" );
        method.body().directStatement("System.out.println(\"\");");
        method.body()._return(JExpr.direct("new String(\"return my name\")"));
```
        
Here the we can create a function with name doPrint, which return a String and having a public scope.

We can add statements intomthe method via the method.body().addStatement or with method.body().directStatement or assignStatement.

## What will be the OutPut ?

To generate output – the java class named Sample.java call the build method

```java
sample.build(new File("D:\\output"));
```

Your Java Class Look like this…

```java
package org.letmeshare.sample;

public class Sample {

    public String doPrint() {
        System.out.println("");
        return (new String("return my name"));
    }
}
```

Ok, lets try an example how to generate a sample java file.

```java
public static void main(String[] args) throws JClassAlreadyExistsException, IOException {
        JCodeModel jcm = new JCodeModel();
        JDefinedClass sample = jcm._class("org.letmeshare.sample.Sample");
        //Declare doPrint method
        JMethod doprint = sample.method(JMod.PUBLIC, String.class, "doPrint");
        //Declare a String variable in doPrint method
        JVar myNameVariable = doprint.body().decl(jcm._ref(String.class), "myName");
         //Declare doScan method
        JMethod doScan = sample.method(JMod.PUBLIC, String.class, "doScan");
        //Declare parameter for doScan method
        JVar param = doScan.param(String.class, "myName");
        // assign value to the declared variable with return of doScan with argument String -'Letmeshare'
        doprint.body().assign(myNameVariable, doprint.body().invoke(doScan).arg("Letmeshare"));
        //print my variable
        doprint.body().directStatement("System.out.println(" + myNameVariable.name() + ");");
        // return doScan function with string with param 'myName'
        doScan.body()._return(JExpr.direct("new String(\"return my name\"+myName)"));
        //tell whre to write this code
        jcm.build(new File("D:\\OUTGWT"));
    }
```
    
Your output look like..

```java
package org.letmeshare.sample;

public class Sample {

    public String doPrint() {
        String myName;
        doScan("Letmeshare");
        myName = doScan("Letmeshare");
        System.out.println(myName);
    }
 
    public String doScan(String myName) {
        return (new String("return my name"+myName));
    }
}
```
