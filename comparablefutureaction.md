<h1>CompletableFuture in action</h1>

After thoroughly exploring `CompletableFuture` API in Java 8 we
are prepared to write a simplistic web crawler. We solved similar problem
already using  `ExecutorCompletionService`, Guava 
`ListenableFuture` and Scala/Akka. I choose the same problem
so that it's easy to compare approaches and implementation techniques.

First we shall define a simple, blocking method to download the contents of 
a single URL:

```java

private String downloadSite(final String site) {
    try {
        log.debug("Downloading {}", site);
        final String res = IOUtils.toString(new URL("http://" + site), UTF_8);
        log.debug("Done {}", site);
        return res;
    } catch (IOException e) {
        throw Throwables.propagate(e);
    }
}

```

Nothing fancy. This method will be later invoked for different sites inside 
thread pool. Another method parses the String into an XML Document 
(let me leave out the implementation, no one wants to look at it):

```java

private Document parse(String xml)  //...

```

Finally the core of our algorithm, function computing <i>relevance</i> of each 
website taking `Document` as input. Just as above we don't care about 
the implementation, only the signature is important:

```java

private CompletableFuture<Double> calculateRelevance(Document doc) //...

```

Let's put all the pieces together. Having a list of websites our crawler shall 
start downloading the contents of each web site asynchronously and concurrently.
Then each downloaded HTML string will be parsed to XML `Document` and later
_relevance_ will be computed. As a last step we take all computed _relevance_ 
metrics and find the biggest one. This sounds pretty straightforward to the 
oment when you realize that both downloading content and computing _relevance_ 
is asynchronous (returns `CompletableFuture`) and we definitely don't want to 
block or busy wait. Here is the first piece:    


```java
ExecutorService executor = Executors.newFixedThreadPool(4);
 
List<String> topSites = Arrays.asList("www.google.com", "www.youtube.com", 
                                      "www.yahoo.com",  "www.msn.com");
 
List<CompletableFuture<Double>> relevanceFutures = topSites.stream().
    map(site -> CompletableFuture.supplyAsync(() -> downloadSite(site), executor)).
    map(contentFuture -> contentFuture.thenApply(this::parse)).
    map(docFuture -> docFuture.thenCompose(this::calculateRelevance)).
    collect(Collectors.<CompletableFuture<Double>>toList());
```

There is actually *a lot* going on here. Defining thread pool and sites to 
crawl is obvious. But there is this chained expression computing 
`relevanceFutures`. The sequence of `map()` and `collect()` in the end is 
quite descriptive. Starting from a list of web sites we transform each 
site (`String`) into `CompletableFuture<String>` by submitting asynchronous 
task (`downloadSite()`) into thread pool.

So we have a list of `CompletableFuture<String>`. We continue transforming it, 
this time applying `parse()` method on each of them. Remember that `thenApply()` 
will invoke supplied lambda when underlying future completes and returns 
`CompletableFuture<Document>` immediately.  Third and last transformation step 
composes each `CompletableFuture<Document>` in the input list with 
`calculateRelevance()`. Note that `calculateRelevance()` returns 
`CompletableFuture<Double>` instead of `Double`, thus we use `thenCompose()` 
rather than `thenApply()`. After that many stages we finally `collect()` 
a list of `CompletableFuture<Double>`.

Now we would like to run some computations on _all_ results. We have a list of 
futures and we would like to know when all of them (last one) complete. Of course we can register completion callback on each future and use <code>CountDownLatch</code> to block until all callbacks are invoked. I am too lazy for that, let us utilize existing `CompletableFuture.allOf()`
 Unfortunately it has two minor drawbacks - takes vararg instead of <code>Collection</code> and doesn't return a future of aggregated results but <code>Void</code> instead. By aggregated results I mean: if we provide <code>List&lt;CompletableFuture&lt;Double&gt;&gt;</code> such method should return <code>CompletableFuture&lt;List&lt;Double&gt;&gt;</code>, not <code>CompletableFuture&lt;Void&gt;</code>! Luckily it's easy to fix with a bit of glue code:

```java
private static <T> CompletableFuture<List<T>> 
                       sequence(List<CompletableFuture<T>> futures) {
    CompletableFuture<Void> allDoneFuture = CompletableFuture.allOf(
                futures.toArray(new CompletableFuture[futures.size()]));
    return allDoneFuture.thenApply(v ->
            futures.stream().
                    map(future -> future.join()).
                    collect(Collectors.<T>toList())
    );
}
```

Watch carefully <code>sequence()</code> argument and return types. The implementation is surprisingly simple, the trick is to use existing <code>allOf()</code> but when <code>allDoneFuture</code> completes (which means all underlying futures are done), simply iterate over all futures and <code>join()</code> (blocking wait) on each. However this call is guaranteed not to block because by now all futures completed! Equipped with such utility method we can finally complete our task:

```java
CompletableFuture<List<Double>> allDone = sequence(relevanceFutures);
CompletableFuture<OptionalDouble> maxRelevance = allDone.thenApply(
    relevances -> relevances.stream()
                            .mapToDouble(Double::valueOf)
                            .max()
);
```

This one is easy - when <code>allDone</code> completes, apply our function that counts maximal relevance in whole set. <code>maxRelevance</code> is still a future. By the time your JVM reaches this line, probably none of the websites are yet downloaded. But we encapsulated business logic on top of futures, stacking them in an event-driven manner. Code remains readable (version without lambda and with ordinary <code>Future</code>s would be at least twice as long) but avoids blocking main thread. Of course <code>allDone</code> can as well be an intermediate step, we can further transform it, not really having the result yet.

<h2>Shortcomings</h2>

<code>CompletableFuture</code> in Java 8 is a huge step forward. From tiny, thin abstraction over asynchronous task to full-blown, functional, feature rich utility. However after few days of playing with it I found few minor disadvantages:

- `CompletableFuture.allOf()` returning `CompletableFuture&lt;Void&gt;` discussed earlier. I think it's fair to say that if I pass a collection of futures and want to wait for all of them, I would also like to extract the results when they arrive easily. It's even worse with <code>CompletableFuture.anyOf()</code></a>. If I am waiting for <i>any</i> of the futures to complete, I can't imagine passing futures of different types, say <code>CompletableFuture&lt;Car&gt;</code> and <code>CompletableFuture&lt;Restaurant&gt;</code>. If I don't care which one completes first, how am I suppose to handle return type? Typically you will pass a collection of homogeneous futures (e.g. <code>CompletableFuture&lt;Car&gt;</code>) and then <code>anyOf()</code> can simply return future of that type (instead of <code>CompletableFuture&lt;Void&gt;</code> again).<br><br>


- Mixing <i>settable</i> and <i>listenable</i> abstractions. In Guava there is <a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/util/concurrent/ListenableFuture.html"><code>ListenableFuture</code></a> and <a href="http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/util/concurrent/SettableFuture.html"><code>SettableFuture</code></a> extending it. <code>ListenableFuture</code> allows registering callbacks while <code>SettableFuture</code> adds possibility to set value of the future (resolve it) from arbitrary thread and context. <code>CompletableFuture</code> is equivalent to <code>SettableFuture</code> but there is no limited version equivalent to <code>ListenableFuture</code>. Why is it a problem? If API returns <code>CompletableFuture</code> and then two threads wait for it to complete (nothing wrong with that), one of these threads can resolve this future and wake up other thread, while it's only the API implementation that should do it. But when API tries to resolve the future later, call to <code>complete()</code> is ignored. It can lead to really nasty bugs which are avoided in Guava by separating these two responsibilities.


- `CompletableFuture` is ignored in JDK. `ExecutorService` was not retrofitted to return <code>CompletableFuture</code>. Literally `CompletableFuture` is not referenced anywhere in JDK. It's a really useful class, backward compatible with <code>Future</code>, but not really promoted in standard library.


- Bloated API (?) Fifty methods in total, most in three variants. 
Splitting <i>settable</i> and <i>listenable</i> (see above) would help. 
Also some methods like <code>runAfterBoth()</code> or `runAfterEither()` 
IMHO do not really belong to any <code>CompletableFuture</code>. 
Is there a difference between `fast.runAfterBoth(predictable, ...)` and 
`predictable.runAfterBoth(fast, ...)`? No, but API favours one or the other. 
Actually I believe `runAfterBoth(fast, predictable, ...)` much better 
expresses my intention.


- `CompletableFuture.getNow(T)` should take `Supplier<>` instead of raw reference. 
In the example below `expensiveAlternative()` is always code, irrespective to 
whether future finished or not:

```java
future.getNow(expensiveAlternative());
```

However we can easily tweak this behaviour (I know, there is a small race 
condition here, but the original <code>getNow()</code> works this way as well):

```java
public static <T> T getNow(
            CompletableFuture<T> future, 
            Supplier<T> valueIfAbsent) throws ExecutionException, InterruptedException {
    if (future.isDone()) {
        return future.get();
    } else {
        return valueIfAbsent.get();
    }
}
```

With this utility method we can avoid calling `expensiveAlternative()` when it's
not needed:

```java
getNow(future, () -> expensiveAlternative());
//or:
getNow(future, this::expensiveAlternative);
```



In overall `CompletableFuture` is a wonderful new tool in our JDK belt. 
Minor API issues and sometimes too verbose syntax due to limited type 
inference shouldn't stop you from using it. At least it's a solid 
foundation for better abstractions and more robust code.



