

## A better domain events pattern ##

[Domain events](http://www.udidahan.com/2009/06/14/domain-events-salvation/) are one of the final patterns needed to create a [fully encapsulated domain model](http://lostechies.com/jimmybogard/2010/02/04/strengthening-your-domain-a-primer/) – one that fully enforces a consistency boundary and invariants. The need for domain events comes from a desire to inject services into domain models. What we really want is to create an encapsulated domain model, but decouple ourselves from potential side effects and isolate those explicitly. The original example I gave looked something like this:

```java
public Payment recordPayment(BigDecimal paymentAmount, IBalanceCalculator balanceCalculator) {
      
    Payment payment = new Payment(paymentAmount, this);

    _payments.add(payment);

    balance = balanceCalculator.calculate(this);
    if (balance == 0) {
          DomainEvents.raise(new FeePaidOff(this));
    }
    return payment;
}
```      

There’s one small problem with our domain event implementation. Because the domain events class is static, it also dispatches to handlers immediately. This makes testing our domain model difficult, because side effects, however crazy, are executed immediately at the point of raising the domain event.

Typically, I want the side effects of a domain event to occur within the same logical transaction, but not necessarily in the same scope of raising the domain event. If I cared enough to have the side effects occur, I would instead just couple myself directly to that other service as an argument to my domain’s method.

Instead of dispatching to a domain event handler immediately, what if instead we recorded our domain events, and before committing our transaction, dispatch those domain events at that point? This will have a number of benefits, besides us not tearing our hair out. Instead of raising domain events, let’s define a container for events on our domain object:</p>

```java
public interface IEntity {

    Collection<IDomainEvent> getEvents();
}
```      

In our method that raises the domain event, instead of dispatching immediately, we simply record a domain event on our entity:

```java 
public Payment recordPayment(BigDecimal paymentAmount, IBalanceCalculator balanceCalculator) {

    Payment payment = new Payment(paymentAmount, this);

    _payments.add(payment);

    balance = balanceCalculator.calculate(this);
    if (balance == 0) {
      Events.add(new FeePaidOff(this));
    }
    return payment;
}
```

This makes testing quite a bit simpler since we don’t this global domain event dispatcher firing things off, we can just assert on our self-contained entity class:

```java
@Test
public void should_notify_when_the_balance_is_paid_off() {

      Fee paidOffFee = null;

      Customer customer = new Customer();

      Fee fee = customer.chargeFee(100m);

      fee.recordPayment(100m, new BalanceCalculator());

      FeePaidOff paidOffEvent = fee.Events.OfType<FeePaidOff>().singleOrDefault();

      paidOffEvent.shouldNotBeNull();
      paidOffEvent.getFee().shouldEqual(fee);
}
```

Finally, we need to actually fire off these domain events. This is something we can hook into our infrastructure of whatever ORM we’re using. In EF, we can hook into the SaveChanges method:


```java
@Override
public int saveChanges() {

    var domainEventEntities = ChangeTracker.Entries<IEntity>()
      .map(po -> po.entity)
      .filter(po -> po.getEvents().any())
      .toArray();

      foreach (IEntity entity: domainEventEntities) {
          Collection<IEntity> events = entity.getEvents().toArray();
          entity.getEvents().clear();
          foreach (IDomainEvent domainEvent: events) {
              _dispatcher.dispatch(domainEvent);
          }
      }

      return base.saveChanges();
}
```

Just before we commit our transaction, we dispatch our events to their respective handlers. With this approach, we decouple the raising of a domain event from dispatching to a handler, which is much more obvious to me as a developer. If you want immediate side effects with other entities, just pass those in as arguments.

In dispatching our domain events, we have some flexibility now. We can dispatch synchronously, as we did in our EF example, or asynchronously by persisting our events as JSON and having an offline process pick up those undispatched events and dispatch them out-of-band. All of this is decoupled from our domain event raiser and handler.

If there’s one lesson to be learned here, it’s to beware of static, opaque dependencies, even it is a really cool concept. Separating the two concerns of raising versus dispatching keeps our domain model fully encapsulated without introducing land mines in our model.
  
