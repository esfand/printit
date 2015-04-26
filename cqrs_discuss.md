Patrick • 18 days ago    
re: "Unfortunately a lot of existing LOB applications do not follow this rather straight forward pattern."

I agree that the pattern is straight forward, but do you feel like the user-experience takes on an additional layer of complexity if the design of the system should be one where the user can "read what they wrote"? The amazon shopping experience, as an example, has trained the user to expect asynchronous updates. Similarly, paying your credit card online has a similar non-immediate expectation. However, suppose you are building an SaaS accounting platform where your targeted user base has an expectation that when they pay a batch of invoices, the invoices are immediately closed and balanced out to $0. You could argue that they need to be re-trained an accept an asynchronous user experience, but for the sake of conversation, suppose the business doesn't want to do so.

With a single read/write model, the synchronous user experience is "free". Of course, this comes at other costs, which your article does a good job describing. When you split the models, doesn't UX become more complex? I've heard of people using client-side state in the browser to simulate read-what-you-wrote - there is some engineering required to pull this off.

Anyhow, good write up. You definitely highlight some of the strength of this approach. It would be interesting to see a post that describes the complexities of CQRS. Everything has a trade-off, right?


gabrielschenker Mod  Patrick • 18 days ago    
In my first sample of a classical n-tier application using an RDBMS as back-end storage everything is synchronous. CQRS by no ways implies that we have to use event sourcing or an eventual consistent read model. The latter is just an option which scales out nicely.

Stefan Billiet  Patrick • 18 days ago    
Concerning the the synchronous user experience:

It depends. There are workflows where you are changing data in one place, that is used in a completely different place. In this case async doesn't matter because by the time you get to that other screen, the async process would have already been completed (unless it's a long running process).
If the changed data needs to be "immediately" visible, you can setup a system where the backend sends (domain) events back to the frondend (sort of server-sent events), through libraries like SignalR or socket.io. If you have this system, you just wait on the page after having pressed save untill you get an event back indicating the job is done; then you can refresh the page.

Having set up something like this recently, if your backend is already eventdriven, then sending those events back to the browser is reasonably straightforward.

jbogard  Stefan Billiet • 18 days ago    
"There are workflows where you are changing data in one place, that is used in a completely different place" <- I find these few and far between. Even in ecommerce systems where I've worked with billions of dollars flowing through them, pricing is most assuredly synchronous with the "back end". A nightly ETL job pushes these to the read-only caches on the front-end.

Users might not expect to see the side effects of their changes immediately, but they certainly expect to see their change immediately in whatever system they're using.

We used event sourcing in one app. Devs loved it, users h...a...t...e...d it, because every operation introduced latency through all this back-end event publishing, denormalizing, SignalR'ing etc etc. We were promised better scaling, and users complained the app was slow.

gabrielschenker Mod  jbogard • 18 days ago    
I'm sorry for you Jimmy if your user hated the app. I have a completely different experience. Our users loved it. But that's not the point here. You can use CQRS in an application that is fully synchronous all the time. By the way we should never forget that as soon as we build a web application everything is asynchronous!

jbogard  gabrielschenker • 18 days ago    
Oh yes - I was referring to async read models. Users expect consistency. That's what DBs are great at - providing transactional consistency and have spent man-centuries perfecting that model. Anything I build client-side to mimic it is a buggy, half-implemented version of a transaction. When I meet clients with async read models - in which the read model is in the same bounded context/app, I have them ditch it immediately and rip out all the plumbing they've put in to try and deal with it in the various layers of the app.

Stefan Billiet  jbogard • 18 days ago    
It depends on the interface; if it's a task based interface it's generally possible.
As to the situation you described, I can only assume something went wrong architecturally. If latency was such a big problem, you could always resort to handling projections in the same process as the handling of commands and only then publish the events.

jbogard  Stefan Billiet • 18 days ago    
Synchronous projections should be the default mode. Async projections goes against decades of previous art and millenia of how humans have interacted with each other.

In our task-based interfaces, the users always expected that at least they could see that their task was recorded. Users expect consistency. They don't expect "approve invoice" "view outstanding invoices" "wtf why is my approved invoice still there". We then have to mimic consistency with tricks, either with write-through client-side models, notifications that worsen client experience, or by simply forcing consistency by waiting on the server side for projections to finish.

It's also why every single one of our clients has ditched RavenDB. Nobody actually wanted asynchronous read models inside the same bounded context.

I use CQRS all the time now, just not the event-sourced async mode. Just instead of views I use LINQ projections.

gabrielschenker Mod  jbogard • 18 days ago    
You have a very strong opinion and I respect it. But please don't make it a "one has to..." global law. I can give you countless examples where it makes perfectly sense and it is even desirable to have asynchronously processed projections (eventual consistent read model). Right now I am working in a project where it is strictly required by the customer that everything happens asynchronously (we use NServiceBus). In my previous project which was a huge enterprise app we also used eventual consistent read models mixed with fully consistent ones. But one has to match that with a task based UI of course. CRUD-ish applications are not good candidates for eventual consistency.
Please let's not make this a religious war. As always "it depends" which approach is the "right" one.

jbogard  gabrielschenker • 17 days ago    
I'm genuinely curious what kind of problem needs async projections in the same app/context boundary. I've seen it implemented countless times, and every time it's been needless complexity, matched up with some sort of half-baked mimicked consistency in the front-end.

And these were *all* task-based UIs. To me that's orthogonal. What I never found were fire-and-forget messages. Users never wanted fire-and-forget except for very particular cases (send email). If something was a long-running process, they expected a synchronous recording of the request, and dispatching of the request. The request was recorded in a process entity with status, and back-end events updated the status of the request.

The places where I've seen Event Sourcing (not CQRS) work well are places where the domain is actually around complex event processing. And not "Approve Invoice/Invoice Approved" sort of things, but where the business conceptualized their domain, thought about it, dreamed about it, represented it, as a stream of events. Like stock prices, for example.

gabrielschenker Mod  jbogard • 17 days ago    
I will try to author another post about synchronous versus asynchronous. There I hope to clarify certain open questions. But once again this post was first and foremost about the CQRS pattern which is in no way related to the question of synchronous versus asynchronous. CQRS is about recognizing that changing state is fundamentally different from querying state and that as a consequence these two concerns should be treated separately if possible. Also I mentioned it in my post that I am strictly referring to (somewhat) complex LOB applications.

jbogard  gabrielschenker • 17 days ago    
Oh of course - we use CQRS in just about every app we build these days. It's the Event Sourcing part (one that often gets conflated with CQRS) that I was cautioning on. And its value has less to do with domain complexity than the business already seeing work done as processing events.

gabrielschenker Mod  jbogard • 17 days ago
I knew we will be on the same page, eventually ;-)

Andrew Siemer  jbogard • 17 days ago    
The thing that I find happens all the time when folks talk about CQRS is that they quickly down grade the conversation into all the things that people like to dangle off the pattern of CQRS. The pattern doesn't dictate synch vs asynch. Though having components that are asynch can enable scalability and having task based UI's vs crud based UI's are a major part of the retraining effort to drive the health of the internal model and how you maneuver changes to that model over time.

But there are always strategies to get around this any of the shortcomings that you might bump against depending on the client/project needs. FB does a good job of this on first approach. I post something to my wall. And in my view the post is on my wall. However, if I immediately open a new window and view my wall it has not yet found its way there. Essentially they either cached my addition locally or locked me in to a server path that had my changes in its realm. I feel that thy are making head way on educating users about eventual consistency but they do a great job of ensuring that most users have no idea that this misdirection has happened. And they have a good enough durability picture in their infrastructure that they mean that it will truly eventually be consistent. If you don't have the ability to ensure durability - yes - eventual consistency is a horrible idea.

But in many cases I agree that this "experience" is not acceptable to business users on business apps. But oddly enough the users of business systems have been mis-trained to expect this immediate processing/consistency. I like to provide a representation of reality where we treat system transactions as though they are pages in the steps of the process. When I submit something I guarantee that I have the document with a status of "posted", "submitted", or "received" --- not "accepted" or "processed" etc. This initial transaction may be consistent - not eventual. Posted simply means that we have it. The answer (such as for loan processing, or order submission) will come eventually - and eventually may usually be a wicked fast rate in that the consumer of the application may never see the initial status. But at least with this model the user sees the reality of the system.

The best use case to point too for this way of doing things in most of our every day lives is when you buy something from Amazon. They take my card info and the order. They send me an order received email. Everything - for the majority of users - appears that the order is good to go. However, if for some reason the card doesn't go through at a later time they loop me into another process where they attempt to get updated payment information. This too is an acceptable experience that does at times require some end user retraining. But most of the time it is a feature of the system that is totally invisible.

jbogard  Andrew Siemer • 17 days ago    

Small quibble - this isn't "mistraining". It's millenia of human experience. I write down an order on a piece of paper, hand it to a human - they don't forget about my order.

Let's say we have the 2 tabs issue - this DOES happen IRL. I hand a cashier a deposit, they write it down. I go to the next cashier, they don't have my deposit yet and the world doesn't explode.

Why? Because the first teller *tells me* that "my deposit will show up in a couple of days". The user experience doesn't pretend it's sync - it's not, but it explicitly builds around an async process, makes it completely up-front and obvious.

If you look at talks done by Facebook/Disqus/Twitter engineers, NONE of them built this async architecture at the beginning. It's much more complicated. They only did it when demands of scale forced them to, years into the product and only when they absolutely had to.

My single biggest beef with the event sourcing community is that the async architecture is somehow simpler, despite so, so much evidence to the contrary. Your Amazon example is more an example of a "Saga" or "Process manager" - every step is synchronous, but there's an overall asynchronous flow. Very, very different than "I approved the invoice, I go to the open invoices and it's still there". If you wanted to have a good UX, you'd have something like dual write models on client and server, or an explicit UX that tells the user that this request is getting processed in the backend.

Separate "asynchronous, long-running business transaction" from "asynchronous read models" - users understand the first, but the second is far more complicated to both manage from the UX and backend perspective.

Patrick  jbogard • 18 days ago    
The "wtf why is my approved invoice still there" comment is spot on - this is where CQRS, IMO, becomes less "straight-forward" and may itself create a lot of "unnecessary complexity". For other parts of the system, like a dashboard with some summary-level information, separating the read-model feels much more natural. This aligns with your previous remark:

> "Users might not expect to see the side effects of their changes immediately, but they certainly expect to see their change immediately in whatever system they're using."

+1 .. good thread

Andrew Siemer  Patrick • 16 days ago    
The problem I have with the statement above "this is where CQRS IMO becomes less..." CQRS doesn't assume any form of event sourcing, eventual consistency, etc. It is just a pattern to separate your read and write models/paths. You don't have to put DDD behind it, or a messaging sub system, etc. The CQRS as a pattern conversations always gets (no offense) polluted with additional threads of conversation that reflect poorly on a very simple pattern.

Daniel Whittaker  Patrick • 18 days ago    
"Everything has a trade-off" - so true. I've been using CQRS and event sourcing for a few years now. The biggest trade off I've noticed is in the initial learning curve. It is a very different way of thinking and as such creates a steep learning curve. I've also found the infrastructure code to be more complex, however, once it is done it rarely changes.

On the other side, in most cases, I've found it much easier to add new features and change the way existing things work. I really love the knock on benefits of having an event store. Being able to replay events into new features so they look like they existed from the beginning of the application. This is particularly when building reports. I've been blogging a lot about CQRS and event sourcing recently. Feel free to take a look to get some more insight: http://danielwhittaker.me

Gabriel - Thanks for the great post!

gabrielschenker Mod  Daniel Whittaker • 18 days ago    
I agree, the learning curve is quite steep. It is a paradigm shift. It's somewhat similar to other areas e.g. using the VI editor versus other editors. Once it makes "click" though you don't want to go back ever :-)

Blake H • 12 days ago    
Excellent Post Gabriel. I'm a huge fan of CQRS/ES and I believe, if applied correctly, it solves a lot of issues that the standard relational CRUD model cannot. Chief among these issues is scalability, partitionability, elimination of transaction deadlocks from reading where you write, and providing fast targeted reads from an appropriate data store (graph, relational, document db) for different data sets, etc,etc.

I think the biggest benefit for developers is that it keeps the code clean and maintainable... the biggest benefit for business is that it allows for scale without a complete rewrite, and it can provide insights on data that other methods just can't.

That being said, eventual consistency does require some new ways of thinking and is not without it's own "baggage". But this article brings it home for me: http://www.xaprb.com/blog/2014...

If you are looking to produce an MVP, then maybe CRUD is the way to go. If you are looking to build a system for a lot of users, and needs to survive for the long haul, I'd recommend you use a different pattern.

w1ngnut • 12 days ago    
Hello Gabriel,
I've been searching for approaches to handling eventual consistency in the front-end apart from "Thanks we are working on your request". Any thoughts on this?

Regards.

gabrielschenker Mod  w1ngnut • 12 days ago    
Eventual consistency can only really work well in conjunction with a task based UI. First of all we want to make sure that each command that is sent to the back-end is very specific and if possible limited in what is changed. Secondly we should make sure that the command is highly likely to succeed. Then you shouldn't reload data without need. If a command succeeds usually the corresponding view has already all necessary data displayed. Also eventual consistency means that in general the read model is in sync in a few milliseconds. That is nothing compared to the normal reaction time of a user. If the user navigates away from the current view to another view that needs the updated data then this happens in a matter of 100-eds of milliseconds.

w1ngnut  gabrielschenker • 12 days ago    
Thanks for the reply! Yes, I'm very aware of those requirements. But my question is more about scenarios where the system is not capable of processing requests in less than a few seconds (let's say 5s) for whatever reason (latency, performance, load etc).

Any suggestions in dealing with this?

gabrielschenker Mod  w1ngnut • 12 days ago    
I need to have a specific scenario to try to answer it. As always "it depends". Can you give a concrete sample?

w1ngnut  gabrielschenker • 12 days ago    
A very simple example could be: user submits information on (a task-based) screen A and navigates to screen B expecting to see that result in a grid. If that happened before the 5s mentioned above, he wouldn't see his record and would be very frustrated. How to approach this? Thanks.

gabrielschenker Mod  w1ngnut • 12 days ago    
As I said above, normally the eventual consistent read model is in sync with the write model in a matter of milliseconds. That is way more than the user needs to navigate to another screen. If for some reason the time is not sufficient and the read model is stale then one could use push notification technologies such as signal-r to push updates to the screen or the screen can poll for updates. If you are using auto save then you do not need to trigger a command only when the user clicks e.g. Next to navigate to the next screen and you will never run into the problem of out of sync...

w1ngnut  gabrielschenker • 11 days ago    
Makes sense! Thanks for the input.

Mike Rowley • 18 days ago    
In your opinion with regard to CQRS (no es) + DDD, would domain services and domain factories make use of the read model (i.e. IQuery<tdto>) or would they use a IQueryRepository<taggregate>? I struggle with this because factories and services often need more than GetById and need to do querying but if they use IQuery<dto> then Dtos are exposed in the domain which doesn't feel right.

gabrielschenker Mod  Mike Rowley • 18 days ago    
Good question. This is more of a DDD question and not so much about CQRS. But my rule of thumb is that the domain should never rely on an eventual consistent read model. Also most often if you need something other than GetById in the domain that "could" be an indication of a design flaw. Normally an aggregate encompasses all that is usually needed for processing. If that is not sufficient then maybe different instances of aggregates need to collaborate. But then again those instances are retrieved by their respective ID. I am sorry but the topic is way too complex to discuss it in a comment. I will try to author a post specifically about how I/we do DDD.

Andrew Siemer  gabrielschenker • 16 days ago    
Gabriel - this is fun!
