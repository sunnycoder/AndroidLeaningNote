Process && Thread

-----

## 概述

[官方文档|Processes and threads overview ](https://developer.android.google.cn/guide/components/processes-and-threads)

[中文翻译](https://blog.csdn.net/jennyliliyang/article/details/78713327)

Android中的线程，线程池，多线程通信

## 进程

### 优先级

下面是五类Android进程，他们的优先级顺序排列：

    Foreground process：前台进程。
    Visible prcess：可见进程。
    Service process：服务进程。
    Background process：后台进程。
    Empty process：空进程。

Tips：一个进程的优先级是可以变化的。

### 生命周期


## 线程

进程(Process)是程序的一个运行实例，以区别于“程序”这一静态的概念
线程(Thread)则是cpu调度的基本单位。
一个进程中至少有一个线程。

在JAVA中默认情况下一个进程只有一个线程，这个线程就是主线程。
主线程主要处理界面交互相关的逻辑，因为用户随时会和界面发生交互，因此主线程在任何时候都必须有比较高的响应速度，否则就会产生一种界面卡顿的感觉。为了保持较高的响应速度，这就要求主线程中不能执行耗时的任务，这个时候子线程就派上用场了。
子线程也叫工作线程，除了主线程以外的线程都是子线程。


Android沿用了JAVA的线程模型，其中的线程也分为主线程和子线程，其中主线程又叫UI线程。
在Android系统中，在默认情况下，一个应用程序内的各个组件(如Activity、BroadcastReceiver、Service)都会在同一个进程(Process)里执行，且由此进程的主线程负责执行。如果有特别指定(通过android:process)，也可以让特定组件在不同的进程中运行。无论组件在哪一个进程中运行，默认情况下，他们都由此进程的主线程负责执行。

主线程既要处理Activity组件的UI事件，又要处理Service后台服务工作，通常会忙不过来。为了解决此问题，主线程可以创建多个子线程来处理后台服务工作，而本身专心处理UI画面的事件。子线程的任务则是执行耗时任务，比如网络请求，I/O操作等。

当应用程序启动时，系统为应用程序创建主线程。
此线程非常重要，因为它负责将事件分派到的widgets。
应用程序与UI组件交换也主要在主线程中进行，所以主线程也叫UI线程。
然而，在特殊情况下，应用程序的主线程可能不是它的UI线程；
有关更多信息，请参见 Thread annotations。

系统不为组件的每个实例创建单独的线程。运行在同一进程中的所有组件都在该进程的UI线程中实例化，系统从UI线程发出对每个组件的调用。
因此，响应系统的回调函数（如onkeydown()或onCreate()）始终运行在进程的UI线程。
例如，当用户触摸屏幕上的某个按钮时，应用程序的UI线程将触摸事件分发给UI控件，后者依次设置其按下状态，并向事件队列发送无效请求。
UI线程取出请求并通知UI控件重新绘制。

如果在UI线程中做所有事情，那么执行诸如网络访问或数据库查询之类的耗时操作将阻塞整个UI。当线程被阻塞时，就不能发送任何事件，包括绘制事件。从用户的角度来看，应用程序就会出现卡顿。更糟糕的是，如果UI线程被阻塞超过几秒钟（现在大约是5秒），系统就会向用户弹出“应用程序没有响应”（ANR）的对话框。然后，用户可能会退出应用程序，甚至卸载它。

另外，安卓UI toolkit不是线程安全的。因此，您不能从工作线程操作您的UI，您必须从UI线程对用户界面进行操作。因此，Android的单线程模型有两条规则：
1、不要阻塞UI线程
2、不要在非UI线程中操作UI控件。

### 工作线程

Android提供了下面几种从其他线程访问UI线程的方法： 

### 线程安全的方法

Activity.runOnUiThread(Runnable)
View.post(Runnable)
View.postDelayed(Runnable, long)

然而，随着操作复杂度的增加，这种代码会变得复杂和难以维护。为了处理UI线程与工作线程更复杂的交互，您可以考虑在工作线程中使用Handler来处理从UI线程传递的消息。或者用AsyncTask，实现后台工作任务，且更新UI。

### 使用AsyncTask

AsyncTask允许你在用户界面上执行异步操作。它在工作线程中执行耗时操作，然后更新UI，而不需要您自己处理线程或Handler。你必须扩展AsyncTask并且实现doInBackground()回调方法，它启动的线程会运行在后台线程池。如果要更新UI，你应该实现onPostExecute()。它会获取从doInBackground()返回的结果，并在UI线程中更新UI。然后可以在UI线程中调用execute()运行任务。更多关于AsyncTask的资料请参考 AsyncTask。

### 开启一个线程

1. 继承Thread类

```
public class MyThread extends Thread {  
      
    public void run(){  
      
    }  
}

new MyThread().start(); 
```

2. 实现Runnable接口

```
public class MyRunnable implements Runnable{

@Override
public void run(){
    //TODO
    }
}


```

### Runnable

### Thread

### 线程注解

[线程注解](https://developer.android.google.cn/studio/write/annotations#thread-annotations)


## 进程间通信

Android使用远程过程调用（RPC）提供了一种进程间通信机制（IPC）

## 多线程通信

### Activity.runOnUiThread(Runnable)

### View.post(Runnable)

### View.postDelayed(Runnable,long)

### Handle

#### AsyncTask


## 线程池

## 面试相关

* 线程和进程有什么区别？

一个进程是一个独立(self contained)的运行环境，它可以被看作一个程序或者一个应用。而线程是在进程中执行的一个任务。
线程是进程的子集，一个进程可以有很多线程，每条线程并行执行不同的任务。
不同的进程使用不同的内存空间，而所有的线程共享一片相同的内存空间。
别把它和栈内存搞混，每个线程都拥有单独的栈内存用来存储本地数据。


* 如何在Java中实现线程？

一、继承 Thread 类，扩展线程。
二、实现 Runnable 接口。

* Thread 类中的 start() 和 run() 方法有什么区别？

调用 start() 方法才会启动新线程；如果直接调用 Thread 的 run() 方法，它的行为就会和普通的方法一样；为了在新的线程中执行我们的代码，必须使用 Thread.start() 方法。

* 用 Runnable 还是 Thread ？

Java不支持类的多重继承，但允许你调用多个接口。所以如果你要继承其他类，当然是调用Runnable接口更好了。

* Runnable 和 Callable 有什么不同？

Runnable 和 Callable 都代表那些要在不同的线程中执行的任务。
Runnable 从 JDK1.0 开始就有了，Callable 是在 JDK1.5 增加的。
它们的主要区别是 Callable 的 call() 方法可以返回值和抛出异常，而 Runnable 的 run() 方法没有这些功能。
Callable 可以返回装载有计算结果的 Future 对象。


```
public interface Runnable {
    public void run();
}

public interface Callable<V> {
    V call() throws Exception;
}
```

* 什么是 Executor 框架？

Executor框架在Java 5中被引入，Executor 框架是一个根据一组执行策略调用、调度、执行和控制的异步任务的框架。

无限制的创建线程会引起应用程序内存溢出，所以创建一个线程池是个更好的的解决方案，因为可以限制线程的数量并且可以回收再利用这些线程。利用 Executor 框架可以非常方便的创建一个线程池。

* Executors 类是什么？

Executors为Executor、ExecutorService、ScheduledExecutorService、ThreadFactory 和 Callable 类提供了一些工具方法。Executors 可以用于方便的创建线程池。

## 参考文档

[Android中线程那些事儿](https://www.jianshu.com/p/d59b3cce2b54)
[线程、多线程与线程池总结](https://www.jianshu.com/p/b8197dd2934c)
