
## Functional Domain Models and Event Sourcing ##

Recently I gave a talk for the first time on Functional Domain Models.
There have been many questions in the Domain Driven Design community
about whether you could create a domain model without using an Object
Oriented Language. For the many of you who have been using Event
Sourcing, you already have been building Hybrid Functional Domain
Models but could make them purely functional in a few easy steps in
most cases.

As is usually the case I prefer to express such concepts in code as
opposed to in prose and will illustrate the transformations in code.
Let’s start off with a simple method in a domain object taken from the
Simple.CQRS example.

```java
public void deactivate() {
    if(!_activated) {
        throw new InvalidOperationException(“already deactivated”);
    }
    _activated =true;
}
```

The first refactor we will do is to break this method apart into two
methods. The first method being a public behaviour that decides
whether the operation is allowed and the second method simply
transitioning the state of the object (no beahviours, cannot throw
exceptions etc).

```java
public void deactivate() {
    if(!_activated) {
        throw new InvalidOperationException(“already deactivated”);
    }
    doDeactivate();
}

private void doDeactivate() {
    _activated = false;
}
```

This is a refactor I quite often apply in domain models regardless of
if I am using Event Sourcing or not. I like to separate the
behavioural code from the code that actually does the state
transition.

To move this code to be Event Sourced. We would refactor parameters to
object of the method and rename it to an event (generally we also give
a common name by convention to the method so that it can easily be
dispatched to).

```java
public void deactivate() {
    if(!_activated) {
        throw new InvalidOperationException(“already deactivated”);
    }
    doDeactivate(new InventoryItemDeactivated(this.id);
}

private void aoDeactivate(InventoryItemDeactivated e) {
    _activated = false;
}
```

These are the two common methods that get used in an Event Sourced
system. The public behaviour decides whether the Apply method should
be called (doing any behaviours involved) and only the apply method is
only allowed to mutate its state. It can change state but it cannot
say charge your credit card because doing so would result in
interesting behaviours when replaying events (by calling the Apply
Method) to get the object back to its current state. Getting to this
state has also brought us to a Hybrid Functional Domain Model.
In order to get my object back to its state I will replay the events
that I have saved for the object. If an InventoryItem had for instance
been Created, Audited, and Deactivated the equivalent would be.

```java
InventoryItem.apply(new Created());
InventoryItem.apply(new Audited());
InventoryItem.apply(new Deactivated());
```

If I were just to make the Apply method return this at the end I could
also chain these methods resulting in.

```java
item = InventoryItem.apply(new Created())
                    .apply(new Audited())
                    .apply(new Deactivated())
```

Said differently:    
*Current State is a Left Fold of previous behaviours.*

There is still some magic happening in the code however that the
compiler does for you.  When you call  InventoryItem
`InventoryItem::Apply(Created)` the method signature that is actually
used is `InventoryItem::Apply(InventoryItem, Created)`. The first
parameter is implicitly created by the compiler and named `this`. What
would happen if `this` were made explicit?

```java
public void Deactivate(InventoryItem item) {
    if(!_activated) {
        throw new InvalidOperationException(“already deactivated”);
    }
    doDeactivate(new InventoryItemDeactivated(item.id);
}

private InventoryItem deactivated(InventoryItem item, InventoryItemDeactivated e) {
    boolean activated = false;
    return new InventoryItem(item, activated);
}
```

There is no more object! Just some data. Also note that in the process
I made the data immutable (which tends to be a good thing). If we were
to write our chain now we would end up with.

```java
Item = deactivated(audited(created(null, new Created(…)), 
                           new Audited(…),
                           new Deactivated(…))
```

The functions themselves can now also be generalized as *State -> Event -> State*. 
In fact all stateful projections can be defined this way. If
you look in the EventStore’s JavaScript query language you will notice
all are defined as `function(state, event)` returning `state`. 
At the end of this chain, we have our current state. Thus to do a behaviour we have:

```java
deactivate(deactivated(dudited(created(null, new Created(…)), 
                               new Audited(…), 
                               new Deactivated(…)))
```

If you are a haskell/clojure/scala/etc developer the next bits of code
should be completely comfortable for you, for C# developer the next
bits of code should seem somewhat familiar to you, and if you are a
Java developer, well I hear Scala is a nice language :)

The above pattern is a well known concept represented by a higher order
function. Let’s try it that way.

```java
State currentState = events.foldLeft(0)(state, event) -> 
    event match {
        case d: inventoryItemDeactivated -> deactivated(d.id, d.reason)
        case c: inventoryItemCreated     -> created(c.id, c.name)
        case a: inventoryItemAudited     -> audited(a.id, a.outcome, a.endValue)
    };
//extractors can be used here as well
//a snapshot is just a memoization of the foldLeft operation at a given point.
```

The whole idea here is that each Event now represents a function.
We can determine what function that should be when we replay, this code
maps a previous event that has been saved to whatever function we
currently want to use to represent what it means to us.

This is not to say that everyone should immediately give up on writing
object oriented domain models. This is to show that many today are
already writing non-object oriented domain models and have not even
realized it. *An Event Store is a functional database*.
