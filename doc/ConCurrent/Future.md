Future

------------


## 概述

[官方API文档|Future](https://developer.android.google.cn/reference/java/util/concurrent/Future)

    A Future represents the result of an asynchronous computation. 
    Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation. 
    The result can only be retrieved using method get when the computation has completed, blocking if necessary until it is ready. 
    Cancellation is performed by the cancel method. 
    Additional methods are provided to determine if the task completed normally or was cancelled. Once a computation has completed, the computation cannot be cancelled. 
    If you would like to use a Future for the sake of cancellability but not provide a usable result, you can declare types of the form Future<?> and return null as a result of the underlying task. 

* 一个Future代表了一个异步计算的结果。 它提供了可以检查计算是否完成、等待计算完成、检索计算结果的方法。
* get方法获得计算结果的唯一方法，如果计算没有完成，此方法会堵塞直到计算完成。
* cancel方法可以用来取消这次计算。一个已完成的计算是不能被取消的。
* isDone和isCancelled方法可以查询计算是否正常完成还是被取消掉了。
* 如果我们不想知道此异步计算的结果，只是想随时取消这次计算，可以通过声明Future<?>并将get的返回值设为null。


代码示例 (伪代码)： 

```
 interface ArchiveSearcher { String search(String target); }
 class App {
   ExecutorService executor = ...
   ArchiveSearcher searcher = ...
   void showSearch(final String target)
       throws InterruptedException {
     Future<String> future
       = executor.submit(new Callable<String>() {
         public String call() {
             return searcher.search(target);
         }});
     displayOtherThings(); // do other things while searching
     try {
       displayText(future.get()); // use future
     } catch (ExecutionException ex) { cleanup(); return; }
   }
 }
```

    The FutureTask class is an implementation of Future that implements Runnable, and so may be executed by an Executor. 
    
For example, the above construction with submit could be replaced by: 

```
 FutureTask<String> future =
   new FutureTask<>(new Callable<String>() {
     public String call() {
       return searcher.search(target);
   }});
 executor.execute(future);
```

    Memory consistency effects: Actions taken by the asynchronous computation happen-before actions following the corresponding Future.get() in another thread.

## 常用方法

<table>
    <tr>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td> abstract boolean </td>
        <td>cancel(boolean mayInterruptIfRunning)

Attempts to cancel execution of this task. </td>
    </tr>
    <tr>
        <td>abstract V</td>
        <td> 	get(long timeout, TimeUnit unit)

Waits if necessary for at most the given time for the computation to complete, and then retrieves its result, if available. </td>
    </tr>
    <tr>
        <td>abstract V</td>
        <td>get()

Waits if necessary for the computation to complete, and then retrieves its result. </td>
    </tr>
    <tr>
        <td>abstract boolean</td>
        <td>isCancelled()

Returns true if this task was cancelled before it completed normally. </td>
    </tr>
    <tr>
        <td>abstract boolean</td>
        <td> 	isDone()

Returns true if this task completed. </td>
    </tr>
    <tr>
        <td></td>
        <td></td>
    </tr>
</table>

## RunnableFuture

    A Future that is Runnable. Successful execution of the run method causes completion of the Future and allows access to its results.

一个Runnable的Future,成功执行run方法将导致Future的完成，并允许访问其结果。

```
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

## FutureTask
[官方API文档|FutureTask](https://developer.android.google.cn/reference/java/util/concurrent/FutureTask)

先看一下官方的介绍： 

    A cancellable asynchronous computation. 
    This class provides a base implementation of Future, with methods to start and cancel a computation, query to see if the computation is complete, and retrieve the result of the computation. 
    The result can only be retrieved when the computation has completed; the get methods will block if the computation has not yet completed. Once the computation has completed, the computation cannot be restarted or cancelled (unless the computation is invoked using runAndReset()). 

    A FutureTask can be used to wrap a Callable or Runnable object. Because FutureTask implements Runnable, a FutureTask can be submitted to an Executor for execution.

    In addition to serving as a standalone class, this class provides protected functionality that may be useful when creating customized task classes.


要点如下：

1. FutureTask同时实现了Runnable和Future两个接口，所以FutureTask可以提交给Executor。


### FutureTask源码分析

构造方法：
不管我们使用哪个构造函数，其内部都是把将传入的参数保存到callable，并且把状态置为NEW。

```

    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }


    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
```

FutureTask一共声明了7个状态
```
    /**
     * The run state of this task, initially NEW.  The run state
     * transitions to a terminal state only in methods set,
     * setException, and cancel.  During completion, state may take on
     * transient values of COMPLETING (while outcome is being set) or
     * INTERRUPTING (only while interrupting the runner to satisfy a
     * cancel(true)). Transitions from these intermediate to final
     * states use cheaper ordered/lazy writes because values are unique
     * and cannot be further modified.
     *
     * Possible state transitions:
     * NEW -> COMPLETING -> NORMAL
     * NEW -> COMPLETING -> EXCEPTIONAL
     * NEW -> CANCELLED
     * NEW -> INTERRUPTING -> INTERRUPTED
     */
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;

```

<table>
    <tr>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>状态</td>
        <td>说明</td>
    </tr>
    <tr>
        <td>NEW</td>
        <td>初始状态</td>
    </tr>
    <tr>
        <td>COMPLETING</td>
        <td>进行中状态，表示正在设置结果。很短暂的一个状态</td>
    </tr>
    <tr>
        <td>NORMAL</td>
        <td>正常结束的状态</td>
    </tr>
    <tr>
        <td>EXCEPTIONAL</td>
        <td>异常状态，任务异常结束</td>
    </tr>
    <tr>
        <td>CANCELLED</td>
        <td>任务成功被取消的状态</td>
    </tr>
    <tr>
        <td>INTERRUPTING</td>
        <td>很短暂的状态，当在NEW状态下，调用了cancel(true),则状态就会转换为INTERRUPTING，直到执行了Thread#interrupt()方法，状态转换为INTERRUPTED</td>
    </tr>
    <tr>
        <td>INTERRUPTED</td>
        <td>任务被中断后的状态</td>
    </tr>
    <tr>
        <td></td>
        <td></td>
    </tr>
</table>

run()方法讲解:

1. 先进行状态检查，检查是否处于NEW状态,不是则返回
2. 将执行线程的引用保存在runner的变量
3. FurtureTask的runner变量用来引用任务执行所在的线程。
4. 然后执行Callable的call的方法，进行任务执行。
5. 接下来会出现两种情况：
    * 情况一： 如果执行顺利完成，则调用set(result)的方法。在set()方法中，先将状态置为COMPLETING，然后将执行结果保存到全局变量outcome中，然后将状态置为NORMAL。然后调动finishCompletion()方法，通知所有等待结果的线程，并调用done()(在这里是个空方法)
    * 情况二：如果执行出现了异常。则执行setException()方法。在setException()方法中，操作基本和set()方法一样，只是outcome保存的是Throwable。


```
    public void run() {
        if (state != NEW ||
            !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
```

isCanceled()和isDone()

```
    public boolean isCancelled() {
        return state >= CANCELLED;
    }

    public boolean isDone() {
        return state != NEW;
    }
```

cancel(boolean)方法

1. 检测当前状态是否是NEW,如果不是，说明任务已经完成或取消或中断，所以直接返回。
2. 当符合条件后，检查mayInterruptIfRunning的值：
    * 如果mayInterruptIfRunning == false,则直接将状态设置为CANCELLED，并且调用finishCompletion()方法，通知正在等待结果的线程。
    * 如果mayInterruptIfRunning == true,则暂时将状态设置为INTERRUPTING，然后试着中断线程，完成后将状态设置为INTERRUPTED，最后调用finishCompletion()方法，通知正在等待结果的线程。
3. mayInterruptIfRunning为true,调用Thread的interrupt方法停止线程

```
    public boolean cancel(boolean mayInterruptIfRunning) {
        if (!(state == NEW &&
              U.compareAndSwapInt(this, STATE, NEW,
                  mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
            return false;
        try {    // in case call to interrupt throws exception
            if (mayInterruptIfRunning) {
                try {
                    Thread t = runner;
                    if (t != null)
                        t.interrupt();
                } finally { // final state
                    U.putOrderedInt(this, STATE, INTERRUPTED);
                }
            }
        } finally {
            finishCompletion();
        }
        return true;
    }

```

get()方法的实现

1. 首先还是进行状态监测:
    * 如果现在正处于NEW和COMPLETING状态，则会调用awaitDone()，直到状态的转变为其他状态, 然后调用report()方法
    * 否则直接调用report()方法
2. 在report()方法中，首先监测状态：
    * 如果是NORMAL状态，直接返回保存在outCome中的结果；
    * 如果是CANCELLED、INTERRUPTING、INTERRUPTED状态，则抛出CancellationException()；
    * 如果处于其他状态则抛出ExecutionException（比如调用了get(true,atime)方法，时间到期后状态可能还处于NEW状态）。


```
    /**
     * @throws CancellationException {@inheritDoc}
     */
    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }


    private V report(int s) throws ExecutionException {
        Object x = outcome;
        if (s == NORMAL)
            return (V)x;
        if (s >= CANCELLED)
            throw new CancellationException();
        throw new ExecutionException((Throwable)x);
    }

```

其中awaitDone(boolean timed, long nanos)方法用于等待任务完成、任务中断或时间到期。
在此方法体中，显示将这次等待保存到全局变量watiers中，用于记录所有调用了get()渴望获得结果并堵塞的Thread，然后不停的循环查询state。
直到时间到期或执行完成，则将循环中断，返回转变后的state供report()方法使用。


注：在FutureTask的的源码中，使用了sun.misc.Unsafe进行状态的赋值等操作，这是一个强大的对内存进行操作的类，可以通过它绕过jdk的很多限制。

### 为什么需要FutureTask

FutureTask 实现了 Runnable 和 Future，所以兼顾两者优点，既可以在 Thread 中使用，又可以在 ExecutorService 中使用。

使用 FutureTask 的好处是 FutureTask 是为了弥补 Thread 的不足而设计的，它可以让程序员准确地知道线程什么时候执行完成并获得到线程执行完成后返回的结果。FutureTask 是一种可以取消的异步的计算任务，它的计算是通过 Callable 实现的，它等价于可以携带结果的 Runnable，并且有三个状态：等待、运行和完成。完成包括所有计算以任意的方式结束，包括正常结束、取消和异常。

## 源码分析

```
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

## 参考文档

[Android concurrent（二）—— Future和FutureTask](https://www.jianshu.com/p/567434bd7698)

[并发编程 – Concurrent 用户指南](http://www.importnew.com/26461.html)
