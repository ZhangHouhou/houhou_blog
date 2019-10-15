# 概念和thread创建

### 线程基础概念

1. CPU核心数和线程数的关系

   > 是指物理上，也就是硬件上存在几个核心，比如双核CPU就是物理上有两个相对独立的CPU核心单元组。
   > **线程数**是逻辑上的概念，指模拟出来的CPU核心数。在使用超线程技术前，CPU核心数和线程数是1:1。使用超线程技术后，CPU核心数和线程数可以为1:2

2. 时间片轮转机制

   > 每个进程会被分配一个时间片段，称为它的**时间片**，即它所允许运行的时长
   >
   > 如果时间片内，进程还未执行完，CPU会剥夺给另外一个进程。如果进程在时间片内完成或者阻塞，CPU会立即进行切换。CPU调度会维护一个就绪队列，进程用完它的时间片后，会加入队列尾部。显然，这会引起上引文切换。
   > 时间片设值过短会引起大量上下文切换，降低CPU效率。设值过大，会导致响应变差，用户体验差。时间片以略大于一次典型的交互所需要的时间为宜

3. 进程和线程

   **进程**是资源分配的最小单位，一个进程内部有对个线程，共享进程资源
   **线程**是CPU调度的最小单位，必须依赖进程存在(``java.lang.Thread``对象负责统计和控制这种行为)

   ---

   > 进程提供程序执行所需的资源。
   > 它有虚拟地址空间、可执行代码、系统对象的打开句柄、安全上下文、惟一进程标识符、环境变量、优先级类、最小和最大工作集大小，以及至少一个执行的线程。每个进程由一个线程(通常称为**主线程**)启动，但是可以从它的任何线程创建额外的线程。
   >
   > 
   >
   > 线程是进程中可以调度执行的实体。
   > 进程的所有线程共享其虚拟地址空间和系统资源。此外，每个线程维护异常处理程序、调度优先级、线程本地存储、惟一的线程标识符和一组结构，系统将使用这些结构保存线程上下文，直到调度它为止。
   > 线程上下文包括：线程的机器寄存器集、内核堆栈、线程环境块和线程进程地址空间中的用户堆栈。

4. 并行和并发

   > **并行**是同时处理多个事情的能力
   > **并发**是单位时间内处理事情的能力

5. 混淆的概念

   > Q1：怎么区分 Java threads 和 OS threads?
   >
   > A：在Linux上，Java线程是OS线程实现的。因此Java程序使用线程是没有不同于本地程序使用线程。 “Java线程”只是属于一个**JVM**进程的线程
   >
   > Q2：



### 线程的状态

**生命周期**

![](https://images.cnblogs.com/cnblogs_com/houhou929/1561921/o_thread.png)

- New

  > 创建后尚未启动 

- Runnable

  > 可能正在运行，也可能正在等待 CPU 时间片
  > 包含了操作系统线程状态中的 Running 和 Ready

- Blocked

  > 等待获取一个排它锁，如果其线程释放了锁就会结束此状态

- Waiting

  > 等待其它线程显式地唤醒，否则不会被分配 CPU 时间片

  | 进入方法                                   | 退出方法                             |
  | ------------------------------------------ | ------------------------------------ |
  | 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
  | 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
  | LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |

- Timed Waiting

  >无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。
  >
  >调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。
  >
  >调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。
  >
  > 睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。
  >
  >阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入

  | 进入方法                                 | 退出方法                                        |
  | ---------------------------------------- | ----------------------------------------------- |
  | Thread.sleep() 方法                      | 时间结束                                        |
  | 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
  | 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
  | LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
  | LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

- Terminated

  > 可以是线程结束任务之后自己结束，或者产生了**异常**而结束

### thread的构造方式

1. 继承类Thread

   > 通过 Thread 调用 start() 方法来启动线程。

   ```java
   class MyThread extends Thread {
       public void run() {
           System.out.println("MyThread extends Thread...");
       }
   }
   
   public static void main(String[] args) {
       MyThread mt = new MyThread();
        //启动线程
       mt.start();
   }
   ```

   

2. 实现接口Runnable

   > 当调用 start() 方法启动一个线程时，虚拟机会 将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

   ```Java
   class MyRunnable implements Runnable {
       public void run() {
           System.out.println("MyRunnable implements Runnable...");
       }
   }
   
   public static void main(String[] args) {
       MyRunnable instance = new MyRunnable();
       Thread thread = new Thread(instance);
       thread.start();
   }
   ```

   

3. 实现接口Callable

   > 与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装

   ```java
   class MyCallable implements Callable<String> {
       public String call() {
           return "MyCallable implements Callable";
       }
   }
   
   public static void main(String[] args) throws ExecutionException, InterruptedException {
       MyCallable mc = new MyCallable();
       // 获取返回值
       FutureTask<Integer> ft = new FutureTask<>(mc);
       Thread thread = new Thread(ft);
       thread.start();
       System.out.println(ft.get());
   }
   ```