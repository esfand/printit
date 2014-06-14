## Heterogeneous container to store genericly typed objects in Java ##

I am trying to follow Joshua Bloch's typesafe hetereogeneous container pattern from 
Effective Java to create a container of objects (MyGeneric<T>) with Class<T> as a key.

```java
public class MyClass {

    private Map<Class<?>, MyGeneric<?>> myContainer =
        new HashMap<Class<?>, MyGeneric<?>>();

    public <T> void addToContainer(Class<T> class, MyGeneric<T> thing) {
        myContainer.put(class, thing);
    }

    public <T> MyGeneric<T> getFromContainer(Class<T> class) {
        return (MyGeneric<T>)(myContainer.get(klass));
    }
}
```

The problem is in getFromContainer I have to perform a unchecked cast. 
In Josh Bloch's container, he performs a safe cast - but in my case 
I can't see a way how this is possible.

Does any one have any ideas?

## Answer ##

In Bloch's version, `Class.cast()` is used - which is implemented as `return (T) obj`, an unchecked cast. 
It's cheating in the sense that the compiler warning about unchecked cast is moved to a precompiled lib. 
The type safety of the cast is not guarded by compiler, but by app logic.

You shouldn't worry about unchecked cast either. There are type relations that cannot be expressed in the language, 
but which programmers know to be true. So just overrule the compiler, tell it the cast is safe.

**Correction**

My understanding about "unchecked cast" was incorrect.

`Class.cast()` does not contain **unchecked cast**. 
The cast is done after "checking", if the cast is reached at runtime, it's guaranteed to succeed.

```java
T cast(Object obj)
    if obj is instance of this class   // check
        return (T)obj;                 // cast 
    else
        throw new ClassCastException
```


## When inserting objects into a type-safe heterogeneous container, why do we need the class reference? ##

I'm checking out the heterogeneous container pattern from Bloch's Effective Java and 
I'm trying to determine why the class reference is needed when inserting objects into 
the heterogeneous container. Can't I use instance.getClass() to get this reference? 
Isn't JPA's entity manager an example of this?

```java
interface BlochsHeterogeneousContainer {

    <T> void put(Class<T> clazz, T instance);

    <T> T get(Class<T> clazz);
}
```

```java
interface AlternativeHeterogeneousContainer {

    // Class<T> not needed because we can use instance.getClass()
    <T> void put(T instance);

    <T> T get(Class<T> clazz);
}
```

## Answer ##

No you can't do that, as it won't give you 
class of reference type in case of inheritance, rather than 
the class of actual object type.

Consider this example:

```java
Number num = new Integer(4);
System.out.println(num.getClass());
```

this will print:

`class java.lang.Integer` and not `java.lang.Number`.

+1 Exactly, so the interface allows for put(Number.class, num) or put(Integer.class, num) as appropriate. –  John B
  	 	
put(Integer.class, num) would throw a compiler error. 
This makes sense because the types of the arguments are (Class<Integer>, Number). 
You can't set T to Integer because Number is not a Integer. 
You can't set T to number because Class<Integer> is not a Class<Number>. 
Nonetheless, it is still useful to have a type parameter since you can do things like put(Object.class, num). 
In this case, T is set to Object (and all Numbers are Objects so it works). –  goparkyourcar 



## A type-safe heterogeneous container for parameters that validates the parameter ##

Following the example of typesafe heterogeneous containers in "Effective Java Second Edition" by Joshua Bloch, 
here is a pattern for building such a container with validator support.

The requirement: a typesafe, generic container that manages values of any type, and supports validation of the parameter. 
The validation must be specifiable by the client. First define an interface for a Parameter:

```java
public interface Parameter {

  /**
   * Gets the name that was specified for this parameter. This is an arbitrary name.
   *
   * @return the name that was specified for this parameter.
   */
  public String getParameterName();

  /**
   * Gets the value of this parameter.
   *
   * @return the value of this parameter.
   */
  public <T> T getParameterValue(Class<T> type);

  /**
   * Sets the value of this parameter.
   *
   * @param  type the type of this parameter
   * @param value the value of this parameter
   */
  public <T> void setParameterValue(Class<T> type,T value);

   /**
   * Return the {@code ParameterValidator} for this parameter. The
   * {@code ParameterValidator} exposes the {@code validate} method
   * which will be used by objects that require validation of parameters.
   *
   *
   * @return the parameterValidator for this parameter
   */
  public ParameterValidator getParameterValidator();
}
```

The salient features of the Parameter interface are:

* values are typed, and the type is controlled by the implementing class, under direction of the client
* the interface defines a type-safe machine that always returns the type for which it is asked, 
  and always stores the type supplied by the client, and validates any type based on a 
  ParameterValidator interface that is defined by the client

Next define the ParameterValidator interface:

```java
public interface ParameterValidator {
    public Boolean validate(Parameter parameter);
}
```

The ParameterValidator is simple enough that client can supply any validation needed. 
A good interface is simple, with few methods, as you can see this interface is one method 
more than a marker interface. Next, define the backing code for the Parameter implementation:

```java
/**
 * Provides support for the heterogeneous container {@code Parameter}.
 *
 * @author Terry J. Gardner
 */
public final class ParameterSupport implements Parameter {

  public static <T> ParameterSupport
    getParameterSupport(Class<T> type,
                        T value,
                        ParameterValidator parameterValidator) {
    ParameterSupport ps = new ParameterSupport();
    ps.setParameterValue(type,value);
    ps.setParameterValidator(parameterValidator);
    return ps;
  }

  /**
   * Gets the name that was specified for this parameter. This is an arbitrary name.
   *
   * @return the name that was specified for this parameter.
   */
  public String getParameterName() {
    return this.name;
  }

  private String name;

  /**
   * Gets the value of this parameter.
   *
   * @return the value of this parameter.
   */
  public <T> T getParameterValue(Class<T> type) {
    return type.cast(parameterMap.get(type));
  }

  /**
   * Sets the value of this parameter. If {@code type} is null, this
   * method throws a {@code NullPointerException}.
   *
   * @param type the type of this parameter
   *
   * @param value the value of this parameter
   */
  public <T> void setParameterValue(Class<T> type,T value) {
    if(type == null) {
      throw new NullPointerException("type cannot be null");
    }
    parameterMap.put(type,value);
  }

  private Map<Class<?>,Object> parameterMap = Util.newHashMap();

  /**
   * Return the {@code ParameterValidator} for this parameter. The
   * {@code ParameterValidator} exposes the {@code validate} method
   * which will be used by objects that require validation of parameters.
   *
   * @return the parameterValidator for this parameter
   */
  public ParameterValidator getParameterValidator() {
    return this.parameterValidator;
  }


  /**
   * Sets the {@code ParameterValidator} for this parameter. The
   * {@code ParameterValidator} exposes the {@code validate} method
   * which will be used by objects that require validation of parameters.
   *
   * @param parameterValidator the parameterValidator for this parameter
   */
  public void setParameterValidator(ParameterValidator parameterValidator) {
    this.parameterValidator = parameterValidator;
  }

  private ParameterValidator parameterValidator;
}
```

ParameterSupport implements the methods of the Parameter interface by 
using a java.util.Map to store parameter values keyed to the type of the parameter. 
It also provides a static method to return a parameterSupport object - 
note that the client provides the type, value, the ParameterValidator object.

Here is a method that validates a list of ParameterSupport parameter objects:

```java
  /**
   * Performs validation checks on the {@code parameters}. The
   * parameter checks are specific to the type of database. If the
   * parameters validate successfully, this method return {@code true},
   * otherwise the method returns {@code false}.
   * <br />
   * If {@code parameters} is null, no action is taken, no exception
   * is thrown, and this method returns {@code false}.
   *
   * @param parameters a list of parameters to validate
   */
  public Boolean validateParameters(List<Parameter> parameters) {
  
    Boolean result;
    for(Parameter parameter : parameters)
    {
      if(logger.isDebugEnabled())
      {
        logger.debug("validating parameter: " + parameter.getParameterName());
      }
      ParameterValidator pv = parameter.getParameterValidator();
      pv.validate(parameter);
    }
    return true;
  }
```

A client can use the machinery like this:

```java
    List<Parameter> parameters = Util.newList();
    ParameterSupport ldapServerHostnameParameter =
      ParameterSupport.
      getParameterSupport(String.class,
                          getLdapServerHostname(),
                          new ParameterValidator() {
        public Boolean validate(Parameter parameter) {
          Boolean result;
          String hostname = parameter.getParameterValue(String.class);
          if(hostname == null || hostname.length() == 0) {
            result = false;
          } else {
            try {
              InetAddress inetAddress = InetAddress.getByName(hostname);
              result = true;
            } catch(UnknownHostException unknownHost) {
              result = false;
            }
          }
          return result;
        }
      });

    parameters.add(ldapServerHostnameParameter);
```

The the client calls objectWithParameterListValidator.validateParameters(parameters);

The newList and newHashMap methods are static convenience methods:

```java
  /**
   * Convenience method that creates a {@code Map} object. One
   * use of this function is for saving on typing, and also to ensure
   * that typesafe Lists are created.
   *
   * @return a {@code Map} of keys K and values V
   */
  public static <K,V> Map<K,V> newHashMap() {
    return new HashMap<K,V>();
  }

  /**
   * Convenience method that creates a {@code List} object. One
   * use of this function is for saving on typing, and also to ensure
   * that typesafe Lists are created.
   *
   *
   * @return a {@code List} of T object.
   */
  public static <T> List<T> newList()
  {
    return new ArrayList<T>();
  }
```

