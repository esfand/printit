<h1>Java Future basics </h1>

_Futures_ are very important abstraction, even more these day than ever due
to growing demand for asynchronous, event-driven, parallel and scalable
systems. In the first article we'll discover most basic
`ava.util.concurrent.Future<T>` interface. Later on we will jump into other
frameworks, libraries or even languages. `Future<T>` is pretty limited, 
but essential to understand, future parts.

In a single-threaded application when you call a method it returns only when 
the computations are done `IOUtils.toString()` comes from
_Apache Commons IO_):

```java
public String downloadContents(URL url) throws IOException {
    try(InputStream input = url.openStream()) {
        return IOUtils.toString(input, StandardCharsets.UTF_8);
    }
}
 
//...
 
final String contents = downloadContents(new URL("http://www.example.com"));
```

`downloadContents()` looks harmless, but it can take even arbitrary long time to
complete. Moreover in order to reduce latency you might want to do other,
independent processing in the meantime, while waiting for results. In the old
days you would start a new `Thread` and somehow wait for results (shared
memory, locks, dreadful `wait()`/`notify()` pair, etc.) With `Future<T>' it's much
more pleasant:

```java
public static Future<String> startDownloading(URL url) {
    //...
}
 
final Future<String> contentsFuture = startDownloading(
                                          new URL("http://www.example.com"));

//other computations

final String contents = contentsFuture.get();
```

We will implement `startDownloading()` soon. For now it's important that you 
understand the principles. `startDownloading()` does **not** block, waiting for 
external website. Instead it returns immediately, returning a lightweight `Future<String>` object. This object is a <i>promise</i> that <code>String</code> will be available in the future. Don't know when, but keep this reference and once it's there, you'll be able to retrieve it using <code>Future.get()</code>. In other words <code>Future</code> is a proxy or a wrapper around an object that is not yet there. Once the asynchronous computation is done, you can extract it. So what API does <code>Future</code> provide?

<code>Future.get()</code> is the most important method. It blocks and waits until promised result is available (<i>resolved</i>). So if we really need that <code>String</code>, just call <code>get()</code> and wait. There is an overloaded version that accepts timeout so you won't wait forever if something goes wild. <code>TimeoutException</code> is thrown if waiting for too long.

In some use cases you might want to peek on the <code>Future</code> and continue if result is not yet available. This is possible with <code>isDone()</code>. Imagine a situation where your user waits for some asynchronous computation and you'd like to let him know that we are still waiting and do some computation in the meantime:

```java
final Future<String> contentsFuture = startDownloading(
                                          new URL("http://www.example.com"));
while (!contentsFuture.isDone()) {
    askUserToWait();
    doSomeComputationInTheMeantime();
}

contentsFuture.get();
```

The last call to <code>contentsFuture.get()</code> is guaranteed to return immediately and not block because <code>Future.isDone()</code> returned <code>true</code>. If you follow the pattern above make sure you are not busy waiting, calling <code>isDone()</code> millions of time per second.

Cancelling futures is the last aspect we have not covered yet. Imagine you started some asynchronous job and you can only wait for it given amount of time. If it's not there after, say, 2 seconds, we give up and either propagate error or work around it. However if you are a good citizen, you should somehow tell this future object: I no longer need you, forget about it. You save processing resources by not running obsolete tasks. The syntax is simple:

```java

contentsFuture.cancel(true);    //meh...

```

We all love cryptic, boolean parameters, aren't we? Cancelling comes in two flavours. By passing <code>false</code> to <code>mayInterruptIfRunning</code> parameter we only cancel tasks that didn't yet started, when the <code>Future</code> represents results of computation that did not even began. But if our <code>Callable.call()</code> is already in the middle, we let it finish. However if we pass <code>true</code>, <code>Future.cancel()</code> will be more aggressive, trying to interrupt already running jobs as well. How? Think about all these methods that throw infamous <code>InterruptedException</code>, namely <code>Thread.sleep()</code>, <code>Object.wait()</code>, <code>Condition.await()</code>,  and many others (including <code>Future.get()</code>). If you are blocking on any of such methods and someone decided to cancel your <code>Callable</code>, they will actually throw <code>InterruptedException</code>, signalling that someone is trying to interrupt currently running task.

<hr/>

So we now understand what <code>Future&lt;T&gt;</code> is - a place-holder for something, that you will get in the future. It's like keys to a car that was not yet manufactured. But how do you actually obtain an instance of <code>Future&lt;T&gt;</code> in your application? Two most common sources are thread pools and asynchronous methods (backed by thread pools for you). Thus our <code>startDownloading()</code> method can be rewritten to:

```java

private final ExecutorService pool = Executors.newFixedThreadPool(10);
 
public Future<String> startDownloading(final URL url) throws IOException {
    return pool.submit(new Callable<String>() {
        @Override
        public String call() throws Exception {
            try (InputStream input = url.openStream()) {
                return IOUtils.toString(input, StandardCharsets.UTF_8);
            }
        }
    });
}

```

A lot of syntax boilerplate, but the basic idea is simple: wrap long-running computations in <code>Callable&lt;String&gt;</code> and <code>submit()</code> them to a thread pool of 10 threads. Submitting returns some implementation of <code>Future&lt;String&gt;</code>, most likely somehow linked to your task and thread pool. Obviously your task is not executed immediately. Instead it is placed in a queue which is later (maybe even much later) polled by thread from a pool. Now it should be clear what these two flavours of <code>cancel()</code> mean - you can always cancel task that still resides in that queue. But cancelling already running task is a bit more complex.

Another place where you can meet `Future` is Spring and EJB. For example in
Spring framework you can simply annotate your method with `@Async`:

```java
@Async
public Future<String> startDownloading(final URL url) throws IOException {
    try (InputStream input = url.openStream()) {
        return new AsyncResult<>(
                IOUtils.toString(input, StandardCharsets.UTF_8)
        );
    }
}
```

Notice that we simply wrap our result in <code>AsyncResult</code></a> implementing <code>Future</code>. But the method itself does not deal with thread pool or asynchronous processing. Later on Spring will proxy all calls to <code>startDownloading()</code> and run them in a thread pool. The exact same feature is available through <code>@Asynchronous</code> annotation in EJB.

So we learned a lot about `java.util.concurrent.Future`.  Now it's time to admit -
this interface is quite limited, especially when compared to other languages.
More on that later.



