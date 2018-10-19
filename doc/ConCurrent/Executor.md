Executor

------------


## 概述

[官方API文档|Executor](https://developer.android.google.cn/reference/java/util/concurrent/Executor)

```
public interface Executor {
    void execute(Runnable command);
}
```

当我们想创建并执行一个线程的时候，通常是这样的:

```
new Thread(new(RunnableTask())).start()
```

使用这种方式，我们无法控制程序中线程的数量，创建过多的线程会带来性能问题。我们需要一种方式控制线程的数量，有时我们还希望控制各个任务之间调度关系。Executor接口正式为解决这个问题而来。

    An object that executes submitted {@link Runnable} tasks. This
    interface provides a way of decoupling task submission from the
    mechanics of how each task will be run, including details of thread use, scheduling, etc.  An {@code Executor} is normally used
    instead of explicitly creating threads.

Executor不会显示的创建线程，而是通过下面的方式：

```
 Executor executor = anExecutor;
 executor.execute(new RunnableTask1());
 executor.execute(new RunnableTask2());
 ...

```

Executor接口并不严格要求执行是异步的。在最简单的情况下，执行程序可以在调用者的线程中立即运行提交的任务

```
 class DirectExecutor implements Executor {
   public void execute(Runnable r) {
     r.run();
   }
 }
```

更典型的是，任务在一些线程中执行，而不是在调用者的线程中执行。下面的executor为每个任务生成一个新线程。

```
 class ThreadPerTaskExecutor implements Executor {
   public void execute(Runnable r) {
     new Thread(r).start();
   }
 }
```

许多Executor的实现对任务安排的方式和时间施加了某种限制。下面的执行程序序列化将任务提交给第二个执行程序，用来说明复合执行程序。

```
 class SerialExecutor implements Executor {
   final Queue<Runnable> tasks = new ArrayDeque<>();
   final Executor executor;
   Runnable active;

   SerialExecutor(Executor executor) {
     this.executor = executor;
   }

   public synchronized void execute(final Runnable r) {
     tasks.add(new Runnable() {
       public void run() {
         try {
           r.run();
         } finally {
           scheduleNext();
         }
       }
     });
     if (active == null) {
       scheduleNext();
     }
   }

   protected synchronized void scheduleNext() {
     if ((active = tasks.poll()) != null) {
       executor.execute(active);
     }
   }
 }
```

这个包中提供的Executor的实现同时实现了ExecutorService，它是一个更广泛的接口。ThreadPoolExecutor类提供了一个可扩展的线程池实现。
Executors类为这些Executors提供了方便的工厂方法。


## ExecutorService

```
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

内存一致性影响:在向执行程序提交可运行对象之前，线程中的操作——在执行开始之前，可能是在另一个线程中。


## 设计模式　－　中介者模式

其中Executor就是一个抽象中介者角色，Executor的具体实现就是一个具体中介者角色，每个Runnable就相当于同事角色，Executor的execute(Runnable...)方法即是各个Runnable之间通信的接口，每个Runnable都被提交到Executor的具体实现类中，由Executor的具体实现类来协调各Runnable之间的实现协作行为。

## 常见子类

<table>
    <tr>
        <td>AbstractExecutorService</td>
        <td>Provides default implementations of ExecutorService execution methods. </td>
    </tr>
    <tr>
        <td>ExecutorService</td>
        <td>An Executor that provides methods to manage termination and methods that can produce a Future for tracking progress of one or more asynchronous tasks.  </td>
    </tr>
    <tr>
        <td>ForkJoinPool</td>
        <td>An ExecutorService for running ForkJoinTasks.  </td>
    </tr>
        <tr>
        <td>ScheduledExecutorService</td>
        <td>An ExecutorService that can schedule commands to run after a given delay, or to execute periodically.  </td>
    </tr>
        <tr>
        <td>ScheduledThreadPoolExecutor</td>
        <td> 	A ThreadPoolExecutor that can additionally schedule commands to run after a given delay, or to execute periodically.  </td>
    </tr>
        <tr>
        <td>ThreadPoolExecutor</td>
        <td>An ExecutorService that executes each submitted task using one of possibly several pooled threads, normally configured using Executors factory methods.  </td>
    </tr>
</table>

## 源码分析 - SerializingExecutor

SerializingExecutor确保提交的所有可运行项都在构造方法中所提供的Executor中按顺序执行，并且连续地，不会有两个执行程序同时执行。

```

final class SerializingExecutor implements Executor {
  private static final Logger log =
      Logger.getLogger(SerializingExecutor.class.getName());

  /** Underlying executor that all submitted Runnable objects are run on. */
  private final Executor executor;

  /** A list of Runnables to be run in order. */
  @GuardedBy("internalLock")
  private final Queue<Runnable> waitQueue = new ArrayDeque<Runnable>();


  @GuardedBy("internalLock")
  private boolean isThreadScheduled = false;

  /** The object that actually runs the Runnables submitted, reused. */
  private final TaskRunner taskRunner = new TaskRunner();

  /**
   * Creates a SerializingExecutor, running tasks using {@code executor}.
   *
   * @param executor Executor in which tasks should be run. Must not be null.
   */
  public SerializingExecutor(Executor executor) {
    Preconditions.checkNotNull(executor, "'executor' must not be null.");
    this.executor = executor;
  }

  private final Object internalLock = new Object() {
    @Override public String toString() {
      return "SerializingExecutor lock: " + super.toString();
    }
  };


  @Override
  public void execute(Runnable r) {
    Preconditions.checkNotNull(r, "'r' must not be null.");
    boolean scheduleTaskRunner = false;
    synchronized (internalLock) {
      waitQueue.add(r);

      if (!isThreadScheduled) {
        isThreadScheduled = true;
        scheduleTaskRunner = true;
      }
    }
    if (scheduleTaskRunner) {
      boolean threw = true;
      try {
        executor.execute(taskRunner);
        threw = false;
      } finally {
        if (threw) {
          synchronized (internalLock) {
            // It is possible that at this point that there are still tasks in
            // the queue, it would be nice to keep trying but the error may not
            // be recoverable.  So we update our state and propogate so that if
            // our caller deems it recoverable we won't be stuck.
            isThreadScheduled = false;
          }
        }
      }
    }
  }


  private class TaskRunner implements Runnable {
    @Override
    public void run() {
      boolean stillRunning = true;
      try {
        while (true) {
          Preconditions.checkState(isThreadScheduled);
          Runnable nextToRun;
          synchronized (internalLock) {
            nextToRun = waitQueue.poll();
            if (nextToRun == null) {
              isThreadScheduled = false;
              stillRunning = false;
              break;
            }
          }

          // Always run while not holding the lock, to avoid deadlocks.
          try {
            nextToRun.run();
          } catch (RuntimeException e) {
            // Log it and keep going.
            log.log(Level.SEVERE, "Exception while executing runnable "
                + nextToRun, e);
          }
        }
      } finally {
        if (stillRunning) {
          // An Error is bubbling up, we should mark ourselves as no longer
          // running, that way if anyone tries to keep using us we won't be
          // corrupted.
          synchronized (internalLock) {
            isThreadScheduled = false;
          }
        }
      }
    }
  }
}
```

看一下测试用例

```
/**
 * Tests {@link SerializingExecutor}.
 *
 * @author JJ Furman
 */
public class SerializingExecutorTest extends TestCase {
  private static class FakeExecutor implements Executor {
    Queue<Runnable> tasks = Queues.newArrayDeque();
    @Override public void execute(Runnable command) {
      tasks.add(command);
    }

    boolean hasNext() {
      return !tasks.isEmpty();
    }

    void runNext() {
      assertTrue("expected at least one task to run", hasNext());
      tasks.remove().run();
    }

  }
  private FakeExecutor fakePool;
  private SerializingExecutor e;

  @Override
  public void setUp() {
    fakePool = new FakeExecutor();
    e = new SerializingExecutor(fakePool);
  }

  public void testSerializingNullExecutor_fails() {
    try {
      new SerializingExecutor(null);
      fail("Should have failed with NullPointerException.");
    } catch (NullPointerException expected) {
    }
  }

  public void testBasics() {
    final AtomicInteger totalCalls = new AtomicInteger();
    Runnable intCounter = new Runnable() {
      @Override
      public void run() {
        totalCalls.incrementAndGet();
      }
    };

    assertFalse(fakePool.hasNext());
    e.execute(intCounter);
    assertTrue(fakePool.hasNext());
    e.execute(intCounter);
    assertEquals(0, totalCalls.get());
    fakePool.runNext(); // run just 1 sub task...
    assertEquals(2, totalCalls.get());
    assertFalse(fakePool.hasNext());

    // Check that execute can be safely repeated
    e.execute(intCounter);
    e.execute(intCounter);
    e.execute(intCounter);
    assertEquals(2, totalCalls.get());
    fakePool.runNext();
    assertEquals(5, totalCalls.get());
    assertFalse(fakePool.hasNext());
  }

  public void testOrdering() {
    final List<Integer> callOrder = Lists.newArrayList();

    class FakeOp implements Runnable {
      final int op;

      FakeOp(int op) {
        this.op = op;
      }

      @Override
      public void run() {
        callOrder.add(op);
      }
    }

    e.execute(new FakeOp(0));
    e.execute(new FakeOp(1));
    e.execute(new FakeOp(2));
    fakePool.runNext();

    assertEquals(ImmutableList.of(0, 1, 2), callOrder);
  }

  public void testExceptions() {

    final AtomicInteger numCalls = new AtomicInteger();

    Runnable runMe = new Runnable() {
      @Override
      public void run() {
        numCalls.incrementAndGet();
        throw new RuntimeException("FAKE EXCEPTION!");
      }
    };

    e.execute(runMe);
    e.execute(runMe);
    fakePool.runNext();

    assertEquals(2, numCalls.get());
  }

  public void testDelegateRejection() {
    final AtomicInteger numCalls = new AtomicInteger();
    final AtomicBoolean reject = new AtomicBoolean(true);
    final SerializingExecutor executor = new SerializingExecutor(
        new Executor() {
          @Override public void execute(Runnable r) {
            if (reject.get()) {
              throw new RejectedExecutionException();
            }
            r.run();
          }
        });
    Runnable task = new Runnable() {
      @Override
      public void run() {
        numCalls.incrementAndGet();
      }
    };
    try {
      executor.execute(task);
      fail();
    } catch (RejectedExecutionException expected) {}
    assertEquals(0, numCalls.get());
    reject.set(false);
    executor.execute(task);
    assertEquals(2, numCalls.get());
  }

  public void testTaskThrowsError() throws Exception {
    class MyError extends Error {}
    final CyclicBarrier barrier = new CyclicBarrier(2);
    // we need to make sure the error gets thrown on a different thread.
    ExecutorService service = Executors.newSingleThreadExecutor();
    try {
      final SerializingExecutor executor = new SerializingExecutor(service);
      Runnable errorTask = new Runnable() {
        @Override
        public void run() {
          throw new MyError();
        }
      };
      Runnable barrierTask = new Runnable() {
        @Override
        public void run() {
          try {
            barrier.await();
          } catch (Exception e) {
            throw new RuntimeException(e);
          }
        }
      };
      executor.execute(errorTask);
      service.execute(barrierTask);  // submit directly to the service
      // the barrier task runs after the error task so we know that the error has been observed by
      // SerializingExecutor by the time the barrier is satified
      barrier.await(10, TimeUnit.SECONDS);
      executor.execute(barrierTask);
      // timeout means the second task wasn't even tried
      barrier.await(10, TimeUnit.SECONDS);
    } finally {
      service.shutdown();
    }
  }
}
```