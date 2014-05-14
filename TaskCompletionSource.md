## TaskCompletionSource ##
[Source](http://tutorials.csharp-online.net/TaskCompletionSource)

We've seen how `Task.Run` creates a task that runs a delegate on a 
pooled (or non-pooled) thread.

Another way to create a task is with `TaskCompletionSource`.
`TaskCompletionSource` lets you create a task out of any operation that starts and
finishes some time later. It works by giving you a "slave" task that you manually 
drive—by indicating when the operation finishes or faults. This is ideal for 
I/O- bound work: you get all the benefits of tasks (with their ability to 
propagate return values, exceptions, and continuations) without blocking a thread 
for the duration of the operation.

To use `TaskCompletionSource`, you simply instantiate the class.  It exposes a Task 
property that returns a task upon which you can wait and attach 
continuations—just as with any other task. The task, however, is controlled 
entirely by the `TaskCompletionSource` object via the following methods:

```c#
public class TaskCompletionSource<TResult>
{
    public void SetResult (TResult result);
    public void SetException (Exception exception);
    public void SetCanceled();
 
    public bool TrySetResult (TResult result);
    public bool TrySetException (Exception exception);
    public bool TrySetCanceled();
    ...
}
```

Calling any of these methods signals the task, putting it into a completed, faulted,
or canceled state (we'l cover the latter in the section "Cancellation"). 
You'e supposed to call one of these methods exactly once: if called again,
`SetResult`, `SetException`, or `SetCanceled` will throw an exception, whereas the
Try* methods return false.

The following example prints 42 after waiting for five seconds:

```c#
var tcs = new TaskCompletionSource<int>();
 
new Thread (() => 
   { Thread.Sleep (5000); tcs.SetResult (42); })
   .Start();
 
Task<int> task = tcs.Task;	  // Our "slave" task.
Console.WriteLine (task.Result);  // 42 
With TaskCompletionSource, we can write our own Run method:
Task<TResult> Run<TResult> (Func<TResult> function)
{
   var tcs = new TaskCompletionSource<TResult>();
   new Thread (() =>
   {
      try { tcs.SetResult (function()); }
      catch (Exception ex) { tcs.SetException (ex); }
      }).Start();
   return tcs.Task;
}
...
Task<int> task = Run (() => 
{ 
   Thread.Sleep (5000); 
   return 42; });
```

Calling this method is equivalent to calling Task.Factory.StartNew with the 
`TaskCreationOptions.LongRunning` option to request a non-pooled thread.

The real power of `TaskCompletionSource` is in creating tasks that don't tie up
threads. For instance, consider a task that waits for five seconds and then returns
the number 42. We can write this without a thread through use the `Timer` class,
which with the help of the CLR (and in turn, the operating system) fires an event in
x milliseconds:

```c#
Task<int> GetAnswerToLife()
{
   var tcs = new TaskCompletionSource<int>();
   // Create a timer that fires once in 5000 ms:
   var timer = new System.Timers.Timer (5000) 
      { AutoReset = false }; 
   timer.Elapsed += delegate 
      { timer.Dispose(); 
      tcs.SetResult (42);
      }; 
   timer.Start();
   return tcs.Task;
}
```

Hence our method returns a task that completes five seconds later, 
with a result of 42. By attaching a continuation to the task, we can write its result
without any blocking any thread:

```c#
var awaiter = GetAnswerToLife().GetAwaiter();
awaiter.OnCompleted (() => 
   Console.WriteLine (awaiter.GetResult()));
```

We could make this more useful and turn it into a general-purpose Delay method
by parameterizing the delay time and getting rid of the return value. This means
having it return a Task instead of a `Task<int>`. However, there's no nongeneric
version of TaskCompletionSource, which means we can't directly create a
nongeneric Task. The workaround is simple: since `Task<TResult>` derives from
Task, we create a `TaskCompletionSource<anything>` and then implicitly convert 
the `Task<anything>` that it gives you into a Task, like this:

```c#
var tcs = new TaskCompletionSource<object>(); 
Task task = tcs.Task;
Now we can write our general-purpose Delay method:
Task Delay (int milliseconds)
{
   var tcs = new TaskCompletionSource<object>();
   var timer = new System.Timers.Timer (milliseconds) 
      { AutoReset = false };
   timer.Elapsed += delegate 
      { timer.Dispose(); tcs.SetResult (null); };
   timer.Start();
   return tcs.Task;
}
```

Here's how we can use it to write "42" after five seconds:

```c#
Delay (5000).GetAwaiter().OnCompleted (() => 
   Console.WriteLine (42));
```

Our use of `TaskCompletionSource` without a thread means that a thread is
engaged only when the continuation starts, five seconds later. We can
demonstrate this by starting 10,000 of these operations at once without error 
or excessive resource consumption:

```c#
for (int i = 0; i < 10000; i++)
   Delay (5000).GetAwaiter().OnCompleted (() => 
      Console.WriteLine (42));
```

Timers fire their callbacks on pooled threads, so after five seconds, the thread
pool will receive 10,000 requests to call `SetResult(null)` on a
`TaskCompletionSource`. If the requests arrive faster than they can be processed,
the thread pool will respond by enqueuing and then processing them at the
optimum level of parallelism for the CPU. This is ideal if the thread-bound jobs
are short-running, which is true in this case: the thread-bound job is merely the
call to SetResult plus either the action of posting the continuation to the
synchronization context (in a UI application) or otherwise the continuation itself
`(Console.Write Line(42))`.
