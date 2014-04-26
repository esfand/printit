<h1>Java 8: Definitive guide to CompletableFuture</h1>

<p>Java 8 is coming so it's time to study new features. While Java 7 and 
Java 6 were rather minor releases, version 8 will be a big step forward. 
Maybe even too big? Today I will give you a thorough explanation of new
abstraction in JDK 8 - <code>CompletableFuture&lt;T&gt;</code>. 
As you all know Java 8 will hopefully be released in less than a year, 
therefore this article is based on .  <code>CompletableFuture&lt;T&gt;</code>
extends <code>Future&lt;T&gt;</code> by providing functional, monadic (!) 
operations and promoting asynchronous, event-driven programming model, 
as opposed to blocking in older Java. If you opened JavaDoc of 
<code>CompletableFuture&lt;T&gt;</code> you are surely overwhelmed. 
About <b>fifty methods</b> (!), some of them being extremely cryptic and
exotic, e.g.:</p>

<pre class="brush: java">
public &lt;U,V&gt; CompletableFuture&lt;V&gt; 
   thenCombineAsync(CompletableFuture&lt;? extends U&gt; other, 
                                  BiFunction&lt;? super T,? super U,? extends V&gt; fn, 
                                  Executor executor)
</pre>

</p>Don't worry, but keep reading. <code>CompletableFuture</code> collects
all the features of <code>ListenableFuture</code> in Guavawith 
<code>SettableFuture</code>.  Moreover built-in lambda support brings it
closer to Scala/Akka futures. Sounds too good to be true, but keep reading.
<code>CompletableFuture</code> has two major areas superior to good ol' 
<code>Future&lt;T&gt;</code> - 
asynchronous callback/transformations support and 
the ability to set value of <code>CompletableFuture</code> from any thread at
any point in time. </p>

<h2>Extract/modify wrapped value</h2>

<p>Typically futures represent piece of code running by other thread. But that's 
not always the case. Sometimes you want to create a <code>Future</code> 
representing some event that you know will occur, e.g. JMS message arrival. 
So you have <code>Future&lt;Message&gt;</code> but there is no asynchronous job 
underlying this future. You simply want to complete (resolve) that future when 
JMS message arrives, and this is driven by an event. In this case you can simply 
create <code>CompletableFuture</code>, return it to your client and whenever
you think your results are available, simply <code>complete()</code> the future 
and unlock all clients waiting on that future.</p>

<p>For starters you can simply create new <code>CompletableFuture</code> out of 
thin air and give it to your client:</p>

<pre class="brush: java">
public CompletableFuture&lt;String&gt; ask() {
    final CompletableFuture&lt;String&gt; future = new CompletableFuture&lt;&gt;();
    //...
    return future;
}
</pre>

<p>Notice that this future is not associated wtih any
<code>Callable&lt;String&gt;</code>, no thread pool, no asynchronous job. 
If now the client code calls <code>ask().get()</code> it will block forever. 
If it registers some completion callbacks, they will never fire. 
So what's the point? Now you can say:</p>

<pre class="brush: java">
future.complete("42")
</pre>

...and at this very moment all clients blocked on <code>Future.get()</code> will 
get the result string. Also completion callbacks will fire immediately. 
This comes quite handy when you want to represent a task in the future, but not 
necessarily computational task running on some thread of execution. 
<code>CompletableFuture.complete()</code> can only be called once, subsequent 
invocations are ignored. But there is a back-door called
<code>CompletableFuture.obtrudeValue(...)</code> which overrides previous value 
of the <code>Future</code> with new one. Use with caution.<br>
<br>

Sometimes you want to signal failure. As you know <code>Future</code> objects can handle either wrapped result or exception. If you want to pass some exception further, there is <code>CompletableFuture.completeExceptionally(ex)</code> (and <code>obtrudeException(ex)</code> evil brother that overrides the previous exception). <code>completeExceptionally()</code> also unlock all waiting clients, but this time throwing an exception from <code>get()</code>. Speaking of <code>get()</code>, there is also <code>CompletableFuture.join()</code> method with some subtle changes in error handling. But in general they are the same. And finally there is also <code>CompletableFuture.getNow(valueIfAbsent)</code> method that doesn't block but if the <code>Future</code> is not completed yet, returns default value. Useful when building robust systems where we don't want to wait too much.<br>
<br>

Last <code>static</code> utility method is <code>completedFuture(value)</code> that returns already completed <code>Future</code> object. Might be useful for testing or when writing some adapter layer.<br>
<br>

<h2>Creating and obtaining <code>CompletableFuture</code></h2>

OK, so is creating <code>CompletableFuture</code> manually our only option? Not quite. Just as with normal <code>Future</code>s we can wrap existing task with <code>CompletableFuture</code> using the following family of factory methods:<br>
<br>

<pre class="brush: java">
static &lt;U&gt; CompletableFuture&lt;U&gt; supplyAsync(
                                            Supplier&lt;U&gt; supplier);
static &lt;U&gt; CompletableFuture&lt;U&gt; supplyAsync(
                                            Supplier&lt;U&gt; supplier, 
                                            Executor executor);
static CompletableFuture&lt;Void&gt; runAsync(Runnable runnable);
static CompletableFuture&lt;Void&gt; runAsync(Runnable runnable, 
                                              Executor executor);
</pre>

Methods that do not take an <code>Executor</code> as an argument but end with <code>...Async</code> will use <code>ForkJoinPool.commonPool()</code></a> (global, general purpose pool introduces in JDK 8). This applies to most methods in <code>CompletableFuture</code> class. <code>runAsync()</code> is simple to understand, notice that it takes <code>Runnable</code>, therefore it returns <code>CompletableFuture&lt;Void&gt;</code> as <code>Runnable</code> doesn't return anything. If you need to process something asynchronously and return result, use <code>Supplier&lt;U&gt;</code>:<br>
<br>

<pre class="brush: java">
final CompletableFuture&lt;String&gt; 
       future = CompletableFuture.supplyAsync(new Supplier&lt;String&gt;() {

    @Override
    public String get() {
        //...long running...
        return "42";
    }
}, executor);
</pre>

<p>But hey, we have lambdas in Java 8!</p>

<pre class="brush: java">
final CompletableFuture&lt;String&gt; future = CompletableFuture.supplyAsync(() -&gt; {
    //...long running...
    return "42";
}, executor);
</pre>

or even:<br>
<br>

<pre class="brush: java">
final CompletableFuture&lt;String&gt; 
       future = CompletableFuture.supplyAsync(() -&gt; longRunningTask(params),
                                              executor);
</pre>

<p>This article is not about project Lambda, but I will be using lambdas quite extensively.</p>

<h2>Transforming and acting on one <code>CompletableFuture</code> (<code>thenApply</code>)</h2>

So I said that <code>CompletableFuture</code> is superior to <code>Future</code> but you haven't yet seen why? Simply put, it's because <code>CompletableFuture</code> is a monad and a functor. Not helping I guess? Both Scala and JavaScript allow registering asynchronous callbacks when future is completed. We don't have to wait and block until it's ready. We can simply say: <i>run this function on a result, when it arrives</i>. Moreover, we can stack such functions, combine multiple futures together, etc. For example if we have a function from <code>String</code> to <code>Integer</code> we can turn <code>CompletableFuture&lt;String&gt;</code> to <code>CompletableFuture&lt;Integer</code> without unwrapping it. This is achieved with <code>thenApply()</code> family of methods:<br>
<br>

<pre class="brush: java">
&lt;U&gt; CompletableFuture&lt;U&gt; thenApply(Function&lt;? super T,? extends U&gt; fn);
&lt;U&gt; CompletableFuture&lt;U&gt; thenApplyAsync(Function&lt;? super T,? extends U&gt; fn);
&lt;U&gt; CompletableFuture&lt;U&gt; thenApplyAsync(Function&lt;? super T,? extends U&gt; fn, Executor executor);
</pre>

As stated before <code>...Async</code> versions are provided for most operations on <code>CompletableFuture</code> thus I will skip them in subsequent sections. Just remember that first method will apply function within the same thread in which the future completed while the remaining two will apply it asynchronously in different thread pool.<br>
<br>
Let's see how <code>thenApply()</code> works:<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;String&gt; f1 = //...
CompletableFuture&lt;Integer&gt; f2 = f1.thenApply(Integer::parseInt);
CompletableFuture&lt;Double&gt; f3 = f2.thenApply(r -&gt; r * r * Math.PI);
</pre>

Or in one statement:<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;Double&gt; f3 =
    f1.thenApply(Integer::parseInt).thenApply(r -&gt; r * r * Math.PI);
</pre>

<p>You see a sequence of transformations here. From <code>String</code> to <code>Integer</code> and then to <code>Double</code>. But what's most important, these transformations are neither executed immediately nor blocking. They are simply remembered and when original <code>f1</code> completes they are executed for you. If some of the transformations are time-consuming, you can supply your own <code>Executor</code> to run them asynchronously. Notice that this operation is equivalent to monadic <code>map</code> in Scala.</p>

<h2>
Running code on completion
(<code>thenAccept</code>/<code>thenRun</code>)
</h2>

<pre class="brush: java">
CompletableFuture&lt;Void&gt; thenAccept(Consumer&lt;? super T&gt; block);
CompletableFuture&lt;Void&gt; thenRun(Runnable action);
</pre>

These two methods are typical "final" stages in future pipeline. They allow you to consume future value when it's ready. While <code>thenAccept()</code> provides the final value, <code>thenRun</code> executes <code>Runnable</code> which doesn't even have access to computed value. Example:<br>
<br>
<pre class="brush: java">
future.thenAcceptAsync(dbl -&gt; log.debug("Result: {}", dbl), executor);
log.debug("Continuing");
</pre>

<code>...Async</code> variants are available as well for both methods, with implicit and explicit executor. I can't emphasize this enough: <code>thenAccept()</code>/<code>thenRun()</code> methods <b>do not block</b> (even without explicit <code>executor</code>). Treat them like an event listener/handler that you attach to a future and that will execute some time in the future. <code>"Continuing"</code> message will appear immediately, even if <code>future</code> is not even close to completion.<br>
<br>

<h2>Error handling of single <code>CompletableFuture</code></h2>

So far we only talked about result of computation. But what about exceptions? Can we handle them asynchronously as well? Sure!<br>
<br>
<pre class="brush: java">
CompletableFuture&lt;String&gt; safe = 
    future.exceptionally(ex -&gt; "We have a problem: " + ex.getMessage());
</pre>
<code>exceptionally()</code> takes a function that will be invoked when original future throws an exception. We then have an opportunity to recover by transforming this exception into some value compatible with <code>Future</code>'s type. Further transformations of <code>safe</code> will no longer yield an exception but instead a <code>String</code> returned from supplied function.<br>
<br>
A more flexible approach is <code>handle()</code> that takes a function receiving either correct result or exception:<br>

<pre class="brush: java">
CompletableFuture&lt;Integer&gt; safe = future.handle((ok, ex) -&gt; {
    if (ok != null) {
        return Integer.parseInt(ok);
    } else {
        log.warn("Problem", ex);
        return -1;
    }
});
</pre>

<p><code>handle()</code> is called always, with either result or exception argument being not-<code>null</code>. This is a one-stop catch-all strategy.</p>

<h2>Combining two <code>CompletableFuture</code> together</h2>

Asynchronous processing of one <code>CompletableFuture</code> is nice but it really shows its power when multiple such futures are combined together in various ways.<br>
<br>

<h3>Combining (chaining) two futures (<code>thenCompose()</code>)</h3>

Sometimes you want to run some function on future's value (when it's ready). But this function returns future as well. <code>CompletableFuture</code> should be smart enough to understand that the result of our function should now be used as top-level future, as opposed to <code>CompletableFuture&lt;CompletableFuture&lt;T&gt;&gt;</code>. Method <code>thenCompose()</code> is thus equivalent to <code>flatMap</code> in Scala:<br>
<br>

<pre class="brush: java">
&lt;U&gt; CompletableFuture&lt;U&gt; thenCompose(Function&lt;? super T,CompletableFuture&lt;U&gt;&gt; fn);
</pre>

<code>...Async</code> variations are available as well. Example below, look carefully at the types and the difference between <code>thenApply()</code> (<code>map</code>) and <code>thenCompose()</code> (<code>flatMap</code>) when applying a <code>calculateRelevance()</code> function returning <code>CompletableFuture&lt;Double&gt;</code>:<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;Document&gt; docFuture = //...

CompletableFuture&lt;CompletableFuture&lt;Double&gt;&gt; f =
    docFuture.thenApply(this::calculateRelevance);

CompletableFuture&lt;Double&gt; relevanceFuture =
    docFuture.thenCompose(this::calculateRelevance);

//...

private CompletableFuture&lt;Double&gt; calculateRelevance(Document doc)  //...
</pre>

<code>thenCompose()</code> is an essential method that allows building robust, asynchronous pipelines, without blocking or waiting for intermediate steps.<br>
<br>

<h3>Transforming values of two futures (<code>thenCombine()</code>)</h3>

While <code>thenCompose()</code> is used to chain one future dependent on the other, <code>thenCombine</code> combines two independent futures when they are both done:<br>
<br>

<pre class="brush: java">
&lt;U,V&gt; CompletableFuture&lt;V&gt; thenCombine(CompletableFuture&lt;? extends U&gt; other, BiFunction&lt;? super T,? super U,? extends V&gt; fn)
</pre>

<code>...Async</code> variations are available as well. Imagine you have two <code>CompletableFuture</code>s, one that loads <code>Customer</code> and other that loads nearest <code>Shop</code>. They are completely independent from each other, but when both of them are completed, you want to use their values to calculate <code>Route</code>. Here is a stripped example:<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;Customer&gt; customerFuture = loadCustomerDetails(123);
CompletableFuture&lt;Shop&gt; shopFuture = closestShop();
CompletableFuture&lt;Route&gt; routeFuture = 
    customerFuture.thenCombine(shopFuture, (cust, shop) -&gt; findRoute(cust, shop));

//...

private Route findRoute(Customer customer, Shop shop) //...
</pre>

Notice that in Java 8 you can replace <code>(cust, shop) -&gt; findRoute(cust, shop)</code> with simple <code>this::findRoute</code> method reference:<br>
<br>

<pre class="brush: java">
customerFuture.thenCombine(shopFuture, this::findRoute);
</pre>

So you get the idea. We have <code>customerFuture</code> and <code>shopFuture</code>. Then <code>routeFuture</code> wraps them and "waits" for both to complete. When both of them are ready, it runs our supplied function that combines results (<code>findRoute()</code>). Thus <code>routeFuture</code> will complete when two underlying futures are resolved <i>and</i> <code>findRoute()</code> is done.<br>
<br>

<h3>
Waiting for <i>both</i> <code>CompletableFuture</code>s to complete</h3>

If instead of producing new <code>CompletableFuture</code> combining both results we simply want to be notified when they finish, we can use <code>thenAcceptBoth()</code>/<code>runAfterBoth()</code> family of methods (<code>...Async</code> variations are available as well). They work similarly to <code>thenAccept()</code> and <code>thenRun()</code> but wait for two futures instead of one:<br>
<br>

<pre class="brush: java">
&lt;U&gt; CompletableFuture&lt;Void&gt; thenAcceptBoth(CompletableFuture&lt;? extends U&gt; other, BiConsumer&lt;? super T,? super U&gt; block)
CompletableFuture&lt;Void&gt; runAfterBoth(CompletableFuture&lt;?&gt; other, Runnable action)
</pre>

Imagine that in the example above, instead of producing new <code>CompletableFuture&lt;Route&gt;</code> you simply want send some event or refresh GUI immediately. This can be easily achieved with <code>thenAcceptBoth()</code>:<br>
<br>

<pre class="brush: java">
customerFuture.thenAcceptBoth(shopFuture, (cust, shop) -&gt; {
    final Route route = findRoute(cust, shop);
    //refresh GUI with route
});
</pre>

I hope I'm wrong but maybe some of you are asking themselves a question: <i>why can't I simply block on these two futures?</i> Like here:<br>
<br>

<pre class="brush: java">
Future&lt;Customer&gt; customerFuture = loadCustomerDetails(123);
Future&lt;Shop&gt; shopFuture = closestShop();
findRoute(customerFuture.get(), shopFuture.get());
</pre>

Well, of course you can. But the whole point of <code>CompletableFuture</code> is to allow asynchronous, event driven programming model instead of blocking and eagerly waiting for result. So functionally two code snippets above are equivalent, but the latter unnecessarily occupies one thread of execution.<br>
<br>

<h2>Waiting for first <code>CompletableFuture</code> to complete</h2>

Another interesting part of the <code>CompletableFuture</code> API is the ability to wait for <i>first</i> (as opposed to <i>all</i>) completed future. This can come handy when you have two tasks yielding result of the same type and you only care about response time, not which task resulted first. API methods (<code>...Async</code> variations are available as well):<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;Void&gt; acceptEither(CompletableFuture&lt;? extends T&gt; other, Consumer&lt;? super T&gt; block)
CompletableFuture&lt;Void&gt; runAfterEither(CompletableFuture&lt;?&gt; other, Runnable action)
</pre>

As an example say you have two systems you integrate with. One has smaller average response times but high standard deviation. Other one is slower in general, but more predictable. In order to take best of both worlds (performance and predictability) you call both systems at the same time and wait for the first one to complete. Normally it will be the first one, but in case it became slow, second one finishes in an acceptable time:<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;String&gt; fast = fetchFast();
CompletableFuture&lt;String&gt; predictable = fetchPredictably();
fast.acceptEither(predictable, s -&gt; {
    System.out.println("Result: " + s);
});
</pre>

<code>s</code> represents <code>String</code> reply either from <code>fetchFast()</code> or from <code>fetchPredictably()</code>. We neither know nor care.<br>
<br>

<h3>Transforming first completed</h3>

<code>applyToEither()</code> is an older brother of <code>acceptEither()</code>. While the latter simply calls some piece of code when faster of two futures complete, <code>applyToEither()</code> will return a new future. This future will complete when <i>first</i> of the two underlying futures complete. API is a bit similar (<code>...Async</code> variations are available as well):<br>
<br>

<pre class="brush: java">
&lt;U&gt; CompletableFuture&lt;U&gt; applyToEither(CompletableFuture&lt;? extends T&gt; other, Function&lt;? super T,U&gt; fn)
</pre>

The extra <code>fn</code> function is invoked on the result of first future that completed. I am not really sure what's the purpose of such a specialized method, after all one could simply use: <code>fast.applyToEither(predictable).thenApply(fn)</code>. Since we are stuck with this API but we don't really need extra function application, I will simply use <a href="http://download.java.net/lambda/b88/docs/api/java/util/function/Function.html#identity()"><code>Function.identity()</code></a> placeholder:<br>
<br>

<pre class="brush: java">
CompletableFuture&lt;String&gt; fast = fetchFast();
CompletableFuture&lt;String&gt; predictable = fetchPredictably();
CompletableFuture&lt;String&gt; firstDone = 
    fast.applyToEither(predictable, Function.&lt;String&gt;identity());
</pre>

<code>firstDone</code> future can then be passed around. Notice that from the client perspective the fact that two futures are actually behind <code>firstDone</code> is hidden. Client simply waits for future to complete and <code>applyToEither()</code> takes care of notifying the client when any of the two finish first.<br>
<br>

<h2>Combining multiple <code>CompletableFuture</code> together</h2>

So we now know how to wait for two futures to complete (using <code>thenCombine()</code>) and for the first one to complete (<code>applyToEither()</code>). But can it scale to arbitrary number of futures? Sure, using <code>static</code> helper methods:<br>
<br>

<pre class="brush: java">
static CompletableFuture&lt;Void&gt; allOf(CompletableFuture&lt;?&gt;... cfs)
static CompletableFuture&lt;Object&gt; anyOf(CompletableFuture&lt;?&gt;... cfs)
</pre>

<code>allOf()</code> takes an array of futures and returns a future that completes when all of the underlying futures are completed (barrier waiting for all). <code>anyOf()</code> on the other hand will wait only for the fastest of the underlying futures. Please look at the generic type of returned futures. Not quite what you would expect? We will take care of this issue in the next article.<br>
<br>

<h2>Summary</h2>

We explored pretty much whole <code>CompletableFuture</code> API. 
I'm sure this was quite overwhelming so in the next article shortly we will
develop yet another implementation of simple web crawling program, taking 
advantage of <code>CompletableFuture</code> functionalities and Java 8
lambdas. We will also look at disadvantages and shortcomings of 
<code>CompletableFuture</code>.<br>
<br>

