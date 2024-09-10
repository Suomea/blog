---
comments: 
tags:
---
为什么需要并发，用来解决什么问题？  

- 提高资源的利用率，在某些情况下线程必须等待磁盘或者网络的 I/O 而阻塞，此时 CPU 空闲的时间片就可以运行另一个线程，提高资源的利用率。
- 公平性，多个用户和程序共享计算机资源，而不是一个程序一直运行，然后再启动下一个程序。
- 现代的 CPU 往往具有多个核心，天然具备并行运算的能力。多线程能够充分发挥处理器的强大能力。

并发带来的主要挑战是什么？

- 原子性，我们希望一些列指令能够原子地执行，在并发情况下，由于 CPU 调度带来了不确定性。
- 可视性，由于 CPU 的缓存和 Java 的内存模型，导致一个线程对共享变量的修改另一个线程不一定能过够及时看到。
- 有序性，在执行程序时，为了提高性能，编译器和处理器可能会对指令做重排序。

## 基础知识

### 原子操作
是不能被线程调度机制中断的操作，一旦操作开始，那么它一定可以在可能发生的上下文切换之前执行完毕。

### 监视器 Monitor
Java 每个对象都有一个监视器（monitor），监视器相当于锁。监视器具有 ownership 和 entry count 两个属性，表示锁的持有者和重入次数。

监视器用来实现语法层面的锁 synchronized，由 JVM 处理锁的获取，等待队列。ReentrantLock 为 API 层面的锁，Java 提供了 CAS、LockSupport 和 AQS 来支持 ReentrantLock，来实现更强大的锁。AQS 内部通过成员变量来维护锁的状态、等待队列等信息。
## 线程的状态

- **New** 新建的线程还没有开始执行。
- **Runnable** 可运行，已经在运行或者可以运行但是还没有获得 CPU 时间。
- **Blocked** 阻塞，在等待锁。
- **Waiting** 等待，Object.wait、Thread.join 不指定时间、 LockSupport.park 不指定时间，都会让线程处于该状态。该状态该在等待另一个线程的特定行为，比如 Object.notify/notifyAll、另一个线程结束、另一个线程 unpark 调用。 条件变量和信号量也会让线程处于该状态。
- **Timed waiting** 计时等待，指定指定的时间，Thread.sleep、Thread.join 指定时间、LockSupport.park 指定时间。
- **Terminated** 终止，线程执行完毕。

获取线程状态示例代码：
```c
    static volatile int done = 0;

    public static void main(String[] args) {
        Thread b = new Thread("B") {
            @Override
            public void run() {
                Thread currentThread = Thread.currentThread();
                System.out.println("hello, i'm " + currentThread.getName() 
					                + ", and my state is " + currentThread.getState());
                done = 1;
            }
        };

        System.out.println("B start before state is " + b.getState());
        b.start();
        while (done == 0)
            ;
        System.out.println("B start after state is " + b.getState());
    }
```

运行输出：
```
B start before state is NEW
hello, i'm B. And my state is RUNNABLE
B start after state is TERMINATED
```

## 线程中断
java.lang.Thread 中断常用的方法：
```
1. void interrupt() 向线程发送中断请求，线程的中断状态将被设置为 true。

2. static boolean interrupted() 测试当前线程是否被中断，静态方法，会把当前线程的中断状态重置为 false。

3. boolean isInterrupted() 测试线程是否被中断，不会改变线程的重置状态。

4. static Thread currentThread() 返回当前正在执行的线程。
```

void interrupt() 是一个明确的指示，告诉线程应该停止当前所做的事情。要使中断机制正常工作，线程本身必须要支持中断，换句话说要由程序员决定如何响应中断。

如果线程调用的方法需要捕获处理 InteruptedException，如下面代码，那么当线程阻塞在 sleep() 方法上时，此时对线程调用 interrupt() 方法，线程将立刻从 sleep() 阻塞返回并捕获 InteruptedException（或者任何要求抛出 InterruptedException 的调用）。
```java
for (int i = 0; i < importantInfo.length; i++) {
    // Pause for 4 seconds
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // We've been interrupted: no more messages.
        return;
    }
    // Print a message
    System.out.println(importantInfo[i]);
}
```


如果线程没有调用抛出 InterruptedException 的方法，那么如果想要使中断机制正常工作，就要在代码中不断调用 Thread.interrupted() 检查是否已收到中断：
```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```
更复杂业务的情况下，最好是检测到中断后抛出 InterruptedException，这样可以在外部的调用将处理中断的代码放在一个 catch 子句中。

中断机制是使用内部 interrupted 标志来实现的，通过 interrupt() 方法将设置该标志。当线程内部使用静态方法 static boolean interrupted() 方法检查中断时，会获取中断状态然后会清除中断状态。而线程通过 boolean isInterrupted() 方法查询另一个线程的中断状态时，不会更改中断状态。
```java
	// Thread 的成员变量
	volatile boolean interrupted;
```

通过 InterruptedException 如 sleep() 方法，在捕获该异常后线程的中断状态将自动清除。但是另一个线程可以继续设置中断。

如果线程阻塞在锁或者 I/O 等待时，也不会对 interrupt() 中断进行响应。

当然 Java 也提供了 ReentrantLock 阻塞可以被中断的能力，通过 lockIntettuptibly() 方法获取锁，该方法要求抛出 InterruptedException 异常。

## 线程相关的方法
### wait/notify 方法
java.lang.Object wait/notify 相关的方法可以用来实现条件变量的效果（需要搭配 synchronized），建议使用 Lock/Condtion 来实现条件变量。

wait 的使用模板：
```java
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait();
    ... // Perform action appropriate to condition
}
```

wait/notify 为什么需要 synchronized 或者说为什么需要锁？因为它们是使用在条件变量的场景下，wait 等待一个条件成立，那么必然另一个线程要修改条件的状态才能使条件成立，所以条件本身就是临界资源，需要用锁来保护临界资源（原子性、可视性）。因此，wait 调用会释放锁，调用返回时会自动获取锁。

while 循环判断是为了解决虚假唤醒或者多线程竞争同一个条件，notifyAll 全部唤醒时，重新检查条件，因为条件可能被其它线程修改。

如果 wait/notify 的调用线程不是对象 monitor 的持有者，那么方法会抛出 IllegalMonitorStateException。

相关方法介绍：
```
final void wait() 线程进入 Waiting 状态，直到它得到通知。

final void wait(long millis)
final void wait(long millis, int nanos)
与 wait() 类似，会进入 Timed waiting 状态，直到它得到通知或者经过了指定的时间。

final void nitofy() 随机选择一个在这个对象监视器上等待的线程，解除其等待状态。
final void notifyAll() 解除在这个对象监视器上等待的所有线程，之后这些线程会一直尝试获取锁，然后从 wait 调用返回执行。
```


!!!note "wait 会释放锁，wait 返回的时候会自动重新获取锁。那么如果计时结束返回或者 wait 等待过程中被中断返回，是立马返回并且自动获取锁吗？"
	wait 返回时线程会重新获取对象的锁，下面无论哪种情况，都会在尝试获取并且获取锁之后发生。
	
	- 计时等待结束返回。
	- 外部中断，抛出 IntrruptedException 返回。
	- 被 notify 或 notifyAll 唤醒。
### join 方法
相关的方法：
```
final void join() 线程进入 Waiting 状态，直到它等待的线程结束运行。

final void join(long millis)
final void join(long millis, int nanos)
与 join() 类似，会进入 Timed waiting 状态，直到它等待的线程结束运行或者经过了指定的时间。
```

实现 10 个线程先后运行，主线线程 main，0，1，2……：
```java
    public static void main(String[] args) {
        Thread previous = Thread.currentThread();
        for(int i = 0; i < 10; i ++) {
            Thread thread = new Thread(new Runner(previous), String.valueOf(i));
            thread.start();
            previous = thread;
        }
        System.out.println(Thread.currentThread().getName() + " done!");
    }

    static class Runner implements Runnable {

        private Thread thread;

        public Runner(Thread thread) {
            this.thread = thread;
        }

        @Override
        public void run() {
            try {
                thread.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(Thread.currentThread().getName() + " done!");
        }
    }
```


jion 方法的逻辑，精简版：
```java
    public final void join() throws InterruptedException {
        synchronized (this) {
            while (isAlive()) {
                wait(0);
            }
        }
    }
```
可以看到如果线程已经结束了，直接返回。否则，也是使用了 wait/notify 机制，当线程终止时，会调用自身的 notifyAll 方法，唤醒所有等待在该线程监视器上的对象。

join 并不会放弃当前线程在调用 jion 之前已经持有的锁。

## 使用线程的方法

- 实现 Runnable 接口。
- 实现 Callable 接口。
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当作一个可以在线程中运行的任务，不是真正意义上的线程，因为最后还需要通过 Thread 来启动线程执行任务。

通过继承 Thread 调用 start() 方法来启动线程：
```java
    static volatile int done = 0;

    public static void main(String[] args) {
        System.out.println("main thread start!");

        new Thread("B") {
            @Override
            public void run() {
                System.out.println("hello, i'm " + Thread.currentThread().getName());
                done = 1;
            }
        }.start();

        while (done == 0)
            ;
        System.out.println("main thread end!");
    }
```

实现 Runnable 接口：
```java
    static volatile int done = 0;  
  
    public static void main(String[] args) {  
        System.out.println("main thread start!");  
  
        new Thread(new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("hello, i'm " + Thread.currentThread().getName());  
                done = 1;  
            }  
        }, "B").start();  
        while (done == 0)  
            ;  
        System.out.println("main thread end!");  
    }  
```

实现 Callable 接口：
```c
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println("main thread start!");

        FutureTask<String> futureTask = new FutureTask<>(new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "hello, i'm B";
            }
        });
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
        System.out.println("main thread end!");
    }
```


## volatile 
`volatile` 可以说是 JVM 提供的轻量级线程同步机制。如果将一个域声明为 `volatile` 的，那么只要对这个域产生了写操作，那么所有后续的读操作都可以看到这个修改，即使使用了本地缓存。换句话说，`volatile` 保证可视性。

原子操作可以应用于除 `long` 和 `double` 之外的所有基本类型之上的“简单操作”。对于读和写入除 `long` 和 `double` 之外的基本类型变量的操作，可以保证它们是原子的。但是 JVM 对于 64 位的 `long` 和 `double` 的读取和写入当作两个分离的 32 位操作来执行，如果发生了上下文切换，可能会导致错误（称为字撕裂）。当定义 `long` 和 `double` 变量时，如果使用 `volatile` 关键字，读和写就会获得原子性保证。

`volatile` 还有一个语义是禁止指令重排序优化。先看下面的 `Double Check Lock` 单例实现代码：
```java
public class Singleton {  
    private volatile static Singleton INSTANCE = null;  
    public static Singleton instance() {  
        if(INSTANCE == null) {  
            synchronized (Singleton.class) {  
                if(INSTANCE == null) {  
                    INSTANCE = new Singleton();  
                }  
            }  
        }  
        return INSTANCE;  
    }  
}
```

语句 `INSTANCE = new Singleton();` 会包含至少三步操作：

1. 给 `Singleton` 的实例分配内存。
2. 调用 `Singleton()` 的构造函数，初始化成员变量。
3. 将 `INSTANCE` 指向分配的内存空间，此时 `INSTANCE` 就不是 `null` 了。

如果 `INSTANCE` 不使用 `volatile` 修饰，就有可能发生指令重排序，那么步骤 2 和 3 的顺序就有可能颠倒。如果这种情况下，切换线程，新的线程判断 `INSTANCE` 不为空，不需要获取锁就拿到未初始化完成的对象了，使用 `volatile` 就能避免这种问题。
## synchronized 同步

关键字 synchronized 可以保证在同一时刻，只有一个线程可以执行某一个方法或者某一个代码块。当一个对象被一个线程修改的时候，它能够阻止另一个线程观察到对象内部不一致的状态，并且还可以保证进入同步方法或者代码块的每个线程都能看到由同一个锁保护的之前所有的修改结果。

synchronized 锁是非公平的，可重入的。

synchronized 代码块经过编译之后，会在同步块的前后分别形成 [monitorenter](https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-6.html#jvms-6.5.monitorenter) 和 [monitorexit](https://docs.oracle.com/javase/specs/jvms/se11/html/jvms-6.html#jvms-6.5.monitorexit) 这两个字节码指令。

源码：
```java
    public int add(int a, int b) {
        synchronized(this) {
            return a + b;
        }
    }
```

获取字节码 `javap -c class_file`：
```
	   // ...
       3: monitorenter
       4: iload_1
       5: iload_2
       6: iadd
       7: aload_3
       8: monitorexit
       9: ireturn
      10: astore        4
      12: aload_3
      13: monitorexit
      // ...
```

monitorenter 和 monitorexit 这两个字节码指令都需要一个 reference 类型的参数来指定要锁定和解锁的对象，注意，是引用类型不能是基本类型。如果没有明确指定，那么就根据 synchronized 修饰的是实例方法还是静态方法，去取对应的对象的实例或者 Class 对象来作为锁对象。

每个对象都与一个监视器（可以理解为锁）关联，当监视器具有所有者时，监视器就是被锁定的。

执行 monitorenter 的线程会尝试将自己设置为 reference 对象监视器的所有者（ownership）：

- 如果监视器的 entry count 为 0，则线程将进入监视器并将 entry count 设置为 1。这样线程就是监视器的所有者。
- 如果线程已是监视器的所有者，它将重新进入监视器，并增加 entry count。
- 如果另一个线程已经拥有监视器，则该线程将阻塞，直到监视器的 entry count 为 0，然后再次尝试获取监视器的所有权。

执行 monitorexit 的线程必须是监视器的所有者，否则会抛出 IllegalMonitorStateException 异常。该操作会减少监视器的 entry count，如果减少到 0 则退出监视器。此时，其它线程可以尝试锁定监视器。

synchronized 语法的使用方式：
```java
    void incrAmount(int amount) {
        synchronized (this) {
            ...
        }
    }

	synchronized void incrAmount(int amount) {  
		...
	}

    void incrAmount(int amount) {
        synchronized (App.class) {
            ...
        }
    }

	synchronized static void incrAmount(int amount) {  
        ...  
	}
```


## Lock 同步

### CAS
参考并发控制的笔记，CAS 是实现锁的必要底层指令。

Java 使用 Unsafe 封装了 CAS 的操作。

注意一方面 CAS 提供了原子性操作，不可分割。另一方面 CAS 也具有互斥性，同一时间只有一个线程能够对相同的地址执行 CAS 操作。

### LockSupport
什么线程的阻塞（指锁等待阻塞）和唤醒？本质上应该是操作系统内核停止调度该线程（将当前线程设置为停止调度并立即切换执行的线程）和将线程重新加入调度队列。在[[并发控制]]中介绍过 park 和 unpark 系统调用，Java 使用 LockSupport 来提供同样的功能，底层应该也是通过 JNI 调用系统调用。

通过 CAS 操作和 park/unpark 系统调用，能够实现简单的非可重入锁：
```java
 class FIFOMutex {
   private final AtomicBoolean locked = new AtomicBoolean(false);
   private final Queue<Thread> waiters = new ConcurrentLinkedQueue<>();
     
   public void lock() {
     boolean wasInterrupted = false;
     // publish current thread for unparkers
     waiters.add(Thread.currentThread());
     // Block while not first in queue or cannot acquire lock
     while (waiters.peek() != Thread.currentThread() || !locked.compareAndSet(false, true)) {
       LockSupport.park(this);
       // ignore interrupts while waiting
       if (Thread.interrupted())
         wasInterrupted = true;
     }
     waiters.remove();
     // ensure correct interrupt status on return
     if (wasInterrupted)
       Thread.currentThread().interrupt();
   }
   
   public void unlock() {
     locked.set(false);
     LockSupport.unpark(waiters.peek());
   }
   
   static {
     // Reduce the risk of "lost unpark" due to classloading
     Class<?> ensureLoaded = LockSupport.class;
   }
 }
```


LockSupport.park 方法用来阻塞当前线程，LockSupport.unpark(Thread thread) 用来唤醒一个被阻塞的线程。如果先对一个线程执行 unpark，则紧接下来的 park 调用将直接返回。

public static void park() 将停止当前线程的 CPU 调度，直到下列任何情况发生：

- 另一个线程执行了 unpark 唤醒。
- 另一个线程中断了 park 的线程，对线程调用 interrupt 会使线程从 park 返回。
- 虚假唤醒。

park 调用的返回并不会告诉线程是那种情况返回，所以应该总是重新检查导致线程暂停的条件。
### AbstractQueuedSynchronizer
队列同步器 AbstractQueuedSynchronizer 是用来构建锁和其它同步组件的基础框架。

锁是面向使用者的，它定义了使用者与锁交互的接口，隐藏了实现细节。队列同步器面向的是锁和其它同步组件的，它简化了锁和其它同步组件的实现方式，提供了状态管理、线程排队、等待和唤醒等底层操作。

它使用一个 int 成员变量表示同步状态：
```java
private volatile int state;   // 注意 state 使用 volatile 修饰
protected final int getState() {
    return state;
}

// state 从 0 到 1 的过程永远是 cas 设置的，所以永远只有一个线程能对 state 进行更新，不需要同步
protected final void setState(int newState) {   
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    return U.compareAndSetInt(this, STATE, expect, update);
}
```

通过内置的 FIFO 队列来完成线程的排队工作：
```java
    abstract static class Node {
        volatile Node prev;       // initially attached via casTail
        volatile Node next;       // visibly nonnull when signallable
        Thread waiter;            // visibly nonnull when enqueued
        volatile int status;      // written by owner, atomic bit ops by others
	    // ……
    }

    private transient volatile Node head;

    private transient volatile Node tail;
    
	private boolean casTail(Node c, Node v) {  
	    return U.compareAndSetReference(this, TAIL, c, v);  
	}
```

AQS 使用队列来维护排队的线程，并发数据结构的实现一般会用到锁，但是 AQS 本身就是实现锁的基础组件，在用锁去实现并发队列就死循环了。所以 AQS 使用 CAS 和 volatile 语义来实现并发队列的正确性，非常厉害！需要注意的是只有入队是存在竞态条件的，出队由获取锁的线程负责，不存在竞争。

队列同步器的涉及是基于模板方法，使用者需要继承同步器并重写指定的方法，随后将同步器组合在自定义同步组件的实现中，并调用同步器提供的模板方法，而这些模板方法将会调用使用者重写的方法。

队列同步器可重写的方法：
```java
	/**
	 * 尝试以独占的方式获取锁。这个方法应该查询状态是否允许以独占模式获取它，如果允许则尝试 CAS 设置同步状态。
	 * 这个方法应该始终由执行 acquire 的线程进行调用，如果返回 false 则 acquire 可能会将线程排队。
	 * 队列同步器提供了 acquire 方法，排队，条件判断等工作，将尝试获取锁的操作交给自定义同步组件实现。
	 */	
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
    
	/**
	 * 独占式释放同步状态。
	 */	
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }


    /**
     * 尝试在共享模式下获取锁。返回大于等于 0 的值表示获取成功，反之获取失败。
     */
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * 尝试在共享模式下释放锁。
     */
    protected boolean tryReleaseShared(int arg) {
        throw new UnsupportedOperationException();
    }

    /**
     * 返回当前同步组件是否在独占模式下被线程占用。
     */
    protected boolean isHeldExclusively() {
        throw new UnsupportedOperationException();
    }
```

队列同步器的模板方法基本分为 3 类：独占式获取与释放同步状态、共享式获取与释放同步状态和查询同步队列中等待线程的情况。
```java
    /**
     * 独占模式获取同步状态，忽略中断。如果当前线程设置同步状态成功，则返回；否则线程会进入同步队列等待。
     */
    public final void acquire(int arg) {
        if (!tryAcquire(arg))
            acquire(null, arg, false, false, false, 0L);
    }
    
    /**
     * 与 acquire 相同，但是该方法响应中断。
     */
    public final void acquireInterruptibly(int arg)
        throws InterruptedException {
        if (Thread.interrupted() ||
            (!tryAcquire(arg) && acquire(null, arg, false, true, false, 0L) < 0))
            throw new InterruptedException();
    }

    /**
     * 在 acquireInterruptibly 基础上增加了超时限制，如果当前线程在超时时间内没有设置状态成功，那么将返回 false。
     */
    public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
        if (!Thread.interrupted()) {
            if (tryAcquire(arg))
                return true;
            if (nanosTimeout <= 0L)
                return false;
            int stat = acquire(null, arg, false, true, true,
                               System.nanoTime() + nanosTimeout);
            if (stat > 0)
                return true;
            if (stat == 0)
                return false;
        }
        throw new InterruptedException();
    }
    
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            signalNext(head);
            return true;
        }
        return false;
    }

    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            acquire(null, arg, true, false, false, 0L);
    }
    
    public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
        if (Thread.interrupted() ||
            (tryAcquireShared(arg) < 0 &&
             acquire(null, arg, true, true, false, 0L) < 0))
            throw new InterruptedException();
    }

    public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (!Thread.interrupted()) {
            if (tryAcquireShared(arg) >= 0)
                return true;
            if (nanosTimeout <= 0L)
                return false;
            int stat = acquire(null, arg, true, true, true,
                               System.nanoTime() + nanosTimeout);
            if (stat > 0)
                return true;
            if (stat == 0)
                return false;
        }
        throw new InterruptedException();
    }

    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            signalNext(head);
            return true;
        }
        return false;
    }

	// 获取等待在同步队列上的线程集合
    public final Collection<Thread> getQueuedThreads() {
        ArrayList<Thread> list = new ArrayList<>();
        for (Node p = tail; p != null; p = p.prev) {
            Thread t = p.waiter;
            if (t != null)
                list.add(t);
        }
        return list;
    }
```

AQS 继承了 AbstractOwnableSynchronizer 类，AbstractOwnableSynchronizer 主要的作用是设置锁的当前持有线程，配合 AQS 的 state 实现锁的可重入功能。
```java
public abstract class AbstractOwnableSynchronizer  
    implements java.io.Serializable {  
  
    /** Use serial ID even though all fields transient. */  
    private static final long serialVersionUID = 3737899427754241961L;  
  
    protected AbstractOwnableSynchronizer() { }  
   
    private transient Thread exclusiveOwnerThread;  
	
	protected final void setExclusiveOwnerThread(Thread thread) {  
        exclusiveOwnerThread = thread;  
    }  
  
    protected final Thread getExclusiveOwnerThread() {  
        return exclusiveOwnerThread;  
    }  
}
```

在 ReentrantLock.NonfairSync#initialTryLock 方法中可以看到对 AbstractOwnableSynchronizer 的使用，如果 CAS 成功即获取锁成功，则调用 setExclusiveOwnerThread 方法设置锁的持有线程。
```java
        final boolean initialTryLock() {
            Thread current = Thread.currentThread();
            if (compareAndSetState(0, 1)) { // first attempt is unguarded
                setExclusiveOwnerThread(current);
                return true;
            } else if (getExclusiveOwnerThread() == current) {
                int c = getState() + 1;
                if (c < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(c);
                return true;
            } else
                return false;
        }
```
### Lock 接口
ReentrantLock 的作用和 synchronized 很相似，它们都具备可重入的特性。但是它们有如下区别：

- 一个区别是 RenntranLock 为 API 层面的互斥锁，需要 lock() 和 unlock() 配合 try/finally 语块来完成，而 synchronized 则是语法层面的。
- 另一个区别是，ReentranLock 具备一些高级功能：锁定可中断 lockIntettuptibly()、可是设置等待时间、可实现公平锁，以及锁可以绑定多个条件。

ReentrantLock 实现了 Lock 接口：
```java
public interface Lock {  
	void lock();  
	void lockInterruptibly() throws InterruptedException;  
	
	boolean tryLock();  
	boolean tryLock(long time, TimeUnit unit) throws InterruptedException;  
	
	void unlock(); 
	
	Condition newCondition();  
}
```

ReentrantLock 保护代码块的基本结构：
```java
myLock.lock();
try {
	 // critical section
} finally {
	myLock.unlock();
}
```

!!! tip "为什么 myLock.lock() 语句没有放在 try 代码块中？"
	因为如果在获取锁（自定义锁的实现）时发生了 异常，异常抛出的同时，也会导致锁无故释放。

### ReentrantLock
Sync 继承 AQS，NonfairSync 和 FairSync 继承 Sync，Sync、NonfairSync 和 FairSync 均为 ReentrantLock 的内部类。
![](../LocalFile/Picture/ReentrantLock类图.svg)

ReentrantLock 默认是非公平的，可以通过构造函数指定为公平锁。
```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

ReentrantLock 首次加锁的逻辑：
```java
// 调用 sync.lock() 方法，sync 默认是 NonfairSync 的实例
public void lock() {  
    sync.lock();  
}

// Sync 类的 lock 方法，先尝试 initialTryLock()
@ReservedStackAccess  
final void lock() {  
    if (!initialTryLock())  
        acquire(1);  
}

// NonfairSync 实现了 Sync 定义的 initialTryLock 方法
final boolean initialTryLock() {  
    Thread current = Thread.currentThread();  
    if (compareAndSetState(0, 1)) { // CAS 尝试获取锁，假设第一个线程直接获取成功
        setExclusiveOwnerThread(current);  // 获锁成功的线程直接设置锁的占有线程为当前线程
        return true;  // 返回 true，进而从 ReentrantLock.lock() 返回，继续执行后续代码 
    } else if (getExclusiveOwnerThread() == current) {  // CAS 获取锁失败，判断锁的持有者是不是当前线程
        int c = getState() + 1;  // 如果锁的持有者是当前线程将 state + 1
        if (c < 0) // overflow  
            throw new Error("Maximum lock count exceeded");  
        setState(c);  // 设置 state 的值，因为只有一个线程获取锁，所以这里修改 state 的值不需要额外的锁来保护
        return true;  // 返回 true，进而从 ReentrantLock.lock() 返回
    } else  
        return false;  // 表示当前锁已被其它线程持有
}
```

如果锁已经被其它线程持有，获取锁和释放锁的逻辑：
```java
// 调用 sync.lock() 方法，sync 默认是 NonfairSync 的实例
public void lock() {  
    sync.lock();  
}

// Sync 类的 lock 方法，先尝试 initialTryLock()
@ReservedStackAccess  
final void lock() {  
    if (!initialTryLock())  // 如果锁已经被持有，返回 false，进入 acquire
        acquire(1);  
}

// AQS 的 acquire 方法
public final void acquire(int arg) {  
    if (!tryAcquire(arg))  // 再次尝试获取锁  
        acquire(null, arg, false, false, false, 0L);  
}

// NofairSync 的 tryAcquire 方法
protected final boolean tryAcquire(int acquires) {  
    if (getState() == 0 && compareAndSetState(0, acquires)) {  // 如果此时锁已经被被释放并且 CAS 获取成功的话
        setExclusiveOwnerThread(Thread.currentThread());  // 设置锁的持有线程为当前线程
        return true;  // 返回 true，从 AQS 的 acquire 返回，进而从 ReentrantLock.lock() 返回
    }  
    return false;  // 如果锁继续被其它线程持有，返回 false，进入 AQS 的 acquire 方法
}

// AQS 的 acquire 方法，主要是进行等待队列的处理
final int acquire(Node node, int arg, boolean shared,  
                  boolean interruptible, boolean timed, long time) {  
    Thread current = Thread.currentThread();  
    byte spins = 0, postSpins = 0;   // retries upon unpark of first thread  
    boolean interrupted = false, first = false;  // first 表示当前节点是否是第一个进入等待队列的节点线程
    Node pred = null;               // predecessor of node when enqueued  当前节点的前驱节点
 
    for (;;) {  
        if (!first && (pred = (node == null) ? null : node.prev) != null && !(first = (head == pred))) {  
            if (pred.status < 0) {  
                cleanQueue();           // predecessor cancelled  
                continue;  
            } else if (pred.prev == null) {  
                Thread.onSpinWait();    // ensure serialization  
                continue;  
            }  
        }  
        if (first || pred == null) {  // 进入这里
            boolean acquired;  
            try {  
                if (shared)  
                    acquired = (tryAcquireShared(arg) >= 0);  
                else  
                    acquired = tryAcquire(arg); // 再次进入 NonfairSync 
            } catch (Throwable ex) {  
                cancelAcquire(node, interrupted, false);  
                throw ex;  
            }  
            if (acquired) {  // 如果获取锁成功
                if (first) {  // 如果锁传递给当前的头节点，进行维护队列的操作，队列的并发操作，头节点的释放由锁来保护
                    node.prev = null;  
                    head = node;  
                    pred.next = null;  
                    node.waiter = null;  
                    if (shared)  
                        signalNextIfShared(node);  
                    if (interrupted)  
                        current.interrupt();  
                }  
                return 1;  
            }  
        }  
        Node t;  
        if ((t = tail) == null) {           // initialize queue  第一次循环进入这里，初始化队列
            if (tryInitializeHead() == null)  // 初始化队列，head tail
                return acquireOnOOME(shared, arg);  
        } else if (node == null) {          // allocate; retry before enqueue // 第二次循环进入这里，初始当前节点
            try {  
                node = (shared) ? new SharedNode() : new ExclusiveNode();  
            } catch (OutOfMemoryError oome) {  
                return acquireOnOOME(shared, arg);  
            }  
        } else if (pred == null) {          // try to enqueue  //第三次循环进入这里，设置节点的前驱，并设置当前节点为尾节点
            node.waiter = current;  
            node.setPrevRelaxed(t);         // avoid unnecessary fence  
            if (!casTail(t, node))  // 队列的并发操作，尾节点的添加由 CAS 操作来确保原子性
                node.setPrevRelaxed(null);  // back out  
            else  
                t.next = node;  
        } else if (first && spins != 0) {  
            --spins;                        // reduce unfairness on rewaits  
            Thread.onSpinWait();  
        } else if (node.status == 0) {  // 设置节点的状态为 WAITING
            node.status = WAITING;          // enable signal and recheck  
        } else {  // 最后进入 park
            long nanos;  
            spins = postSpins = (byte)((postSpins << 1) | 1);  
            if (!timed)  
                LockSupport.park(this);  
            else if ((nanos = time - System.nanoTime()) > 0L)  
                LockSupport.parkNanos(this, nanos);  
            else  
                break;  
            node.clearStatus();  
            if ((interrupted |= Thread.interrupted()) && interruptible)  
                break;  
        }  
    }  
    return cancelAcquire(node, interrupted, interruptible);  
}

// AQS 初始化等待队列
private Node tryInitializeHead() {  
    for (Node h = null, t;;) {  
        if ((t = tail) != null)  
            return t;  
        else if (head != null)  
            Thread.onSpinWait();  
        else {  
            if (h == null) {  
                try {  
                    h = new ExclusiveNode();  // 初始化头节点
                } catch (OutOfMemoryError oome) {  
                    return null;  
                }  
            }  
            if (U.compareAndSetReference(this, HEAD, null, h))  // 头节点设置为 HEAD
                return tail = h;  // 尾节点也指向 HEAD
        }  
    }  
}

// ReentrantLock 的 unlock 方法
public void unlock() {  
    sync.release(1);  
}

// 进入到 AQS 里面
public final boolean release(int arg) {  
    if (tryRelease(arg)) {  
        signalNext(head);  
        return true;  
    }  
    return false;  
}

// Sync 的 tryRelease
@ReservedStackAccess  
protected final boolean tryRelease(int releases) {  
    int c = getState() - releases;  
    if (getExclusiveOwnerThread() != Thread.currentThread())  
        throw new IllegalMonitorStateException();  
    boolean free = (c == 0);  
    if (free)  
        setExclusiveOwnerThread(null);  
    setState(c);  
    return free;  
}

// AQS 的唤醒操作，唤醒下一个节点
private static void signalNext(Node h) {  
    Node s;  
    if (h != null && (s = h.next) != null && s.status != 0) {  
        s.getAndUnsetStatus(WAITING);  
        LockSupport.unpark(s.waiter);  
    }  
}
```

公平锁和非公平锁的区别：
```java
static final class NonfairSync extends Sync {  
    private static final long serialVersionUID = 7316153563782823691L;  
  
    final boolean initialTryLock() {  
        Thread current = Thread.currentThread();  
        if (compareAndSetState(0, 1)) { // first attempt is unguarded  非公平锁进来就尝试获取锁，不管当前的状态如何
            setExclusiveOwnerThread(current);  
            return true;  
        } else if (getExclusiveOwnerThread() == current) {  
            int c = getState() + 1;  
            if (c < 0) // overflow  
                throw new Error("Maximum lock count exceeded");  
            setState(c);  
            return true;  
        } else  
            return false;  
    }  
  
    /**  
     * Acquire for non-reentrant cases after initialTryLock prescreen     */    
    protected final boolean tryAcquire(int acquires) {  // 自旋获取锁的时候先判断 state 状态，然后再尝试获取锁
        if (getState() == 0 && compareAndSetState(0, acquires)) {  
            setExclusiveOwnerThread(Thread.currentThread());  
            return true;  
        }  
        return false;  
    }  
}

static final class FairSync extends Sync {  
    private static final long serialVersionUID = -3000897897090466540L;  
  
    /**  
     * Acquires only if reentrant or queue is empty.     */    
    final boolean initialTryLock() {  
        Thread current = Thread.currentThread();  
        int c = getState();  
        if (c == 0) {  // 公平锁首次获取锁的时候判断状态
            if (!hasQueuedThreads() && compareAndSetState(0, 1)) {  // 然后还要判断是否有等待线程
                setExclusiveOwnerThread(current);  
                return true;  
            }  
        } else if (getExclusiveOwnerThread() == current) {  
            if (++c < 0) // overflow  
                throw new Error("Maximum lock count exceeded");  
            setState(c);  
            return true;  
        }  
        return false;  
    }  
  
    /**  
     * Acquires only if thread is first waiter or empty     */    
    protected final boolean tryAcquire(int acquires) {  
        if (getState() == 0 && !hasQueuedPredecessors() &&  // 自旋获取锁的时候也是要判断状态，判断等待线程
            compareAndSetState(0, acquires)) {  
            setExclusiveOwnerThread(Thread.currentThread());  
            return true;  
        }  
        return false;  
    }  
}
```

### ReentrantReadWriteLock
Sync、ReadLock、WriteLock、NonfairSync 和 FairSync 均为 ReentrantReadWriteLock 的内部类。
![](../LocalFile/Picture/ReentrantReadWriteLock类图.svg)
ReadWriteLock 接口的定义：
```java
public interface ReadWriteLock {
    Lock readLock();

    Lock writeLock();
}
```

读写锁的经典用法：
```java
class RWDictionary {
    private final Map<String, Data> m = new TreeMap<>();
    private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private final Lock r = rwl.readLock();
    private final Lock w = rwl.writeLock();

    public Data get(String key) {
        r.lock();
        try {
            return m.get(key);
        } finally {
            r.unlock();
        }
    }

    public List<String> allKeys() {
        r.lock();
        try {
            return new ArrayList<>(m.keySet());
        } finally {
            r.unlock();
        }
    }

    public Data put(String key, Data value) {
        w.lock();
        try {
            return m.put(key, value);
        } finally {
            w.unlock();
        }
    }

    public void clear() {
        w.lock();
        try {
            m.clear();
        } finally {
            w.unlock();
        }
    }
}
```

- 当读写锁处于写加锁状态时，所有试图加锁的线程都会被阻塞。

- 当读写锁处于读加速状态时，所有试图以读模式获取锁的线程都会成功，但是任何希望以写模式加锁的线程都会被阻塞直到所有的线程都释放它们的读锁为止。

读写锁的状态设计
AQS 中只有一个 int 类型的 state 变量表示锁的状态，ReentrantReadWriteLock 使用 state 变量的高 16 位表示读锁的获取（或者重入）的次数，使用 state 变量的低 16 位表示写锁的获取（或者重入）次数。

ReentrantReadWriteLock 中的代码实现：
```java
// 传入 state 获取读锁的获取（或者重入）的次数
static int sharedCount(int c)    { return c >>> 16; }  // 无符号右移，使用 0 填充高位

// 传入 state 获取写锁的获取（或者重入）的次数
static int exclusiveCount(int c) { return c & 0x0000ffff; }
```

读写锁的初始化
```java
    /** Inner class providing readlock */
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** Inner class providing writelock */
    private final ReentrantReadWriteLock.WriteLock writerLock;
    /** Performs all synchronization mechanics */
    final Sync sync;

    public ReentrantReadWriteLock() {
        this(false); // 默认非公平锁
    }

    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
```

写锁的加锁逻辑：
```java
// WriteLock 的 lock 方法
public void lock() {  
    sync.acquire(1);  
}

// 进入到 AQS 的 acquire 方法
public final void acquire(int arg) {  
    if (!tryAcquire(arg))  
        acquire(null, arg, false, false, false, 0L);  
}

// 进入 Sync 的 tryAcquire 方法
protected final boolean tryAcquire(int acquires) {  
    /*  
     * Walkthrough:     
     * 1. If read count nonzero or write count nonzero     
     *    and owner is a different thread, fail.     
     * 2. If count would saturate, fail. (This can only     
     *    happen if count is already nonzero.)     
     * 3. Otherwise, this thread is eligible for lock if     
     *    it is either a reentrant acquire or     
     *    queue policy allows it. If so, update state     
     *    and set owner.     
     * */    
    Thread current = Thread.currentThread();  
    int c = getState();  
    int w = exclusiveCount(c);  // 获取写锁的值
    if (c != 0) {  // 有锁的情况下
        // (Note: if c != 0 and w == 0 then shared count != 0)  
        // w == 0 判断是否有读锁，如果没有读锁表示有写锁，然后判断是否是当前线程持有
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;  
        if (w + exclusiveCount(acquires) > MAX_COUNT)  // 如果前面一个 if 没有进入表明当前线程持有写锁
            throw new Error("Maximum lock count exceeded");  
        // Reentrant acquire  
        setState(c + acquires);  // 前面个两个都过了，表示没有问题，更新 state 的值返回 true
        return true;    
    }  

	// writerShouldBlock 方法，FairSync 锁判断是否有前驱节点，NonfairSync 直接返回 false
	// CAS 进行尝试获取锁
    if (writerShouldBlock() || !compareAndSetState(c, c + acquires))  
        return false;  // 获取锁失败
    setExclusiveOwnerThread(current); // 获取锁成功  
    return true;
}

// 进入 acquire 之后的逻辑和 ReentrantLock 一样了
```

写锁的释放逻辑和 ReentrantLock 一致。

Sync 读锁相关的变量：
```java
        /**
         * A counter for per-thread read hold counts.
         * Maintained as a ThreadLocal; cached in cachedHoldCounter.
         */
        // 记录每个获取读锁的线程重入的次数
        static final class HoldCounter {
            int count;          // 默认初始化为 0
            // Use id, not reference, to avoid garbage retention
            final long tid = LockSupport.getThreadId(Thread.currentThread());
        }

        /**
         * ThreadLocal subclass. Easiest to explicitly define for sake
         * of deserialization mechanics.
         */
        // 自定义 ThreadLock 缓存 HoldCounter
        static final class ThreadLocalHoldCounter
            extends ThreadLocal<HoldCounter> {
            public HoldCounter initialValue() {
                return new HoldCounter();
            }
        }

        /**
         * The number of reentrant read locks held by current thread.
         * Initialized only in constructor and readObject.
         * Removed whenever a thread's read hold count drops to 0.
         */
        private transient ThreadLocalHoldCounter readHolds;

        /**
         * The hold count of the last thread to successfully acquire
         * readLock. This saves ThreadLocal lookup in the common case
         * where the next thread to release is the last one to
         * acquire. This is non-volatile since it is just used
         * as a heuristic, and would be great for threads to cache.
         *
         * <p>Can outlive the Thread for which it is caching the read
         * hold count, but avoids garbage retention by not retaining a
         * reference to the Thread.
         *
         * <p>Accessed via a benign data race; relies on the memory
         * model's final field and out-of-thin-air guarantees.
         */
        
        private transient HoldCounter cachedHoldCounter;

		// 指向第一个获取读锁的线程
        private transient Thread firstReader;
        // 第一个获取读锁的线程重入的次数
        private transient int firstReaderHoldCount;
```

读锁的加锁逻辑：
```java
// ReadLock 的 lock 方法
public void lock() {  
    sync.acquireShared(1);  
}

// 进入 AQS 的 acquireShared 方法
public final void acquireShared(int arg) {  
    if (tryAcquireShared(arg) < 0)  
        acquire(null, arg, true, false, false, 0L);  
}

// 进入 Sync 的 tryAcquireShared 方法
@ReservedStackAccess  
protected final int tryAcquireShared(int unused) {  
    /*  
     * Walkthrough:     
     * 1. If write lock held by another thread, fail.     
     * 2. Otherwise, this thread is eligible for     
     *    lock wrt state, so ask if it should block     
     *    because of queue policy. If not, try     
     *    to grant by CASing state and updating count.     
     *    Note that step does not check for reentrant     
     *    acquires, which is postponed to full version     
     *    to avoid having to check hold count in     
     *    the more typical non-reentrant case.     
     * 3. If step 2 fails either because thread     
     *    apparently not eligible or CAS fails or count     
     *    saturated, chain to version with full retry loop.     
     * */    
    Thread current = Thread.currentThread();  
    int c = getState();  
    // 写锁被持有但是不是当前线程
    if (exclusiveCount(c) != 0 &&  getExclusiveOwnerThread() != current)  
        return -1;  
    int r = sharedCount(c);  // 获取读锁获取次数
    // 尝试获取锁
    if (!readerShouldBlock() &&  
        r < MAX_COUNT &&  
        compareAndSetState(c, c + SHARED_UNIT)) {  
        if (r == 0) {  // 如果当前线程是第一个获取读锁的线程
            firstReader = current;  
            firstReaderHoldCount = 1;  
        } else if (firstReader == current) {  // 第一个获取读锁的线程又获取读锁成功
            firstReaderHoldCount++;  
        } else {  // 当前线程获取读锁成功，当前线程不是第一个获取读锁成功的线程
            HoldCounter rh = cachedHoldCounter;  
            // 获取当前线程的 HoldCounter
            if (rh == null || rh.tid != LockSupport.getThreadId(current))  
                cachedHoldCounter = rh = readHolds.get();  
            else if (rh.count == 0)  
                readHolds.set(rh);  
            rh.count++;  // 当前线程获取读锁的次数加 1
        }  
        return 1;  
    }  
    // 没有获取读锁成功
    return fullTryAcquireShared(current);  
}

final int fullTryAcquireShared(Thread current) {  
    /*  
     * This code is in part redundant with that in     
     * tryAcquireShared but is simpler overall by not     
     * complicating tryAcquireShared with interactions between     
     * retries and lazily reading hold counts.     */    
    HoldCounter rh = null;  
    for (;;) {  // 通过 for 循环一直尝试获取锁
        int c = getState();  
        if (exclusiveCount(c) != 0) {  
            if (getExclusiveOwnerThread() != current)  
                return -1;  
            // else we hold the exclusive lock; blocking here  
            // would cause deadlock.        
        } else if (readerShouldBlock()) {  
            // Make sure we're not acquiring read lock reentrantly  
            if (firstReader == current) {  
                // assert firstReaderHoldCount > 0;  
            } else {  
                if (rh == null) {  
                    rh = cachedHoldCounter;  
                    if (rh == null ||  
                        rh.tid != LockSupport.getThreadId(current)) {  
                        rh = readHolds.get();  
                        if (rh.count == 0)  
                            readHolds.remove();  
                    }  
                }  
                if (rh.count == 0)  
                    return -1;  
            }  
        }  
        if (sharedCount(c) == MAX_COUNT)  
            throw new Error("Maximum lock count exceeded");  
        if (compareAndSetState(c, c + SHARED_UNIT)) {  
            if (sharedCount(c) == 0) {  
                firstReader = current;  
                firstReaderHoldCount = 1;  
            } else if (firstReader == current) {  
                firstReaderHoldCount++;  
            } else {  
                if (rh == null)  
                    rh = cachedHoldCounter;  
                if (rh == null ||  
                    rh.tid != LockSupport.getThreadId(current))  
                    rh = readHolds.get();  
                else if (rh.count == 0)  
                    readHolds.set(rh);  
                rh.count++;  
                cachedHoldCounter = rh; // cache for release  
            }  
            return 1;  
        }  
    }  
}

// ReentrantReadWriteLock
public void unlock() {  
    sync.releaseShared(1);  
}

// AQS 
public final boolean releaseShared(int arg) {  
    if (tryReleaseShared(arg)) {  
        signalNext(head);  
        return true;    }  
    return false;  
}

// Sync
@ReservedStackAccess  
protected final boolean tryReleaseShared(int unused) {  
    Thread current = Thread.currentThread();  
    if (firstReader == current) {  
        // assert firstReaderHoldCount > 0;  
        if (firstReaderHoldCount == 1)  
            firstReader = null;  
        else            
	        firstReaderHoldCount--;  
    } else {  
        HoldCounter rh = cachedHoldCounter;  
        if (rh == null ||  
            rh.tid != LockSupport.getThreadId(current))  
            rh = readHolds.get();  
        int count = rh.count;  
        if (count <= 1) {  
            readHolds.remove();  
            if (count <= 0)  
                throw unmatchedUnlockException();  
        }  
        --rh.count;  
    }  
    for (;;) {  
        int c = getState();  
        int nextc = c - SHARED_UNIT;  
        if (compareAndSetState(c, nextc))  
            // Releasing the read lock has no effect on readers,  
            // but it may allow waiting writers to proceed if            
            // both read and write locks are now free.            
            return nextc == 0;  
    }  
}
```
## 死锁
一般两种方式来避免死锁：

- 避免获取锁的顺序不一致。
- 使用 tryLock 方法来获取锁。