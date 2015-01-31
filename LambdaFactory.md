# Functional Factory Pattern #

Source: https://programmingideaswithjake.wordpress.com/2015/01/31/making-simple-factories-via-lambdas/

Do you want a REALLY quick way to make a Factory object? 
Then lambdas or other function-passing is what you need! 
Not only is it quick, it’s really simple. I bet, if you’re a pretty good with Lambdas, 
you have a pretty good idea on how to do this simply by having read the title. 
If you’re one of those, stick around; you never know what you could learn.

### Primer on the Factory Pattern ###

If you already know what the Factory Design Pattern is, then you can skip to the next section.

The point of the Factory pattern is to supply objects and methods with a way 
to instantiate an object without exposing all (or, often, *any*) of 
the instantiation logic (what needs to be passed into the constructor).

### Example ###

As a silly example, say there is a class, `Scientist`, that needs a way to produce 
new `Pen`s to write down his experiment data with, but he doesn’t want to be bothered 
with the creation process. To do this, you would give the `Scientist` a `PenFactory`, 
and all the `Scientist` needs to know is to push the button on the factory to get a new pen.

The `PenFactory` is a simple object with only a `create()` method that supplies a 
new instance of `Pen` whenever you call it. If the `Scientist` cared about what 
color the `Pen` was, you could supply him with a `ColoredPenFactory` whose 
`create()` method also accepts a color parameter. 
Then the `ColoredPenFactory` would have to figure out how to provide a pen with that color.

### Expanding the Factory Pattern Idea ###

The Factory Pattern is a pattern for Object-Oriented code, and is therefore limited 
to how OO works, but we can take its purpose and try to figure out a way to make it 
in a functional way, too, which actually makes it a LOT easier.

In fact, a large number of OO design patterns were created due to the lack of ability 
to pass functions around. Most of *these* can be replaced simply by passing in a function. 
A short list of them include Command, Factory, and Strategy. Many others can remove 
a lot of class hierarchy if they accept functions. Some of these include Template and Visitor.

So, the biggest difference is the idea that the Factory class doesn’t have to be a class; 
it can be a simple “callable”, too. So let’s dig into some examples.

### The OO Pen Factory ###

Just so you can see the difference between the classic OO pattern and the new function pattern, 
here are the example classes and interfaces, in OO Java.

```java
public interface Pen {
   void write(String toWrite);
   boolean outOfInk();
}

public interface PenFactory {
   Pen create();
}

public class Scientist {
	
   private PenFactory penerator;
   private Pen pen;
	
   public Scientist(PenFactory penerator) {
      this.penerator = penerator;
      this.pen = penerator.create();
   }
	
   public void writeData(String data) {
      if(pen.outOfInk()) {
         pen = penerator.create();
      }
      pen.write(data);
   }
}
```

Did you catch how I called the `PenFactory` instances `penerator`? 
I thought it was kind of silly. I hope you enjoyed it too. If not, oh well.

### Converting to a Simple Functional Pattern ###

When it comes to the Java version, you don’t actually NEED to make any changes, 
since the `PenFactory` counts as a functional interface, but, there’s no need for 
it since you can replace any instance of `PenFactory` with `Supplier<Pen>`. 
So, the `Scientist` class would look like this instead:

```java
public class Scientist {
	
   private Supplier penerator;
   private Pen pen;
	
   public Scientist(Supplier penerator) {
      this.penerator = penerator;
      this.pen = penerator.get();
   }
	
   public void writeData(String data) {
      if(pen.outOfInk()) {
         pen = penerator.get();
      }
      pen.write(data);
   }
}
```

So, to create an instance of `Scientist` with lambdas that provide 
instances of `MyPenClass`, you would type this in Java:

```java
Scientist albert = new Scientist(() -> new MyPenClass());
```

### Factories for Classes with Dependencies ###

Let’s say I wanted to make a factory for a class whose constructor requires 
the name of a brand of pens. We’ll call this class `BrandPen`. 
How would we make a factory for that? Well, writing the lambdas wouldn’t
 really be any different, really. How about we look at other ways of 
 defining callables to be passed in, though?

In Java, you can save an instance of the lambda in a variable and pass that in. 
Or you can use a method reference:

```java
Supplier bicPen = () -> new BrandPen("BiC");
Scientist thomas = new Scientist(bicPen);
// assuming that BrandPen has a static method called bicPen
Scientist nicola = new Scientist(BrandPen::bicPen);
```

### Factories with Dependencies ###

Oh man, now the `Scientist` wants to be able to specify the *color* of
the pen that the factory supplies! Well, you could give him different factories 
for each color and tell him to use each different factory to make the different pens, 
but there’s simply no room in his lab for so many `PenFactory`s! We’ll have to give 
a factory that can be told what color to use.

To do this, we’ll have to change Java’s `Supplier<Pen>` to a `Function<>Color, Pen>`. 
Obviously, you won’t need to change the type in Python, since it’s dynamic and doesn’t require type information.

But the `Scientist` classes also need to change how they use their factories. 
In Java, wherever the `Scientist` is asking for a new instance, it needs to provide a color too, like this:

```java
pen = penerator.apply(Color.RED);
```

The factory we pass into the `Scientist` in Java could look like this:

```java
Scientist erwin = new Scientist(color -> new ColoredPen(color, "BiC"));
```

### Multimethod Factories ###

In some examples for the Factory Pattern on the internet, they show factories that 
have multiple methods to call for generating an object. 
I haven’t seen this in action in real life, but it might happen. 
In these instances, it may be better to stick to the OO option, but 
if you want to change it to a functional pattern, simply provide 
separate factory callables instead of one object with multiple methods.

### Outro ###

I didn’t expect to write this much, but as I went, there were so many 
little variances that I wanted to show. I didn’t get around to them all, 
mostly because I didn’t feel like keeping track of them all, but I’m sure that 
I’ve given you a good enough toolbox to figure it out on your own.

I hope you learned something. If you didn’t, I hope you enjoyed the example at least.

 
 
