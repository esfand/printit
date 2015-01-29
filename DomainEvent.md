

    <h1 class="entry-title">A better domain events pattern</h1>

  <p><a href="http://www.udidahan.com/2009/06/14/domain-events-salvation/">Domain events</a> are one of the final patterns needed to create a <a href="http://lostechies.com/jimmybogard/2010/02/04/strengthening-your-domain-a-primer/">fully encapsulated domain model</a> – one that fully enforces a consistency boundary and invariants. The need for domain events comes from a desire to inject services into domain models. What we really want is to create an encapsulated domain model, but decouple ourselves from potential side effects and isolate those explicitly. The original example I gave looked something like this:</p>
  <p>


      &lt;pre&gt;&lt;code class="language-c# c#"&gt;public Payment RecordPayment(decimal paymentAmount, IBalanceCalculator balanceCalculator)
      {
      var payment = new Payment(paymentAmount, this);

      _payments.Add(payment);

      Balance = balanceCalculator.Calculate(this);

      if (Balance == 0)
      DomainEvents.Raise(new FeePaidOff(this));

      return payment;
      }&lt;/code&gt;&lt;/pre&gt;
      &lt;p&gt;
  </noscript>

  <p>There’s one small problem with our domain event implementation. Because the domain events class is static, it also dispatches to handlers immediately. This makes testing our domain model difficult, because side effects, however crazy, are executed immediately at the point of raising the domain event.</p>
  <p>Typically, I want the side effects of a domain event to occur within the same logical transaction, but not necessarily in the same scope of raising the domain event. If I cared enough to have the side effects occur, I would instead just couple myself directly to that other service as an argument to my domain’s method.</p>
  <p>Instead of dispatching to a domain event handler immediately, what if instead we recorded our domain events, and before committing our transaction, dispatch those domain events at that point? This will have a number of benefits, besides us not tearing our hair out. Instead of raising domain events, let’s define a container for events on our domain object:</p>
  <p>

  <noscript>
      &lt;pre&gt;&lt;code class="language-c# c#"&gt;public interface IEntity
      {
      ICollection&amp;lt;IDomainEvent&amp;gt; Events { get; }
      }&lt;/code&gt;&lt;/pre&gt;
      &lt;p&gt;
  </noscript>

  <p>In our method that raises the domain event, instead of dispatching immediately, we simply record a domain event on our entity:</p>
  <p>

  <noscript>
      &lt;pre&gt;&lt;code class="language-c# c#"&gt;public Payment RecordPayment(decimal paymentAmount, IBalanceCalculator balanceCalculator)
      {
      var payment = new Payment(paymentAmount, this);

      _payments.Add(payment);

      Balance = balanceCalculator.Calculate(this);

      if (Balance == 0)
      Events.Add(new FeePaidOff(this));

      return payment;
      }&lt;/code&gt;&lt;/pre&gt;
      &lt;p&gt;
  </noscript>

  <p>This makes testing quite a bit simpler since we don’t this global domain event dispatcher firing things off, we can just assert on our self-contained entity class:</p>
  <p>



  <noscript>
      &lt;pre&gt;&lt;code class="language-c# c#"&gt;[Test]
      public void Should_notify_when_the_balance_is_paid_off()
      {
      Fee paidOffFee = null;

      var customer = new Customer();

      var fee = customer.ChargeFee(100m);

      fee.RecordPayment(100m, new BalanceCalculator());

      var paidOffEvent = fee.Events.OfType&amp;lt;FeePaidOff&amp;gt;().SingleOrDefault();

      paidOffEvent.ShouldNotBeNull();
      paidOffEvent.Fee.ShouldEqual(fee);
      }&lt;/code&gt;&lt;/pre&gt;
      &lt;p&gt;
  </noscript>

  <p>Finally, we need to actually fire off these domain events. This is something we can hook into our infrastructure of whatever ORM we’re using. In EF, we can hook into the SaveChanges method:</p>
  <p>
  

<script src="https://gist.github.com/50407a1c1a823a83e9bb.js"></script><link rel="stylesheet" href="https://gist-assets.github.com/assets/embed-8bf0013c72fb64f0bb1bc1872b43e39e.css"></p><div id="gist11739426" class="gist">
      <div class="gist-file">
          <div class="gist-data gist-syntax">

  <noscript>
      &lt;pre&gt;&lt;code class="language-c# c#"&gt;public override int SaveChanges() {
      var domainEventEntities = ChangeTracker.Entries&amp;lt;IEntity&amp;gt;()
      .Select(po =&amp;gt; po.Entity)
      .Where(po =&amp;gt; po.Events.Any())
      .ToArray();

      foreach (var entity in domainEventEntities)
      {
      var events = entity.Events.ToArray();
      entity.Events.Clear();
      foreach (var domainEvent in events)
      {
      _dispatcher.Dispatch(domainEvent);
      }
      }

      return base.SaveChanges();
      }&lt;/code&gt;&lt;/pre&gt;
      &lt;p&gt;
  </noscript>

  <p>Just before we commit our transaction, we dispatch our events to their respective handlers. With this approach, we decouple the raising of a domain event from dispatching to a handler, which is much more obvious to me as a developer. If you want immediate side effects with other entities, just pass those in as arguments.</p>
  <p>In dispatching our domain events, we have some flexibility now. We can dispatch synchronously, as we did in our EF example, or asynchronously by persisting our events as JSON and having an offline process pick up those undispatched events and dispatch them out-of-band. All of this is decoupled from our domain event raiser and handler.</p>
  <p>If there’s one lesson to be learned here, it’s to beware of static, opaque dependencies, even it is a really cool concept. Separating the two concerns of raising versus dispatching keeps our domain model fully encapsulated without introducing land mines in our model.</p>
  
