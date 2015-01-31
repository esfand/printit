
# Command Handlers and the Domain Model #

Many people are talking about Command Handlers and how they work when 
they are really just CRUD handling very simple operations. 
Likely there is little or no validation in them and they are simply 
passing through information.

A perfect example of this might be the name of a customer in a CRM. 
There are no invariants of the customer object that need the name 
(there could be invariants imagined such as all customers who have 
a name of ‘greg’ get a 15% discount when buying things but let’s 
imagine for a minute that such invariants do not exist in this case).

Many people have been suggesting to just use the command handler as 
a pass-through to then publish the event (transaction script like). 
The code would look something like this (simplified).

```java
class ChangeCustomerName extends Handles<ChangeCustomerNameCommand> {

    public void handle(ChangeCustomerNameCommand cmd) {
        if(somebasic logic) {
            throw new Exception();
        }
        DomainEvents.publish(new CustomerNameChangedEvent(cmd.firstName, cmd.lastName));
    }
}
```

A more stereotypical version of this might look something like.

```
class ChangeCustomerName extends Handles<ChangeCustomerNameCommand> {

    private CustomerRepository repository;

    public ChangeCustomerName(CustomerRepository repository) {
        this.repository = repository;
    }

    public void handle(ChangeCustomerNameCommand cmd) {
        Customer customer = repository.fetchById(cmd.id);
        customer.changeNameTo(cmd.firstName, cmd.lastName);
    }
}
```

with customer then creating the event.

There is nothing wrong with either of these solutions, both have their merit. 
Unfortunately the Ubiquitous Language becomes extremely interesting in the first case 
as there is no concept of a Customer in the domain model. 
There is a concept that the customer’s name can be changed and that when it is 
there is a CustomerNameChangedEvent but there is no explicit concept of what a 
customer is within the domain model.
            
This is perfectly ok if DDD is not being applied (and there are many places where doing this style of transaction script command handler can be very useful). If DDD is being applied though this is probably not a good pattern to be following. Very little code is being saved (the same new of the event is just being put on the customer object instead of in the command handler) and a concept within the domain is being lost. This becomes especially true if the other object already exists within the domain model as there are other behaviors associated with it. 
            
If it is the case that there are just a few operations on an Aggregate and they don’t access current state for invariant protection the domain object will probably only contain an aggregate id and the basic if statements that are present in the command handlers otherwise. These thin little objects still have benefit though as they are defining and making explicit the aggregate boundary as well as giving vocabulary to what the aggregate is and the behaviors contained within its boundary.
            
It is also however a great example of how Domain Driven Design is not a pre-requisite to using CQRS and even events. For a great many systems the first command handler will be a much better choice than the second command handler (especially for line of business or simple web systems). It can also be a good idea to use a hybrid approach where although many things exist in a given context most of them are simple and not specifically modeled and the domain model itself is focused only on those cases where a domain model makes sense.
