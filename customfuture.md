<h1>Implementing custom Future</h1>

<p><code>Future&lt;T&gt;</code> is typically returned by libraries or
 frameworks.  But there is nothing stopping us from implementing it all by
ourselves when it makes sense.  It is not particularly complex and may
significantly improve your design. I did my best to pick interesting use case 
for our example.</p>

<p>JMS (Java Message Service) is a standard Java API for sending
asynchronous messages. When we think about JMS, we immediately see a
client sending a message to a server (broker) in a <i>fire and forget</i>
manner. But it is equally common to implement <i>request-reply</i> messaging
pattern on top of JMS. </p>

<p>The implementation is fairly simple: you send a request message (of course
asynchronously) to an MDB on the other side. MDB processes the request and
sends a reply back either 1) to hardcoded <i>reply</i> queue or 2) to an
arbitrary queue chosen by the client and sent along with the message in 
<code>JMSReplyTo</code> property. The second scenario is much more
interesting.  Client can create a temporary queue and use it as a reply queue
when sending a request.  This way each request/reply pair uses different reply
queue, this there is no need for correlation ID, selectors, etc.</p>

<p>There is one catch, however. Sending a message to JMS broker is simple
and asynchronous. But receiving reply 
is much more cumbersome. You can either implement <code>MessageListener</code> to consume one, single message 
or use blocking <code>MessageConsumer.receive()</code>. First approach is quite heavyweight and hard to 
use in practice.  Second one defeats the purpose of asynchronous messaging. You can also poll the reply 
queue with some interval, which sounds even worse.</p>

<p>Knowing the <code>Future</code> abstraction by now you should have some design idea. What if we can send 
a request message and get a <code>Future&lt;T&gt;</code> back, representing reply message that didn't came yet? 
This <code>Future</code> abstraction should handle all the logic and we can safely use it as a handle to 
future outcome. Here is the plumbing code used to create temporary queue and send request:</p>

<pre class="brush: java">
private &lt;T extends Serializable&gt; Future&lt;T&gt; asynchRequest(
                 ConnectionFactory connectionFactory, 
                 Serializable      request, 
                 String            queue) 
                 throws JMSException {
        
    Connection connection = connectionFactory.createConnection();
    connection.start();
    final Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    final Queue tempReplyQueue = session.createTemporaryQueue();
    final ObjectMessage requestMsg = session.createObjectMessage(request);
    requestMsg.setJMSReplyTo(tempReplyQueue);
    sendRequest(session.createQueue(queue), session, requestMsg);
    return new JmsReplyFuture&lt;T&gt;(connection, session, tempReplyQueue);
}
</pre>

<p><code>asynchRequest()</code> method simply takes a <code>ConnectionFactory</code> to JMS broker and 
arbitrary piece of data. This object will be sent to <code>queue</code> using <code>ObjectMessage</code>. 
Last line is crucial - we return our custom <code>JmsReplyFuture&lt;T&gt;</code> that will represent 
<i>not-yet-received</i> reply. Notice how we pass temporary JMS queue to both <code>JMSReplyTo</code> 
property and our <code>Future</code>. Implementation of the MDB side is not that important. 
Needless to say it is suppose to send a reply back to designated queue:</p>

<pre class="brush: java">
final ObjectMessage reply = session.createObjectMessage(...);
session.createProducer(request.getJMSReplyTo()).send(reply);
</pre>

So let's dive into the implementation of <code>JmsReplyFuture&lt;T&gt;</code>. 
I made an assumption that both request and reply are <code>ObjectMessage</code>s. 
It's not very challenging to use a different type of message. 
First of all let us see how receiving messages from reply channel is set up:<br>
<br>

<pre class="brush: java">
public class JmsReplyFuture&lt;T extends Serializable&gt;
             implements Future&lt;T&gt;, MessageListener {

    //...

    public JmsReplyFuture(Connection connection, 
                                        Session session, 
                                        Queue replyQueue) 
                                        throws JMSException {
        this.connection = connection;
        this.session = session;
        replyConsumer = session.createConsumer(replyQueue);
        replyConsumer.setMessageListener(this);
    }

    @Override
    public void onMessage(Message message) {
        //...
    }
}
</pre>

As you can see <code>JmsReplyFuture</code> implements both <code>Future&lt;T&gt;</code> (where <code>T</code> 
is expected type of object wrapped inside <code>ObjectMessage</code>) and JMS <code>MessageListener</code>. 
In the constructor we simply start listening on <code>replyQueue</code>. 
From our design assumptions we know that there will be at most one message there because reply queue is temporary throw away queue. In the previous article</a> we learned that <code>Future.get()</code> should block while waiting for a result. On the other hand <code>onMessage()</code> is a callback method called from some internal JMS client thread/library. Apparently we need some shared variable/lock to let waiting <code>get()</code> know that reply arrived. Preferably our solution should be lightweight and not introduce any latency so busy waiting on <code>volatile</code> variable is a bad idea. Initially I though about <a href="http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Semaphore.html"><code>Semaphore</code></a> that I would use to unblock <code>get()</code> from <code>onMessage()</code>. But I would still need some shared variable to hold the actual reply object. So I came up with an idea of using <a href="http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ArrayBlockingQueue.html"><code>ArrayBlockingQueue</code></a>. It might sound strange to use a queue when we know there will be no more that one item. But it works perfectly, utilizing good old producer-consumer pattern: <code>Future.get()</code> is a consumer blocking on an empty queue's <code>poll()</code> method. On the other hand <code>onMessage()</code> is a producer, placing reply message in that queue and immediately unblocking consumer. Here is how it looks:<br>
<br>

<pre class="brush: java; highlight: [14, 25]">
public class JmsReplyFuture&lt;Textends Serializable&gt; 
             implements Future&lt;T&gt;, MessageListener {

    private final BlockingQueue&lt;T&gt; reply = new ArrayBlockingQueue&lt;&gt;(1);

    //...

    @Override
    public T get() throws InterruptedException, ExecutionException {
        return this.reply.take();
    }

    @Override
    public T get(long timeout, TimeUnit unit) 
                 throws InterruptedException, ExecutionException, TimeoutException {
        final T replyOrNull = reply.poll(timeout, unit);
        if (replyOrNull == null) {
            throw new TimeoutException();
        }
        return replyOrNull;
    }

    @Override
    public void onMessage(Message message) {
        final ObjectMessage objectMessage = (ObjectMessage) message;
        final Serializable object = objectMessage.getObject();
        reply.put((T) object);
        //...
    }

}
</pre>

<p>The implementation is still not complete, but it covers most important
concepts.  Notice how nicely <code>BlockingQueue.poll(long, TimeUnit)</code>
method fits into <code>Future.get(long, TimeUnit)</code>.   Unfortunately, even
though
they come from the same package and were developed more or less in the
same time, one method returns <code>null</code> upon timeout while the 
other should throw an exception. Easy to fix.</p>

<p>Also notice how easy the implementation of <code>onMessage()</code>
became.  We just place newly received message in a 
<code>BlockingQueue reply</code> and the collection does all the
synchronization for us. We are still missing some less significant, but still
important details - cancelling and clean up. Without going much into details,
here is a full implementation:</p>

<pre class="brush: java">
public class JmsReplyFuture&lt;T extends Serializable&gt; 
             implements Future&lt;T&gt;, MessageListener {

    private static enum State {WAITING, DONE, CANCELLED}

    private final Connection connection;
    private final Session session;
    private final MessageConsumer replyConsumer;
    private final BlockingQueue&lt;T&gt; reply = 
                          new ArrayBlockingQueue&lt;&gt;(1);
    private volatile State state = State.WAITING;

    public JmsReplyFuture(Connection connection, 
                                        Session session, 
                                        Queue replyQueue) 
                                        throws JMSException {
        this.connection = connection;
        this.session = session;
        replyConsumer = session.createConsumer(replyQueue);
        replyConsumer.setMessageListener(this);
    }

    @Override
    public boolean cancel(boolean mayInterruptIfRunning) {
        try {
            state = State.CANCELLED;
            cleanUp();
            return true;
        } catch (JMSException e) {
            throw Throwables.propagate(e);
        }
    }

    @Override
    public boolean isCancelled() {
        return state == State.CANCELLED;
    }

    @Override
    public boolean isDone() {
        return state == State.DONE;
    }

    @Override
    public T get() throws InterruptedException, ExecutionException {
        return this.reply.take();
    }

    @Override
    public T get(long timeout, TimeUnit unit) 
              throws InterruptedException, ExecutionException, TimeoutException {
        final T replyOrNull = reply.poll(timeout, unit);
        if (replyOrNull == null) {
            throw new TimeoutException();
        }
        return replyOrNull;
    }

    @Override
    public void onMessage(Message message) {
        try {
            final ObjectMessage objectMessage = (ObjectMessage) message;
            final Serializable object = objectMessage.getObject();
            reply.put((T) object);
            state = State.DONE;
            cleanUp();
        } catch (Exception e) {
            throw Throwables.propagate(e);
        }
    }

    private void cleanUp() throws JMSException {
        replyConsumer.close();
        session.close();
        connection.close();
    }
}
</pre>

<p>I use special <code>State</code> enum to hold the information about state. 
I find it much more readable compared to complex conditions based on multiple flags, <code>null</code> checks, etc. 
Second thing to keep in mind is cancelling. Fortunately it's quite simple. 
We basically close the underlying session/connection. It has to remain open throughout the course of whole request/reply
message exchange, otherwise temporary JMS reply queue disappears. Note that we cannot easily inform broker/MDB that 
we are no longer interested about the reply. We simply stop listening for it, but MDB will still process request and 
try to send a reply to no longer existing temporary queue.</p>

<hr/>

<p>So how does this all look in practice? Say we have an MDB that receives a number and returns a square of it. 
Imagine the computation takes a little bit of time so we start it in advance, do some work in the meantime and 
later retrieve the results. Here is how such a design might look like:<p>

<pre class="brush: java">
final Future&lt;Double&gt; replyFuture = asynchRequest(connectionFactory, 7, "square");
//do some more work
final double resp = replyFuture.get();      //49
</pre>

<p>Where <code>"square"</code> is the name of request queue. If we refactor 
it and use dependency injection we can further simplify it to something 
like:</p>

<pre class="brush: java">
final Future&lt;Double&gt; replyFuture = calculator.square(7);
//do some more work
final double resp = replyFuture.get();      //49
</pre>

<p>You know what's best about this design? Even though we are exploiting 
quite advanced JMS capabilities, there is no JMS code here. Moreover we 
can later replace <code>calculator</code> with a different implementation, 
using SOAP or GPU. As far as the client code is concerned, we still use 
<code>Future&lt;Double&gt;</code> abstraction.  Computation result that 
is not yet available. The underlying mechanism is irrelevant. 
That is the beauty of abstraction.</p>

<hr/>

<p>Obviously this implementation is not production ready (by far). But even
worse, it misses some essential features.  We still call blocking
<code>Future.get()</code> at some point.  Moreover there is no way of
composing/chaining futures (e.g. <i>when the response arrives, send another
message</i>) or waiting for the <i>fastest</i> future to complete. 
Be patient!</p>


