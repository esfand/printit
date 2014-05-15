<h2>Class CompletableFuture<T> Class</h2>

http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html

```java
public class CompletableFuture<> extends Object
                                 implements Future<T>,
                                 CompletionStage<>
```

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

* Actions supplied for dependent completions of
 **non-async** methods may be performed by the thread that
 completes the current CompletableFuture, or by any other caller of
 a completion method.

* All **async** methods without an explicit Executor
 argument are performed using the `ForkJoinPool.commonPool()`
 (unless it does not support a parallelism level of at least two, in
 which case, a new Thread is used). To simplify monitoring,
 debugging, and tracking, all generated asynchronous tasks are
 instances of the marker interface 
`CompletableFuture.AsynchronousCompletionTask`. 

* All CompletionStage methods are implemented independently of
 other public methods, so the behavior of one method is not impacted
 by overrides of others in subclasses.  

CompletableFuture also implements `Future` with the following
 policies: 

* Since (unlike `FutureTask`) this class has no direct
 control over the computation that causes it to be completed,
 cancellation is treated as just another form of exceptional
 completion.      
    
 Method `cancel` has the same effect as 
 `completeExceptionally(new CancellationException())`.     
    
 Method `isCompletedExceptionally()` can be used to determine if a
 CompletableFuture completed in any exceptional fashion.

* In case of exceptional completion with a CompletionException, methods `get()`
and `get(long, TimeUnit)` throw an `ExecutionException` with the same cause as held in the
corresponding CompletionException.  To simplify usage in most
contexts, this class also defines methods `join()` and getNow(T)` 
that instead throw the CompletionException directly in these cases.

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

Returns a new CompletableFuture that is completed when all of the given 
CompletableFutures complete.
____

    static CompletableFuture<Object>  anyOf(CompletableFuture<?>... cfs)
    
Returns a new CompletableFuture that is completed when any of the given 
CompletableFutures complete, with the same result.
___

    static <U> CompletableFuture<U>  completedFuture(U value)

Returns a new CompletableFuture that is already completed with the given value.
____

    static CompletableFuture<Void> runAsync(Runnable runnable)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
ForkJoinPool.commonPool() after it runs the given action.
____

    static CompletableFuture<Void> runAsync(Runnable runnable, 
                                            Executor executor)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
given executor after it runs the given action.
____

    static <U> CompletableFuture<U>  supplyAsync(Supplier<U> supplier)

Returns a new CompletableFuture that is asynchronously completed by a task running in the 
ForkJoinPool.commonPool() with the value obtained by calling the given Supplier.
____

    static <U> CompletableFuture<U>  supplyAsync(Supplier<U> supplier, 
                                                 Executor    executor)

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
                                       Function<? super T,U>        fn)
```
Returns a new CompletionStage that, when either this or the 
other given stage complete normally, is executed with the 
corresponding result as argument to the supplied function.
____

```java
<U> CompletableFuture<U>  applyToEitherAsync(CompletionStage<? extends T> other, 
                                             Function<? super T,U>        fn)
```
Returns a new CompletionStage that, when either this or the other 
given stage complete normally, is executed using this stage's default 
asynchronous execution facility, with the corresponding result as 
argument to the supplied function.
____

```java
<U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, 
                                            Function<? super T,U>        fn, 
                                            Executor                     executor)
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
T  get()
```
Waits if necessary for this future to complete, and then returns its result.
____

```
T  get(long timeout, TimeUnit unit)
```
Waits if necessary for at most the given time for this future to complete, 
and then returns its result, if available.
____

```
T  getNow(T valueIfAbsent)
```
Returns the result value (or throws any encountered exception) if completed, 
else returns the given valueIfAbsent.

____

```
int  getNumberOfDependents()
```
Returns the estimated number of CompletableFutures whose completions are 
awaiting completion of this CompletableFuture.
____

```
<U> CompletableFuture<U>  handle(BiFunction<? super T,Throwable,? extends U> fn)
```
Returns a new CompletionStage that, when this stage completes either normally 
or exceptionally, is executed with this stage's result and exception as 
arguments to the supplied function.
____________

```
<U> CompletableFuture<U>  handleAsync(BiFunction<? super T,Throwable,? extends U> fn)
```
Returns a new CompletionStage that, when this stage completes either normally or exceptionally, is executed using this stage's default asynchronous execution facility, with this stage's result and exception as arguments to the supplied function.
____________

```
<U> CompletableFuture<U>  handleAsync(BiFunction<? super T,Throwable,? extends U> fn, 
                                      Executor                                    executor)
```
Returns a new CompletionStage that, when this stage completes either normally or exceptionally, is executed using the supplied executor, with this stage's result and exception as arguments to the supplied function.
____________

```
boolean  isCancelled()
```
Returns true if this CompletableFuture was cancelled before it completed normally.
____________

```
boolean  isCompletedExceptionally()
```
Returns true if this CompletableFuture completed exceptionally, in any way.
____________

```
boolean  isDone()
```
Returns true if completed in any fashion: normally, exceptionally, or via cancellation.
____________

```
T  join()
```
Returns the result value when complete, or throws an (unchecked) exception if completed exceptionally.
____________

```
void  obtrudeException(Throwable ex)
```
Forcibly causes subsequent invocations of method get() and related methods to throw the given exception, whether or not already completed.
____________

```
void  obtrudeValue(T value)
```
Forcibly sets or resets the value subsequently returned by method get() and related methods, whether or not already completed.
____________

```
CompletableFuture<Void>  runAfterBoth(CompletionStage<?> other, 
                                      Runnable           action)
```
Returns a new CompletionStage that, when this and the other given stage 
both complete normally, executes the given action.
____________

```
CompletableFuture<Void>  runAfterBothAsync(CompletionStage<?> other, 
                                           Runnable           action)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, executes the given action using this stage's default asynchronous execution facility.
____________

```
CompletableFuture<Void>  runAfterBothAsync(CompletionStage<?> other, 
                                           Runnable           action, 
                                           Executor           executor)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, executes the given action using the supplied executor See the CompletionStage documentation for rules covering exceptional completion.
____________

```
CompletableFuture<Void>  runAfterEither(CompletionStage<?> other, 
                                        Runnable           action)
```
Returns a new CompletionStage that, when either this or the other given stage complete normally, executes the given action.
____________

```
CompletableFuture<Void>  runAfterEitherAsync(CompletionStage<?> other, 
                                             Runnable           action)
```
Returns a new CompletionStage that, when either this or the other given stage complete normally, executes the given action using this stage's default asynchronous execution facility.
____________

```java
CompletableFuture<Void>  runAfterEitherAsync(CompletionStage<?> other, 
                                             Runnable           action, 
                                             Executor           executor)
```
Returns a new CompletionStage that, when either this or the other given stage complete normally, executes the given action using supplied executor.
____________

```java
CompletableFuture<Void>  thenAccept(Consumer<? super T> action)
```
Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied action.
____________

```
CompletableFuture<Void>  thenAcceptAsync(Consumer<? super T> action)
```
Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage's result as the argument to the supplied action.
____________

```
CompletableFuture<Void>  thenAcceptAsync(Consumer<? super T> action, 
                                         Executor            executor)
```
Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied action.
____________

```
<U> CompletableFuture<Void>  thenAcceptBoth(CompletionStage<? extends U>    other, 
                                            BiConsumer<? super T,? super U> action)
```
Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied action.
____________

```
<U> CompletableFuture<Void>  thenAcceptBothAsync(CompletionStage<? extends U>    other, 
                                                 BiConsumer<? super T,? super U> action)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using this stage's default asynchronous execution facility, with the two results as arguments to the supplied action.
____________

```
<U> CompletableFuture<Void>  thenAcceptBothAsync(CompletionStage<? extends U>    other, 
                                                 BiConsumer<? super T,? super U> action, 
                                                 Executor                        executor)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using the supplied executor, with the two results as arguments to the supplied function.
____________

```
<U> CompletableFuture<U>  thenApply(Function<? super T,? extends U> fn)
```
Returns a new CompletionStage that, when this stage completes normally, is executed with this stage's result as the argument to the supplied function.
____________

```
<U> CompletableFuture<U>  thenApplyAsync(Function<? super T,? extends U> fn)
```
Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage's result as the argument to the supplied function.
____________

```
<U> CompletableFuture<U>  thenApplyAsync(Function<? super T,? extends U> fn, 
                                         Executor                        executor)
```
Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied function.
____________

```java
<U,V> CompletableFuture<V>  thenCombine(CompletionStage<? extends U>                other, 
                                        BiFunction<? super T,? super U,? extends V> fn)
```
Returns a new CompletionStage that, when this and the other given stage both complete normally, is executed with the two results as arguments to the supplied function.
____________

```java
<U,V> CompletableFuture<V>  thenCombineAsync(CompletionStage<? extends U>                other, 
                                             BiFunction<? super T,? super U,? extends V> fn)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using this stage's default asynchronous execution facility, with the two results as arguments to the supplied function.
____________

```java
<U,V> CompletableFuture<V>  thenCombineAsync(CompletionStage<? extends U>                other, 
                                             BiFunction<? super T,? super U,? extends V> fn, 
                                             Executor                                    executor)
```
Returns a new CompletionStage that, when this and the other given stage complete normally, is executed using the supplied executor, with the two results as arguments to the supplied function.
____________

```java
<U> CompletableFuture<U>  thenCompose(Function<? super T,? extends CompletionStage<U>> fn)
```
Returns a new CompletionStage that, when this stage completes normally, is executed with this stage as the argument to the supplied function.
____________

```java
<U> CompletableFuture<U>  thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn)
```
Returns a new CompletionStage that, when this stage completes normally, is executed using this stage's default asynchronous execution facility, with this stage as the argument to the supplied function.
____________

```java
<U> CompletableFuture<U>  thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn, 
                                           Executor                                         executor)
```
Returns a new CompletionStage that, when this stage completes normally, is executed using the supplied Executor, with this stage's result as the argument to the supplied function.
____________

```java
CompletableFuture<Void>  thenRun(Runnable action)
```
Returns a new CompletionStage that, when this stage completes normally, executes the given action.
____________

```java
CompletableFuture<Void>  thenRunAsync(Runnable action)
```
Returns a new CompletionStage that, when this stage completes normally, 
executes the given action using this stage's default asynchronous execution facility.
____________

```java
CompletableFuture<Void>  thenRunAsync(Runnable action, 
                                      Executor executor)
```
Returns a new CompletionStage that, when this stage completes normally, 
executes the given action using the supplied Executor.
____________

```java
CompletableFuture<T>  toCompletableFuture()
```
Returns this CompletableFuture
____________

```java
String  toString()
```
Returns a string identifying this CompletableFuture, as well as its completion state.
____________

```java
CompletableFuture<T>  whenComplete(BiConsumer<? super T,? super Throwable> action)
```
Returns a new CompletionStage with the same result or exception as this stage, and when this stage completes, executes the given action with the result (or null if none) and the exception (or null if none) of this stage.
____________

```java
CompletableFuture<T>  whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
```
Returns a new CompletionStage with the same result or exception as this stage, and when this stage completes, executes the given action executes the given action using this stage's default asynchronous execution facility, with the result (or null if none) and the exception (or null if none) of this stage as arguments.
____________

```java
CompletableFuture<T>  whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, 
                                        Executor                                executor)
```
Returns a new CompletionStage with the same result or exception as this stage, and when this stage completes, executes using the supplied Executor, the given action with the result (or null if none) and the exception (or null if none) of this stage as arguments.
____________




