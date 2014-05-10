<h2>Class CompletableFuture<T> Class</h2>

<pre>public class CompletableFuture<> extends `Object`
implements interface in Future<T>, CompletionStage<></pre>


A `Future` that may be explicitly completed (setting its
 value and status), and may be used as a `CompletionStage`,
 supporting dependent functions and actions that trigger upon its
 completion.

 When two or more threads attempt to
 `complete`, `completeExceptionally`, or `cancel`
 a CompletableFuture, only one of them succeeds.

In addition to these and related methods for directly
 manipulating status and results, CompletableFuture implements
 interface `CompletionStage` with the following policies: 

<ul>
 <li>
Actions supplied for dependent completions of
 <em>non-async</em> methods may be performed by the thread that
 completes the current CompletableFuture, or by any other caller of
 a completion method.
</li>

 <li>
All <em>async</em> methods without an explicit Executor
 argument are performed using the `ForkJoinPool.commonPool()`
 (unless it does not support a parallelism level of at least two, in
 which case, a new Thread is used). To simplify monitoring,
 debugging, and tracking, all generated asynchronous tasks are
 instances of the marker interface 
`CompletableFuture.AsynchronousCompletionTask`. 
</li>

 <li>All CompletionStage methods are implemented independently of
 other public methods, so the behavior of one method is not impacted
 by overrides of others in subclasses.  
</li> 

</ul>

 CompletableFuture also implements `Future` with the following
 policies: 

<ul>

<li>
Since (unlike `FutureTask`) this class has no direct
 control over the computation that causes it to be completed,
 cancellation is treated as just another form of exceptional
 completion.  

Method `cancel` has the same effect as 
`completeExceptionally(new CancellationException())`. 

Method `isCompletedExceptionally()` can be used to determine if a
 CompletableFuture completed in any exceptional fashion.
</li>

<li>
In case of exceptional completion with a CompletionException, methods `get()`
and `get(long, TimeUnit)` throw an `ExecutionException` with the same cause as held in the
corresponding CompletionException.  To simplify usage in most
contexts, this class also defines methods `join()` and getNow(T)` 
that instead throw the CompletionException directly in these cases.
</li> 

</ul>



<h3>Nested Class Summary</h3>

    static interface  CompletableFuture.AsynchronousCompletionTask

A marker interface identifying asynchronous tasks produced by `async` methods.


<h3>Constructor Summary</h3>


    CompletableFuture()

Creates a new incomplete CompletableFuture.


<h3>Method Summary</h3>

<h4>Static Methods</h4>

____
    static CompletableFuture<Void>  allOf(CompletableFuture<?>... cfs)

Returns a new CompletableFuture that is completed when all of the given CompletableFutures complete.
____

    static CompletableFuture<Object>  anyOf(CompletableFuture<?>... cfs)
Returns a new CompletableFuture that is completed when any of the given CompletableFutures complete, with the same result.
___

    static <U> CompletableFuture<U>  completedFuture(U value)

Returns a new CompletableFuture that is already completed with the given value.
____

    static CompletableFuture<Void> runAsync(Runnable runnable)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
ForkJoinPool.commonPool() after it runs the given action.
____

    static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
given executor after it runs the given action.
____

    static <U> CompletableFuture<U>  supplyAsync(Supplier<U> supplier)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
ForkJoinPool.commonPool() with the value obtained by calling the given Supplier.
____

    static <U> CompletableFuture<U>	supplyAsync(Supplier<U> supplier, Executor executor)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
given executor with the value obtained by calling the given Supplier.
____

<h4>Instance Methods</h4>

____

```java
CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, 
                                     Consumer<? super T>          action)
```
Returns a new CompletionStage that, when either this or the
other given stage complete normally, is executed with the
corresponding result as argument to the supplied action.

----

```java
CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, 
                                          Consumer<? super T>          action)
```

Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using this
 stage's default asynchronous execution facility, with the
 corresponding result as argument to the supplied action.

----

```java
CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, 
                                          Consumer<? super T>          action, 
                                          Executor                     executor)
```

Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using the
 supplied executor, with the corresponding result as argument to
 the supplied function.</div>

____

```java
<U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, 
                                       Function<? super T,U> fn)
```
Returns a new CompletionStage that, when either this or the 
other given stage complete normally, is executed with the 
corresponding result as argument to the supplied function.
____
```java
<U> CompletableFuture<U>	applyToEitherAsync(CompletionStage<? extends T> other, 
                                            Function<? super T,U>        fn)
```
Returns a new CompletionStage that, when either this or the other 
given stage complete normally, is executed using this stage's default 
asynchronous execution facility, with the corresponding result as 
argument to the supplied function.
____
```java
<U> CompletableFuture<U>	applyToEitherAsync(CompletionStage<? extends T> other, 
                                            Function<? super T,U> fn, 
                                            Executor executor)
```
Returns a new CompletionStage that, when either this or the other given stage 
complete normally, is executed using the supplied executor, 
with the corresponding result as argument to the supplied function.
____

```java
boolean cancel(boolean mayInterruptIfRunning)
```
If not already completed, completes this CompletableFuture with a 
CancellationException.
____
```java
boolean complete(T value)
```
If not already completed, sets the value returned by get() and related 
methods to the given value.
____
```java
boolean completeExceptionally(Throwable ex)
```
If not already completed, causes invocations of get() and related methods 
to throw the given exception.
____
```java
CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```
Returns a new CompletableFuture that is completed when this CompletableFuture 
completes, with the result of the given function of the exception triggering 
this CompletableFuture's completion when it completes exceptionally; 
otherwise, if this CompletableFuture completes normally, then the returned 
CompletableFuture also completes normally with the same value.
____
```
T	get()
```
Waits if necessary for this future to complete, and then returns its result.
____
```
T	get(long timeout, TimeUnit unit)
```
Waits if necessary for at most the given time for this future to complete, and then returns its result, if available.
____
```
T	getNow(T valueIfAbsent)
```
Returns the result value (or throws any encountered exception) if completed, else returns the given valueIfAbsent.

____
```
int	getNumberOfDependents()
```
Returns the estimated number of CompletableFutures whose completions are awaiting completion of this CompletableFuture.
____
```
<U> CompletableFuture<U>	handle(BiFunction<? super T,Throwable,? extends U> fn)
```
Returns a new CompletionStage that, when this stage completes either normally or exceptionally, is executed with this stage's result and exception as arguments to the supplied function.
____________
```
<U> CompletableFuture<U>	handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
```
Returns a new CompletionStage that, when this stage completes either normally or exceptionally, is executed using this stage's default asynchronous execution facility, with this stage's result and exception as arguments to the supplied function.
____________

```
<U> CompletableFuture<U>	handleAsync(BiFunction<? super T,Throwable,? extends U> fn, Executor executor)
```
Returns a new CompletionStage that, when this stage completes either normally or exceptionally, is executed using the supplied executor, with this stage's result and exception as arguments to the supplied function.
____________

```
boolean	isCancelled()
```
Returns true if this CompletableFuture was cancelled before it completed normally.
____________

```
boolean	isCompletedExceptionally()
```
Returns true if this CompletableFuture completed exceptionally, in any way.
____________

```
boolean	isDone()
```
Returns true if completed in any fashion: normally, exceptionally, or via cancellation.
____________

```
T	join()
```
Returns the result value when complete, or throws an (unchecked) exception if completed exceptionally.
____________

```
void	obtrudeException(Throwable ex)
```
Forcibly causes subsequent invocations of method get() and related methods to throw the given exception, whether or not already completed.
____________

```
void	obtrudeValue(T value)
```
Forcibly sets or resets the value subsequently returned by method get() and related methods, whether or not already completed.
____________

```
CompletableFuture<Void>	runAfterBoth(CompletionStage<?> other, Runnable action)
```
Returns a new CompletionStage that, when this and the other given stage both complete normally, executes the given action.
____________

```
CompletableFuture<Void>	runAfterBothAsync(CompletionStage<?> other, Runnable action)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, executes the given action using this stage's default asynchronous execution facility.
____________

```
CompletableFuture<Void>	runAfterBothAsync(CompletionStage<?> other, Runnable action, Executor executor)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, executes the given action using the supplied executor See the CompletionStage documentation for rules covering exceptional completion.
____________

```
CompletableFuture<Void>	runAfterEither(CompletionStage<?> other, Runnable action)
```
Returns a new CompletionStage that, when either this or the other given stage complete normally, executes the given action.
____________

```
CompletableFuture<Void>	runAfterEitherAsync(CompletionStage<?> other, Runnable action)
```
Returns a new CompletionStage that, when either this or the other given stage complete normally, executes the given action using this stage's default asynchronous execution facility.
____________


CompletableFuture<Void>	runAfterEitherAsync(CompletionStage<?> other, Runnable action, Executor executor)

Returns a new CompletionStage that, when either this or the other given stage complete normally, executes the given action using supplied executor.
____________


CompletableFuture<Void>	thenAccept(Consumer<? super T> action)

Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied action.
____________


CompletableFuture<Void>	thenAcceptAsync(Consumer<? super T> action)

Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage's result as the argument to the supplied action.
____________


CompletableFuture<Void>	thenAcceptAsync(Consumer<? super T> action, Executor executor)

Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied action.
____________


<U> CompletableFuture<Void>	thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)

Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied action.
____________


<U> CompletableFuture<Void>	thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)

Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using this stage's default asynchronous execution facility, with the two results as arguments to the supplied action.
____________


<U> CompletableFuture<Void>	thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action, Executor executor)

Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using the supplied executor, with the two results as arguments to the supplied function.
____________


<U> CompletableFuture<U>	thenApply(Function<? super T,? extends U> fn)

Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied function.
____________

<U> CompletableFuture<U>	thenApplyAsync(Function<? super T,? extends U> fn)

Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage's result as the argument to the supplied function.
____________


<U> CompletableFuture<U>	thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)

Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied function.
____________


<U,V> CompletableFuture<V>	thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)

Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied function.
____________


<U,V> CompletableFuture<V>	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)

Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using this stage's default asynchronous execution facility, with the two results as arguments to the supplied function.
____________


<U,V> CompletableFuture<V>	thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)

Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using the supplied executor, with the two results as arguments to the supplied function.
____________


<U> CompletableFuture<U>	thenCompose(Function<? super T,? extends CompletionStage<U>> fn)

Returns a new CompletionStage that, when this stage completes normally, is executed with this stage as the argument to the supplied function.
____________


<U> CompletableFuture<U>	thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn)

Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage as the argument to the supplied function.
____________


<U> CompletableFuture<U>	thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn, Executor executor)

Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied function.
____________


CompletableFuture<Void>	thenRun(Runnable action)

Returns a new CompletionStage that, when this stage completes normally, executes the given action.
____________


CompletableFuture<Void>	thenRunAsync(Runnable action)

Returns a new CompletionStage that, when this stage completes normally, executes the given action using this stage's default asynchronous execution facility.
____________


CompletableFuture<Void>	thenRunAsync(Runnable action, Executor executor)

Returns a new CompletionStage that, when this stage completes normally, executes the given action using the supplied Executor.
____________


CompletableFuture<T>	toCompletableFuture()

Returns this CompletableFuture
____________


String	toString()

Returns a string identifying this CompletableFuture, as well as its completion state.
____________


CompletableFuture<T>	whenComplete(BiConsumer<? super T,? super Throwable> action)

Returns a new CompletionStage with the same result or exception as this stage, and when this stage completes, executes the given action with the result (or null if none) and the exception (or null if none) of this stage.
____________


CompletableFuture<T>	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)

Returns a new CompletionStage with the same result or exception as this stage, and when this stage completes, executes the given action executes the given action using this stage's default asynchronous execution facility, with the result (or null if none) and the exception (or null if none) of this stage as arguments.
____________


CompletableFuture<T>	whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)

Returns a new CompletionStage with the same result or exception as this stage, and when this stage completes, executes using the supplied Executor, the given action with the result (or null if none) and the exception (or null if none) of this stage as arguments.
____________






<table class="memberSummary" border="0" cellpadding="3" cellspacing="0" summary="Method Summary table, listing methods, and an explanation">
<caption><span id="t0" class="activeTableTab"><span>All Methods</span>
<span class="tabEnd">&nbsp;</span>
</span>
<span id="t1" class="tableTab">
<span><a href="javascript:show(1);">Static Methods</a></span><span class="tabEnd">&nbsp;</span></span><span id="t2" class="tableTab"><span><a href="javascript:show(2);">Instance Methods</a></span><span class="tabEnd">&nbsp;</span></span><span id="t4" class="tableTab"><span><a href="javascript:show(8);">Concrete Methods</a></span><span class="tabEnd">&nbsp;</span></span></caption>
<tbody>

<tr>
<th class="colFirst" scope="col">Modifier and Type</th>
<th class="colLast" scope="col">Method and Description</th>
</tr>

<tr id="i3" class="rowColor">
<td class="colFirst"><code>static <a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#allOf-java.util.concurrent.CompletableFuture...-">allOf</a></span>(<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;?&gt;...&nbsp;cfs)</code>

<div class="block">Returns a new CompletableFuture that is completed when all of
 the given CompletableFutures complete.</div>
</td>
</tr>


<tr id="i4" class="altColor">
<td class="colFirst"><code>static <a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Object.html" title="class in java.lang">Object</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#anyOf-java.util.concurrent.CompletableFuture...-">anyOf</a></span>(<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;?&gt;...&nbsp;cfs)</code>
<div class="block">Returns a new CompletableFuture that is completed when any of
 the given CompletableFutures complete, with the same result.</div>
</td>
</tr>
<tr id="i5" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#applyToEither-java.util.concurrent.CompletionStage-java.util.function.Function-">applyToEither</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
             <a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,U&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed with the
 corresponding result as argument to the supplied function.</div>
</td>
</tr>
<tr id="i6" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-">applyToEitherAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                  <a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,U&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using this
 stage's default asynchronous execution facility, with the
 corresponding result as argument to the supplied function.</div>
</td>
</tr>
<tr id="i7" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-java.util.concurrent.Executor-">applyToEitherAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                  <a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,U&gt;&nbsp;fn,
                  <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using the
 supplied executor, with the corresponding result as argument to
 the supplied function.</div>
</td>
</tr>
<tr id="i8" class="altColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#cancel-boolean-">cancel</a></span>(boolean&nbsp;mayInterruptIfRunning)</code>
<div class="block">If not already completed, completes this CompletableFuture with
 a <a href="../../../java/util/concurrent/CancellationException.html" title="class in java.util.concurrent"><code>CancellationException</code></a>.</div>
</td>
</tr>
<tr id="i9" class="rowColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#complete-T-">complete</a></span>(<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;value)</code>
<div class="block">If not already completed, sets the value returned by <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a> and related methods to the given value.</div>
</td>
</tr>
<tr id="i10" class="altColor">
<td class="colFirst"><code>static &lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#completedFuture-U-">completedFuture</a></span>(U&nbsp;value)</code>
<div class="block">Returns a new CompletableFuture that is already completed with
 the given value.</div>
</td>
</tr>
<tr id="i11" class="rowColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#completeExceptionally-java.lang.Throwable-">completeExceptionally</a></span>(<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&nbsp;ex)</code>
<div class="block">If not already completed, causes invocations of <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a>
 and related methods to throw the given exception.</div>
</td>
</tr>
<tr id="i12" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#exceptionally-java.util.function.Function-">exceptionally</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletableFuture that is completed when this
 CompletableFuture completes, with the result of the given
 function of the exception triggering this CompletableFuture's
 completion when it completes exceptionally; otherwise, if this
 CompletableFuture completes normally, then the returned
 CompletableFuture also completes normally with the same value.</div>
</td>
</tr>
<tr id="i13" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#get--">get</a></span>()</code>
<div class="block">Waits if necessary for this future to complete, and then
 returns its result.</div>
</td>
</tr>
<tr id="i14" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#get-long-java.util.concurrent.TimeUnit-">get</a></span>(long&nbsp;timeout,
   <a href="../../../java/util/concurrent/TimeUnit.html" title="enum in java.util.concurrent">TimeUnit</a>&nbsp;unit)</code>
<div class="block">Waits if necessary for at most the given time for this future
 to complete, and then returns its result, if available.</div>
</td>
</tr>
<tr id="i15" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#getNow-T-">getNow</a></span>(<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;valueIfAbsent)</code>
<div class="block">Returns the result value (or throws any encountered exception)
 if completed, else returns the given valueIfAbsent.</div>
</td>
</tr>
<tr id="i16" class="altColor">
<td class="colFirst"><code>int</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#getNumberOfDependents--">getNumberOfDependents</a></span>()</code>
<div class="block">Returns the estimated number of CompletableFutures whose
 completions are awaiting completion of this CompletableFuture.</div>
</td>
</tr>
<tr id="i17" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#handle-java.util.function.BiFunction-">handle</a></span>(<a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends U&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 either normally or exceptionally, is executed with this stage's
 result and exception as arguments to the supplied function.</div>
</td>
</tr>
<tr id="i18" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#handleAsync-java.util.function.BiFunction-">handleAsync</a></span>(<a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends U&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 either normally or exceptionally, is executed using this stage's
 default asynchronous execution facility, with this stage's
 result and exception as arguments to the supplied function.</div>
</td>
</tr>
<tr id="i19" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#handleAsync-java.util.function.BiFunction-java.util.concurrent.Executor-">handleAsync</a></span>(<a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends U&gt;&nbsp;fn,
           <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 either normally or exceptionally, is executed using the
 supplied executor, with this stage's result and exception as
 arguments to the supplied function.</div>
</td>
</tr>
<tr id="i20" class="altColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#isCancelled--">isCancelled</a></span>()</code>
<div class="block">Returns <code>true</code> if this CompletableFuture was cancelled
 before it completed normally.</div>
</td>
</tr>
<tr id="i21" class="rowColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#isCompletedExceptionally--">isCompletedExceptionally</a></span>()</code>
<div class="block">Returns <code>true</code> if this CompletableFuture completed
 exceptionally, in any way.</div>
</td>
</tr>
<tr id="i22" class="altColor">
<td class="colFirst"><code>boolean</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#isDone--">isDone</a></span>()</code>
<div class="block">Returns <code>true</code> if completed in any fashion: normally,
 exceptionally, or via cancellation.</div>
</td>
</tr>
<tr id="i23" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#join--">join</a></span>()</code>
<div class="block">Returns the result value when complete, or throws an
 (unchecked) exception if completed exceptionally.</div>
</td>
</tr>
<tr id="i24" class="altColor">
<td class="colFirst"><code>void</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#obtrudeException-java.lang.Throwable-">obtrudeException</a></span>(<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&nbsp;ex)</code>
<div class="block">Forcibly causes subsequent invocations of method <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a>
 and related methods to throw the given exception, whether or
 not already completed.</div>
</td>
</tr>
<tr id="i25" class="rowColor">
<td class="colFirst"><code>void</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#obtrudeValue-T-">obtrudeValue</a></span>(<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;value)</code>
<div class="block">Forcibly sets or resets the value subsequently returned by
 method <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a> and related methods, whether or not
 already completed.</div>
</td>
</tr>
<tr id="i26" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAfterBoth-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterBoth</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
            <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage both complete normally, executes the given action.</div>
</td>
</tr>
<tr id="i27" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterBothAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                 <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, executes the given action using
 this stage's default asynchronous execution facility.</div>
</td>
</tr>
<tr id="i28" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">runAfterBothAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                 <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action,
                 <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, executes the given action using
 the supplied executor

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
</td>
</tr>
<tr id="i29" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAfterEither-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterEither</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
              <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, executes the given action.</div>
</td>
</tr>
<tr id="i30" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterEitherAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                   <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, executes the given action
 using this stage's default asynchronous execution facility.</div>
</td>
</tr>
<tr id="i31" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">runAfterEitherAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                   <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action,
                   <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, executes the given action
 using supplied executor.</div>
</td>
</tr>
<tr id="i32" class="altColor">
<td class="colFirst"><code>static <a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAsync-java.lang.Runnable-">runAsync</a></span>(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;runnable)</code>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the <a href="../../../java/util/concurrent/ForkJoinPool.html#commonPool--"><code>ForkJoinPool.commonPool()</code></a> after
 it runs the given action.</div>
</td>
</tr>
<tr id="i33" class="rowColor">
<td class="colFirst"><code>static <a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#runAsync-java.lang.Runnable-java.util.concurrent.Executor-">runAsync</a></span>(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;runnable,
        <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the given executor after it runs the given
 action.</div>
</td>
</tr>
<tr id="i34" class="altColor">
<td class="colFirst"><code>static &lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#supplyAsync-java.util.function.Supplier-">supplyAsync</a></span>(<a href="../../../java/util/function/Supplier.html" title="interface in java.util.function">Supplier</a>&lt;U&gt;&nbsp;supplier)</code>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the <a href="../../../java/util/concurrent/ForkJoinPool.html#commonPool--"><code>ForkJoinPool.commonPool()</code></a> with
 the value obtained by calling the given Supplier.</div>
</td>
</tr>
<tr id="i35" class="rowColor">
<td class="colFirst"><code>static &lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#supplyAsync-java.util.function.Supplier-java.util.concurrent.Executor-">supplyAsync</a></span>(<a href="../../../java/util/function/Supplier.html" title="interface in java.util.function">Supplier</a>&lt;U&gt;&nbsp;supplier,
           <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the given executor with the value obtained
 by calling the given Supplier.</div>
</td>
</tr>
<tr id="i36" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenAccept-java.util.function.Consumer-">thenAccept</a></span>(<a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed with this stage's result as the argument
 to the supplied action.</div>
</td>
</tr>
<tr id="i37" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenAcceptAsync-java.util.function.Consumer-">thenAcceptAsync</a></span>(<a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using this stage's default asynchronous
 execution facility, with this stage's result as the argument to
 the supplied action.</div>
</td>
</tr>
<tr id="i38" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenAcceptAsync-java.util.function.Consumer-java.util.concurrent.Executor-">thenAcceptAsync</a></span>(<a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action,
               <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using the supplied Executor, with this
 stage's result as the argument to the supplied action.</div>
</td>
</tr>
<tr id="i39" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenAcceptBoth-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">thenAcceptBoth</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
              <a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U&gt;&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage both complete normally, is executed with the two
 results as arguments to the supplied action.</div>
</td>
</tr>
<tr id="i40" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">thenAcceptBothAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                   <a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U&gt;&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using this stage's
 default asynchronous execution facility, with the two results
 as arguments to the supplied action.</div>
</td>
</tr>
<tr id="i41" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-java.util.concurrent.Executor-">thenAcceptBothAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                   <a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U&gt;&nbsp;action,
                   <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using the supplied
 executor, with the two results as arguments to the supplied
 function.</div>
</td>
</tr>
<tr id="i42" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenApply-java.util.function.Function-">thenApply</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends U&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed with this stage's result as the argument
 to the supplied function.</div>
</td>
</tr>
<tr id="i43" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenApplyAsync-java.util.function.Function-">thenApplyAsync</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends U&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using this stage's default asynchronous
 execution facility, with this stage's result as the argument to
 the supplied function.</div>
</td>
</tr>
<tr id="i44" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenApplyAsync-java.util.function.Function-java.util.concurrent.Executor-">thenApplyAsync</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends U&gt;&nbsp;fn,
              <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using the supplied Executor, with this
 stage's result as the argument to the supplied function.</div>
</td>
</tr>
<tr id="i45" class="rowColor">
<td class="colFirst"><code>&lt;U,V&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;V&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenCombine-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">thenCombine</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
           <a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U,? extends V&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage both complete normally, is executed with the two
 results as arguments to the supplied function.</div>
</td>
</tr>
<tr id="i46" class="altColor">
<td class="colFirst"><code>&lt;U,V&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;V&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">thenCombineAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                <a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U,? extends V&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using this stage's
 default asynchronous execution facility, with the two results
 as arguments to the supplied function.</div>
</td>
</tr>
<tr id="i47" class="rowColor">
<td class="colFirst"><code>&lt;U,V&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;V&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-java.util.concurrent.Executor-">thenCombineAsync</a></span>(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                <a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U,? extends V&gt;&nbsp;fn,
                <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using the supplied
 executor, with the two results as arguments to the supplied
 function.</div>
</td>
</tr>
<tr id="i48" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenCompose-java.util.function.Function-">thenCompose</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;U&gt;&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed with this stage as the argument
 to the supplied function.</div>
</td>
</tr>
<tr id="i49" class="rowColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenComposeAsync-java.util.function.Function-">thenComposeAsync</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;U&gt;&gt;&nbsp;fn)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using this stage's default asynchronous
 execution facility, with this stage as the argument to the
 supplied function.</div>
</td>
</tr>
<tr id="i50" class="altColor">
<td class="colFirst"><code>&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenComposeAsync-java.util.function.Function-java.util.concurrent.Executor-">thenComposeAsync</a></span>(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;U&gt;&gt;&nbsp;fn,
                <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using the supplied Executor, with this
 stage's result as the argument to the supplied function.</div>
</td>
</tr>
<tr id="i51" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenRun-java.lang.Runnable-">thenRun</a></span>(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, executes the given action.</div>
</td>
</tr>
<tr id="i52" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenRunAsync-java.lang.Runnable-">thenRunAsync</a></span>(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, executes the given action using this stage's default
 asynchronous execution facility.</div>
</td>
</tr>
<tr id="i53" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#thenRunAsync-java.lang.Runnable-java.util.concurrent.Executor-">thenRunAsync</a></span>(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action,
            <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, executes the given action using the supplied Executor.</div>
</td>
</tr>
<tr id="i54" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#toCompletableFuture--">toCompletableFuture</a></span>()</code>
<div class="block">Returns this CompletableFuture</div>
</td>
</tr>
<tr id="i55" class="rowColor">
<td class="colFirst"><code><a href="../../../java/lang/String.html" title="class in java.lang">String</a></code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#toString--">toString</a></span>()</code>
<div class="block">Returns a string identifying this CompletableFuture, as well as
 its completion state.</div>
</td>
</tr>
<tr id="i56" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#whenComplete-java.util.function.BiConsumer-">whenComplete</a></span>(<a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super <a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&gt;&nbsp;action)</code>
<div class="block">Returns a new CompletionStage with the same result or exception
 as this stage, and when this stage completes, executes the
 given action with the result (or <code>null</code> if none) and the
 exception (or <code>null</code> if none) of this stage.</div>
</td>
</tr>
<tr id="i57" class="rowColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#whenCompleteAsync-java.util.function.BiConsumer-">whenCompleteAsync</a></span>(<a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super <a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&gt;&nbsp;action)</code>
<div class="block">Returns a new CompletionStage with the same result or exception
 as this stage, and when this stage completes, executes the
 given action executes the given action using this stage's
 default asynchronous execution facility, with the result (or
 <code>null</code> if none) and the exception (or <code>null</code> if
 none) of this stage as arguments.</div>
</td>
</tr>
<tr id="i58" class="altColor">
<td class="colFirst"><code><a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></td>
<td class="colLast"><code><span class="memberNameLink"><a href="../../../java/util/concurrent/CompletableFuture.html#whenCompleteAsync-java.util.function.BiConsumer-java.util.concurrent.Executor-">whenCompleteAsync</a></span>(<a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super <a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&gt;&nbsp;action,
                 <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</code>
<div class="block">Returns a new CompletionStage with the same result or exception
 as this stage, and when this stage completes, executes using
 the supplied Executor, the given action with the result (or
 <code>null</code> if none) and the exception (or <code>null</code> if
 none) of this stage as arguments.</div>
</td>
</tr>
</tbody></table>




<h3>Methods inherited from class&nbsp;java.lang.Object</h3>

<code>
<a href="../../../java/lang/Object.html#clone--">clone</a>, 
<a href="../../../java/lang/Object.html#equals-java.lang.Object-">equals</a>, 
<a href="../../../java/lang/Object.html#finalize--">finalize</a>,
<a href="../../../java/lang/Object.html#getClass--">getClass</a>, 
<a href="../../../java/lang/Object.html#hashCode--">hashCode</a>, 
<a href="../../../java/lang/Object.html#notify--">notify</a>, 
<a href="../../../java/lang/Object.html#notifyAll--">notifyAll</a>, 
<a href="../../../java/lang/Object.html#wait--">wait</a>, 
<a href="../../../java/lang/Object.html#wait-long-">wait</a>, 
<a href="../../../java/lang/Object.html#wait-long-int-">wait</a>
</code>


<!-- ========= CONSTRUCTOR DETAIL ======== -->


<h3>Constructor Detail</h3>

<h4>CompletableFuture</h4>

<pre>public&nbsp;CompletableFuture()</pre>


<!-- ============ METHOD DETAIL ========== -->


<h3>Method Detail</h3>


<a name="supplyAsync-java.util.function.Supplier-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>supplyAsync</h4>
<pre>public static&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;supplyAsync(<a href="../../../java/util/function/Supplier.html" title="interface in java.util.function">Supplier</a>&lt;U&gt;&nbsp;supplier)</pre>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the <a href="../../../java/util/concurrent/ForkJoinPool.html#commonPool--"><code>ForkJoinPool.commonPool()</code></a> with
 the value obtained by calling the given Supplier.</div>
<dl>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>supplier</code> - a function returning the value to be used
 to complete the returned CompletableFuture</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletableFuture</dd>
</dl>



<ul class="blockList">
<li class="blockList">
<h4>supplyAsync</h4>
<pre>public static&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;supplyAsync(<a href="../../../java/util/function/Supplier.html" title="interface in java.util.function">Supplier</a>&lt;U&gt;&nbsp;supplier,
                                                   <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the given executor with the value obtained
 by calling the given Supplier.</div>
<dl>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>supplier</code> - a function returning the value to be used
 to complete the returned CompletableFuture</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletableFuture</dd>
</dl>
</li>
</ul>
<a name="runAsync-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAsync</h4>
<pre>public static&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAsync(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;runnable)</pre>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the <a href="../../../java/util/concurrent/ForkJoinPool.html#commonPool--"><code>ForkJoinPool.commonPool()</code></a> after
 it runs the given action.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>runnable</code> - the action to run before completing the
 returned CompletableFuture</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletableFuture</dd>
</dl>
</li>
</ul>
<a name="runAsync-java.lang.Runnable-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAsync</h4>
<pre>public static&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAsync(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;runnable,
                                               <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block">Returns a new CompletableFuture that is asynchronously completed
 by a task running in the given executor after it runs the given
 action.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>runnable</code> - the action to run before completing the
 returned CompletableFuture</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletableFuture</dd>
</dl>
</li>
</ul>
<a name="completedFuture-java.lang.Object-">
<!--   -->
</a><a name="completedFuture-U-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>completedFuture</h4>
<pre>public static&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;completedFuture(U&nbsp;value)</pre>
<div class="block">Returns a new CompletableFuture that is already completed with
 the given value.</div>
<dl>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the value</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>value</code> - the value</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the completed CompletableFuture</dd>
</dl>
</li>
</ul>
<a name="isDone--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>isDone</h4>
<pre>public&nbsp;boolean&nbsp;isDone()</pre>
<div class="block">Returns <code>true</code> if completed in any fashion: normally,
 exceptionally, or via cancellation.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/Future.html#isDone--">isDone</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/Future.html" title="interface in java.util.concurrent">Future</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd><code>true</code> if completed</dd>
</dl>
</li>
</ul>
<a name="get--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>get</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;get()
      throws <a href="../../../java/lang/InterruptedException.html" title="class in java.lang">InterruptedException</a>,
             <a href="../../../java/util/concurrent/ExecutionException.html" title="class in java.util.concurrent">ExecutionException</a></pre>
<div class="block">Waits if necessary for this future to complete, and then
 returns its result.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/Future.html#get--">get</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/Future.html" title="interface in java.util.concurrent">Future</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the result value</dd>
<dt><span class="throwsLabel">Throws:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CancellationException.html" title="class in java.util.concurrent">CancellationException</a></code> - if this future was cancelled</dd>
<dd><code><a href="../../../java/util/concurrent/ExecutionException.html" title="class in java.util.concurrent">ExecutionException</a></code> - if this future completed exceptionally</dd>
<dd><code><a href="../../../java/lang/InterruptedException.html" title="class in java.lang">InterruptedException</a></code> - if the current thread was interrupted
 while waiting</dd>
</dl>
</li>
</ul>
<a name="get-long-java.util.concurrent.TimeUnit-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>get</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;get(long&nbsp;timeout,
             <a href="../../../java/util/concurrent/TimeUnit.html" title="enum in java.util.concurrent">TimeUnit</a>&nbsp;unit)
      throws <a href="../../../java/lang/InterruptedException.html" title="class in java.lang">InterruptedException</a>,
             <a href="../../../java/util/concurrent/ExecutionException.html" title="class in java.util.concurrent">ExecutionException</a>,
             <a href="../../../java/util/concurrent/TimeoutException.html" title="class in java.util.concurrent">TimeoutException</a></pre>
<div class="block">Waits if necessary for at most the given time for this future
 to complete, and then returns its result, if available.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/Future.html#get-long-java.util.concurrent.TimeUnit-">get</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/Future.html" title="interface in java.util.concurrent">Future</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>timeout</code> - the maximum time to wait</dd>
<dd><code>unit</code> - the time unit of the timeout argument</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the result value</dd>
<dt><span class="throwsLabel">Throws:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CancellationException.html" title="class in java.util.concurrent">CancellationException</a></code> - if this future was cancelled</dd>
<dd><code><a href="../../../java/util/concurrent/ExecutionException.html" title="class in java.util.concurrent">ExecutionException</a></code> - if this future completed exceptionally</dd>
<dd><code><a href="../../../java/lang/InterruptedException.html" title="class in java.lang">InterruptedException</a></code> - if the current thread was interrupted
 while waiting</dd>
<dd><code><a href="../../../java/util/concurrent/TimeoutException.html" title="class in java.util.concurrent">TimeoutException</a></code> - if the wait timed out</dd>
</dl>
</li>
</ul>
<a name="join--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>join</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;join()</pre>
<div class="block">Returns the result value when complete, or throws an
 (unchecked) exception if completed exceptionally. To better
 conform with the use of common functional forms, if a
 computation involved in the completion of this
 CompletableFuture threw an exception, this method throws an
 (unchecked) <a href="../../../java/util/concurrent/CompletionException.html" title="class in java.util.concurrent"><code>CompletionException</code></a> with the underlying
 exception as its cause.</div>
<dl>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the result value</dd>
<dt><span class="throwsLabel">Throws:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CancellationException.html" title="class in java.util.concurrent">CancellationException</a></code> - if the computation was cancelled</dd>
<dd><code><a href="../../../java/util/concurrent/CompletionException.html" title="class in java.util.concurrent">CompletionException</a></code> - if this future completed
 exceptionally or a completion computation threw an exception</dd>
</dl>
</li>
</ul>
<a name="getNow-java.lang.Object-">
<!--   -->
</a><a name="getNow-T-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>getNow</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;getNow(<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;valueIfAbsent)</pre>
<div class="block">Returns the result value (or throws any encountered exception)
 if completed, else returns the given valueIfAbsent.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>valueIfAbsent</code> - the value to return if not completed</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the result value, if completed, else the given valueIfAbsent</dd>
<dt><span class="throwsLabel">Throws:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CancellationException.html" title="class in java.util.concurrent">CancellationException</a></code> - if the computation was cancelled</dd>
<dd><code><a href="../../../java/util/concurrent/CompletionException.html" title="class in java.util.concurrent">CompletionException</a></code> - if this future completed
 exceptionally or a completion computation threw an exception</dd>
</dl>
</li>
</ul>
<a name="complete-java.lang.Object-">
<!--   -->
</a><a name="complete-T-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>complete</h4>
<pre>public&nbsp;boolean&nbsp;complete(<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;value)</pre>
<div class="block">If not already completed, sets the value returned by <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a> and related methods to the given value.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>value</code> - the result value</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd><code>true</code> if this invocation caused this CompletableFuture
 to transition to a completed state, else <code>false</code></dd>
</dl>
</li>
</ul>
<a name="completeExceptionally-java.lang.Throwable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>completeExceptionally</h4>
<pre>public&nbsp;boolean&nbsp;completeExceptionally(<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&nbsp;ex)</pre>
<div class="block">If not already completed, causes invocations of <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a>
 and related methods to throw the given exception.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>ex</code> - the exception</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd><code>true</code> if this invocation caused this CompletableFuture
 to transition to a completed state, else <code>false</code></dd>
</dl>
</li>
</ul>
<a name="thenApply-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenApply</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;thenApply(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends U&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenApply-java.util.function.Function-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed with this stage's result as the argument
 to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenApply-java.util.function.Function-">thenApply</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenApplyAsync-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenApplyAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;thenApplyAsync(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends U&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenApplyAsync-java.util.function.Function-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using this stage's default asynchronous
 execution facility, with this stage's result as the argument to
 the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenApplyAsync-java.util.function.Function-">thenApplyAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenApplyAsync-java.util.function.Function-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenApplyAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;thenApplyAsync(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends U&gt;&nbsp;fn,
                                               <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenApplyAsync-java.util.function.Function-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using the supplied Executor, with this
 stage's result as the argument to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenApplyAsync-java.util.function.Function-java.util.concurrent.Executor-">thenApplyAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenAccept-java.util.function.Consumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenAccept</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenAccept(<a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenAccept-java.util.function.Consumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed with this stage's result as the argument
 to the supplied action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenAccept-java.util.function.Consumer-">thenAccept</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenAcceptAsync-java.util.function.Consumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenAcceptAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenAcceptAsync(<a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptAsync-java.util.function.Consumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using this stage's default asynchronous
 execution facility, with this stage's result as the argument to
 the supplied action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptAsync-java.util.function.Consumer-">thenAcceptAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenAcceptAsync-java.util.function.Consumer-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenAcceptAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenAcceptAsync(<a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action,
                                               <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptAsync-java.util.function.Consumer-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using the supplied Executor, with this
 stage's result as the argument to the supplied action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptAsync-java.util.function.Consumer-java.util.concurrent.Executor-">thenAcceptAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenRun-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenRun</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenRun(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenRun-java.lang.Runnable-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, executes the given action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenRun-java.lang.Runnable-">thenRun</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenRunAsync-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenRunAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenRunAsync(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenRunAsync-java.lang.Runnable-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, executes the given action using this stage's default
 asynchronous execution facility.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenRunAsync-java.lang.Runnable-">thenRunAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenRunAsync-java.lang.Runnable-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenRunAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenRunAsync(<a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action,
                                            <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenRunAsync-java.lang.Runnable-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, executes the given action using the supplied Executor.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenRunAsync-java.lang.Runnable-java.util.concurrent.Executor-">thenRunAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenCombine-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenCombine</h4>
<pre>public&nbsp;&lt;U,V&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;V&gt;&nbsp;thenCombine(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                                              <a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U,? extends V&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenCombine-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage both complete normally, is executed with the two
 results as arguments to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenCombine-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">thenCombine</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the other CompletionStage's result</dd>
<dd><code>V</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenCombineAsync</h4>
<pre>public&nbsp;&lt;U,V&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;V&gt;&nbsp;thenCombineAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                                                   <a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U,? extends V&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using this stage's
 default asynchronous execution facility, with the two results
 as arguments to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-">thenCombineAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the other CompletionStage's result</dd>
<dd><code>V</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenCombineAsync</h4>
<pre>public&nbsp;&lt;U,V&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;V&gt;&nbsp;thenCombineAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                                                   <a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U,? extends V&gt;&nbsp;fn,
                                                   <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using the supplied
 executor, with the two results as arguments to the supplied
 function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenCombineAsync-java.util.concurrent.CompletionStage-java.util.function.BiFunction-java.util.concurrent.Executor-">thenCombineAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the other CompletionStage's result</dd>
<dd><code>V</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenAcceptBoth-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenAcceptBoth</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenAcceptBoth(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                                                  <a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptBoth-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage both complete normally, is executed with the two
 results as arguments to the supplied action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptBoth-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">thenAcceptBoth</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the other CompletionStage's result</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenAcceptBothAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenAcceptBothAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                                                       <a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using this stage's
 default asynchronous execution facility, with the two results
 as arguments to the supplied action.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-">thenAcceptBothAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the other CompletionStage's result</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenAcceptBothAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;thenAcceptBothAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends U&gt;&nbsp;other,
                                                       <a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super U&gt;&nbsp;action,
                                                       <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, is executed using the supplied
 executor, with the two results as arguments to the supplied
 function.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenAcceptBothAsync-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-java.util.concurrent.Executor-">thenAcceptBothAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the other CompletionStage's result</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="runAfterBoth-java.util.concurrent.CompletionStage-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAfterBoth</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAfterBoth(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                                            <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterBoth-java.util.concurrent.CompletionStage-java.lang.Runnable-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage both complete normally, executes the given action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterBoth-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterBoth</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAfterBothAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAfterBothAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                                                 <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, executes the given action using
 this stage's default asynchronous execution facility.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterBothAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAfterBothAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAfterBothAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                                                 <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action,
                                                 <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this and the other
 given stage complete normally, executes the given action using
 the supplied executor

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterBothAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">runAfterBothAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="applyToEither-java.util.concurrent.CompletionStage-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>applyToEither</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;applyToEither(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                                              <a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,U&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#applyToEither-java.util.concurrent.CompletionStage-java.util.function.Function-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed with the
 corresponding result as argument to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#applyToEither-java.util.concurrent.CompletionStage-java.util.function.Function-">applyToEither</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>applyToEitherAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;applyToEitherAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                                                   <a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,U&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using this
 stage's default asynchronous execution facility, with the
 corresponding result as argument to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-">applyToEitherAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>applyToEitherAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;applyToEitherAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                                                   <a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,U&gt;&nbsp;fn,
                                                   <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using the
 supplied executor, with the corresponding result as argument to
 the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#applyToEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Function-java.util.concurrent.Executor-">applyToEitherAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>fn</code> - the function to use to compute the value of
 the returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="acceptEither-java.util.concurrent.CompletionStage-java.util.function.Consumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>acceptEither</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;acceptEither(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                                            <a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#acceptEither-java.util.concurrent.CompletionStage-java.util.function.Consumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed with the
 corresponding result as argument to the supplied action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#acceptEither-java.util.concurrent.CompletionStage-java.util.function.Consumer-">acceptEither</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="acceptEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Consumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>acceptEitherAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;acceptEitherAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                                                 <a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#acceptEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Consumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using this
 stage's default asynchronous execution facility, with the
 corresponding result as argument to the supplied action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#acceptEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Consumer-">acceptEitherAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="acceptEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Consumer-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>acceptEitherAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;acceptEitherAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;other,
                                                 <a href="../../../java/util/function/Consumer.html" title="interface in java.util.function">Consumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;action,
                                                 <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#acceptEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Consumer-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, is executed using the
 supplied executor, with the corresponding result as argument to
 the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#acceptEitherAsync-java.util.concurrent.CompletionStage-java.util.function.Consumer-java.util.concurrent.Executor-">acceptEitherAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="runAfterEither-java.util.concurrent.CompletionStage-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAfterEither</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAfterEither(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                                              <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterEither-java.util.concurrent.CompletionStage-java.lang.Runnable-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, executes the given action.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterEither-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterEither</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAfterEitherAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAfterEitherAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                                                   <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, executes the given action
 using this stage's default asynchronous execution facility.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-">runAfterEitherAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>runAfterEitherAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;runAfterEitherAsync(<a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;?&gt;&nbsp;other,
                                                   <a href="../../../java/lang/Runnable.html" title="interface in java.lang">Runnable</a>&nbsp;action,
                                                   <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when either this or the
 other given stage complete normally, executes the given action
 using supplied executor.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#runAfterEitherAsync-java.util.concurrent.CompletionStage-java.lang.Runnable-java.util.concurrent.Executor-">runAfterEitherAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>other</code> - the other CompletionStage</dd>
<dd><code>action</code> - the action to perform before completing the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenCompose-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenCompose</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;thenCompose(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;U&gt;&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenCompose-java.util.function.Function-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed with this stage as the argument
 to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenCompose-java.util.function.Function-">thenCompose</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the returned CompletionStage's result</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function returning a new CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenComposeAsync-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenComposeAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;thenComposeAsync(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;U&gt;&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenComposeAsync-java.util.function.Function-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using this stage's default asynchronous
 execution facility, with this stage as the argument to the
 supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenComposeAsync-java.util.function.Function-">thenComposeAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the returned CompletionStage's result</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function returning a new CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the CompletionStage</dd>
</dl>
</li>
</ul>
<a name="thenComposeAsync-java.util.function.Function-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>thenComposeAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;thenComposeAsync(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? extends <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;U&gt;&gt;&nbsp;fn,
                                                 <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#thenComposeAsync-java.util.function.Function-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 normally, is executed using the supplied Executor, with this
 stage's result as the argument to the supplied function.

 See the <a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent"><code>CompletionStage</code></a> documentation for rules
 covering exceptional completion.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#thenComposeAsync-java.util.function.Function-java.util.concurrent.Executor-">thenComposeAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the type of the returned CompletionStage's result</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function returning a new CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the CompletionStage</dd>
</dl>
</li>
</ul>
<a name="whenComplete-java.util.function.BiConsumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>whenComplete</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;whenComplete(<a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super <a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#whenComplete-java.util.function.BiConsumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage with the same result or exception
 as this stage, and when this stage completes, executes the
 given action with the result (or <code>null</code> if none) and the
 exception (or <code>null</code> if none) of this stage.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#whenComplete-java.util.function.BiConsumer-">whenComplete</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="whenCompleteAsync-java.util.function.BiConsumer-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>whenCompleteAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;whenCompleteAsync(<a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super <a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&gt;&nbsp;action)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#whenCompleteAsync-java.util.function.BiConsumer-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage with the same result or exception
 as this stage, and when this stage completes, executes the
 given action executes the given action using this stage's
 default asynchronous execution facility, with the result (or
 <code>null</code> if none) and the exception (or <code>null</code> if
 none) of this stage as arguments.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#whenCompleteAsync-java.util.function.BiConsumer-">whenCompleteAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="whenCompleteAsync-java.util.function.BiConsumer-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>whenCompleteAsync</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;whenCompleteAsync(<a href="../../../java/util/function/BiConsumer.html" title="interface in java.util.function">BiConsumer</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,? super <a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&gt;&nbsp;action,
                                              <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#whenCompleteAsync-java.util.function.BiConsumer-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage with the same result or exception
 as this stage, and when this stage completes, executes using
 the supplied Executor, the given action with the result (or
 <code>null</code> if none) and the exception (or <code>null</code> if
 none) of this stage as arguments.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#whenCompleteAsync-java.util.function.BiConsumer-java.util.concurrent.Executor-">whenCompleteAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>action</code> - the action to perform</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="handle-java.util.function.BiFunction-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>handle</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;handle(<a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends U&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#handle-java.util.function.BiFunction-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 either normally or exceptionally, is executed with this stage's
 result and exception as arguments to the supplied function.
 The given function is invoked with the result (or <code>null</code>
 if none) and the exception (or <code>null</code> if none) of this
 stage when complete as arguments.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#handle-java.util.function.BiFunction-">handle</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="handleAsync-java.util.function.BiFunction-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>handleAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;handleAsync(<a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends U&gt;&nbsp;fn)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#handleAsync-java.util.function.BiFunction-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 either normally or exceptionally, is executed using this stage's
 default asynchronous execution facility, with this stage's
 result and exception as arguments to the supplied function.
 The given function is invoked with the result (or <code>null</code>
 if none) and the exception (or <code>null</code> if none) of this
 stage when complete as arguments.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#handleAsync-java.util.function.BiFunction-">handleAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of the
 returned CompletionStage</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="handleAsync-java.util.function.BiFunction-java.util.concurrent.Executor-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>handleAsync</h4>
<pre>public&nbsp;&lt;U&gt;&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;U&gt;&nbsp;handleAsync(<a href="../../../java/util/function/BiFunction.html" title="interface in java.util.function">BiFunction</a>&lt;? super <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>,<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends U&gt;&nbsp;fn,
                                            <a href="../../../java/util/concurrent/Executor.html" title="interface in java.util.concurrent">Executor</a>&nbsp;executor)</pre>
<div class="block"><span class="descfrmTypeLabel">Description copied from interface:&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html#handleAsync-java.util.function.BiFunction-java.util.concurrent.Executor-">CompletionStage</a></code></span></div>
<div class="block">Returns a new CompletionStage that, when this stage completes
 either normally or exceptionally, is executed using the
 supplied executor, with this stage's result and exception as
 arguments to the supplied function.  The given function is
 invoked with the result (or <code>null</code> if none) and the
 exception (or <code>null</code> if none) of this stage when complete
 as arguments.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#handleAsync-java.util.function.BiFunction-java.util.concurrent.Executor-">handleAsync</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Type Parameters:</span></dt>
<dd><code>U</code> - the function's return type</dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of the
 returned CompletionStage</dd>
<dd><code>executor</code> - the executor to use for asynchronous execution</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletionStage</dd>
</dl>
</li>
</ul>
<a name="toCompletableFuture--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>toCompletableFuture</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;toCompletableFuture()</pre>
<div class="block">Returns this CompletableFuture</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#toCompletableFuture--">toCompletableFuture</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>this CompletableFuture</dd>
</dl>
</li>
</ul>
<a name="exceptionally-java.util.function.Function-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>exceptionally</h4>
<pre>public&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;exceptionally(<a href="../../../java/util/function/Function.html" title="interface in java.util.function">Function</a>&lt;<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>,? extends <a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;&nbsp;fn)</pre>
<div class="block">Returns a new CompletableFuture that is completed when this
 CompletableFuture completes, with the result of the given
 function of the exception triggering this CompletableFuture's
 completion when it completes exceptionally; otherwise, if this
 CompletableFuture completes normally, then the returned
 CompletableFuture also completes normally with the same value.
 Note: More flexible versions of this functionality are
 available using methods <code>whenComplete</code> and <code>handle</code>.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/CompletionStage.html#exceptionally-java.util.function.Function-">exceptionally</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/CompletionStage.html" title="interface in java.util.concurrent">CompletionStage</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>fn</code> - the function to use to compute the value of the
 returned CompletableFuture if this CompletableFuture completed
 exceptionally</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the new CompletableFuture</dd>
</dl>
</li>
</ul>
<a name="allOf-java.util.concurrent.CompletableFuture...-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>allOf</h4>
<pre>public static&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Void.html" title="class in java.lang">Void</a>&gt;&nbsp;allOf(<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;?&gt;...&nbsp;cfs)</pre>
<div class="block">Returns a new CompletableFuture that is completed when all of
 the given CompletableFutures complete.  If any of the given
 CompletableFutures complete exceptionally, then the returned
 CompletableFuture also does so, with a CompletionException
 holding this exception as its cause.  Otherwise, the results,
 if any, of the given CompletableFutures are not reflected in
 the returned CompletableFuture, but may be obtained by
 inspecting them individually. If no CompletableFutures are
 provided, returns a CompletableFuture completed with the value
 <code>null</code>.

 <p>Among the applications of this method is to await completion
 of a set of independent CompletableFutures before continuing a
 program, as in: <code>CompletableFuture.allOf(c1, c2,
 c3).join();</code>.</p></div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>cfs</code> - the CompletableFutures</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>a new CompletableFuture that is completed when all of the
 given CompletableFutures complete</dd>
<dt><span class="throwsLabel">Throws:</span></dt>
<dd><code><a href="../../../java/lang/NullPointerException.html" title="class in java.lang">NullPointerException</a></code> - if the array or any of its elements are
 <code>null</code></dd>
</dl>
</li>
</ul>
<a name="anyOf-java.util.concurrent.CompletableFuture...-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>anyOf</h4>
<pre>public static&nbsp;<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;<a href="../../../java/lang/Object.html" title="class in java.lang">Object</a>&gt;&nbsp;anyOf(<a href="../../../java/util/concurrent/CompletableFuture.html" title="class in java.util.concurrent">CompletableFuture</a>&lt;?&gt;...&nbsp;cfs)</pre>
<div class="block">Returns a new CompletableFuture that is completed when any of
 the given CompletableFutures complete, with the same result.
 Otherwise, if it completed exceptionally, the returned
 CompletableFuture also does so, with a CompletionException
 holding this exception as its cause.  If no CompletableFutures
 are provided, returns an incomplete CompletableFuture.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>cfs</code> - the CompletableFutures</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>a new CompletableFuture that is completed with the
 result or exception of any of the given CompletableFutures when
 one completes</dd>
<dt><span class="throwsLabel">Throws:</span></dt>
<dd><code><a href="../../../java/lang/NullPointerException.html" title="class in java.lang">NullPointerException</a></code> - if the array or any of its elements are
 <code>null</code></dd>
</dl>
</li>
</ul>
<a name="cancel-boolean-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>cancel</h4>
<pre>public&nbsp;boolean&nbsp;cancel(boolean&nbsp;mayInterruptIfRunning)</pre>
<div class="block">If not already completed, completes this CompletableFuture with
 a <a href="../../../java/util/concurrent/CancellationException.html" title="class in java.util.concurrent"><code>CancellationException</code></a>. Dependent CompletableFutures
 that have not already completed will also complete
 exceptionally, with a <a href="../../../java/util/concurrent/CompletionException.html" title="class in java.util.concurrent"><code>CompletionException</code></a> caused by
 this <code>CancellationException</code>.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/Future.html#cancel-boolean-">cancel</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/Future.html" title="interface in java.util.concurrent">Future</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>mayInterruptIfRunning</code> - this value has no effect in this
 implementation because interrupts are not used to control
 processing.</dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd><code>true</code> if this task is now cancelled</dd>
</dl>
</li>
</ul>
<a name="isCancelled--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>isCancelled</h4>
<pre>public&nbsp;boolean&nbsp;isCancelled()</pre>
<div class="block">Returns <code>true</code> if this CompletableFuture was cancelled
 before it completed normally.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Specified by:</span></dt>
<dd><code><a href="../../../java/util/concurrent/Future.html#isCancelled--">isCancelled</a></code>&nbsp;in interface&nbsp;<code><a href="../../../java/util/concurrent/Future.html" title="interface in java.util.concurrent">Future</a>&lt;<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&gt;</code></dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd><code>true</code> if this CompletableFuture was cancelled
 before it completed normally</dd>
</dl>
</li>
</ul>
<a name="isCompletedExceptionally--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>isCompletedExceptionally</h4>
<pre>public&nbsp;boolean&nbsp;isCompletedExceptionally()</pre>
<div class="block">Returns <code>true</code> if this CompletableFuture completed
 exceptionally, in any way. Possible causes include
 cancellation, explicit invocation of <code>completeExceptionally</code>, and abrupt termination of a
 CompletionStage action.</div>
<dl>
<dt><span class="returnLabel">Returns:</span></dt>
<dd><code>true</code> if this CompletableFuture completed
 exceptionally</dd>
</dl>
</li>
</ul>
<a name="obtrudeValue-java.lang.Object-">
<!--   -->
</a><a name="obtrudeValue-T-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>obtrudeValue</h4>
<pre>public&nbsp;void&nbsp;obtrudeValue(<a href="../../../java/util/concurrent/CompletableFuture.html" title="type parameter in CompletableFuture">T</a>&nbsp;value)</pre>
<div class="block">Forcibly sets or resets the value subsequently returned by
 method <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a> and related methods, whether or not
 already completed. This method is designed for use only in
 error recovery actions, and even in such situations may result
 in ongoing dependent completions using established versus
 overwritten outcomes.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>value</code> - the completion value</dd>
</dl>
</li>
</ul>
<a name="obtrudeException-java.lang.Throwable-">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>obtrudeException</h4>
<pre>public&nbsp;void&nbsp;obtrudeException(<a href="../../../java/lang/Throwable.html" title="class in java.lang">Throwable</a>&nbsp;ex)</pre>
<div class="block">Forcibly causes subsequent invocations of method <a href="../../../java/util/concurrent/CompletableFuture.html#get--"><code>get()</code></a>
 and related methods to throw the given exception, whether or
 not already completed. This method is designed for use only in
 recovery actions, and even in such situations may result in
 ongoing dependent completions using established versus
 overwritten outcomes.</div>
<dl>
<dt><span class="paramLabel">Parameters:</span></dt>
<dd><code>ex</code> - the exception</dd>
</dl>
</li>
</ul>
<a name="getNumberOfDependents--">
<!--   -->
</a>
<ul class="blockList">
<li class="blockList">
<h4>getNumberOfDependents</h4>
<pre>public&nbsp;int&nbsp;getNumberOfDependents()</pre>
<div class="block">Returns the estimated number of CompletableFutures whose
 completions are awaiting completion of this CompletableFuture.
 This method is designed for use in monitoring system state, not
 for synchronization control.</div>
<dl>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>the number of dependent CompletableFutures</dd>
</dl>
</li>
</ul>
<a name="toString--">
<!--   -->
</a>
<ul class="blockListLast">
<li class="blockList">
<h4>toString</h4>
<pre>public&nbsp;<a href="../../../java/lang/String.html" title="class in java.lang">String</a>&nbsp;toString()</pre>
<div class="block">Returns a string identifying this CompletableFuture, as well as
 its completion state.  The state, in brackets, contains the
 String <code>"Completed Normally"</code> or the String <code>"Completed Exceptionally"</code>, or the String <code>"Not
 completed"</code> followed by the number of CompletableFutures
 dependent upon its completion, if any.</div>
<dl>
<dt><span class="overrideSpecifyLabel">Overrides:</span></dt>
<dd><code><a href="../../../java/lang/Object.html#toString--">toString</a></code>&nbsp;in class&nbsp;<code><a href="../../../java/lang/Object.html" title="class in java.lang">Object</a></code></dd>
<dt><span class="returnLabel">Returns:</span></dt>
<dd>a string identifying this CompletableFuture, as well as its state</dd>
</dl>
</li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>
