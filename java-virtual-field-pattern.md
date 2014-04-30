Java Virtual Field Pattern 
==============================

Source: https://kerflyn.wordpress.com/2012/07/09/java-8-now-you-have-mixins/

This pattern can be used to add services to an existing class by using 
multiple inheritance and delegation. This approach is referenced as 
virtual field pattern.

So lets start with definint our service interface.

```java
interface Switchable {    boolean isActive();
    void setActive(boolean active);
}
```

We need an interface based on Switchable with an additional abstract 
method that returns an implementation of Switchable. 
The inherited methods get a default implementation: they use the 
previous getter to transmit the call to the Switchable implementation.



```java
public interface SwitchableView extends Switchable {
    Switchable getSwitchable();
 
 
    boolean isActive() default { return getSwitchable().isActive(); }
    void setActive(boolean active) default { getSwitchable().setActive(active); }
}
```

Then, we create a complete implementation of Switchable.

```java
public class SwitchableImpl implements Switchable {
 
 
    private boolean active;
 
 
    @Override
    public boolean isActive() {
        return active;
    }
 
 
    @Override
    public void setActive(boolean active) {
        this.active = active;
    }
}
```

Here is an example where we use the virtual field pattern.

```java
public class Device {}
 
 
public class DeviceA extends Device implements SwitchableView {
    private Switchable switchable = new SwitchableImpl();
 
 
    @Override
    public Switchable getSwitchable() {
        return switchable;
    }
}
 
 
public class DeviceB extends Device implements SwitchableView {
    private Switchable switchable = new SwitchableImpl();
 
 
    @Override
    public Switchable getSwitchable() {
        return switchable;
    }
}
```

```java
DeviceA a = new DeviceA();DeviceB b = new DeviceB();
 
a.setActive(true);
 
assertThat(a.isActive()).isTrue();
assertThat(b.isActive()).isFalse();
```
