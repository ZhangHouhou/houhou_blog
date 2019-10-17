# Thread及其创建

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

4. **比较**三种方式的区别

   - 实现 Runnable 接口可以避免 Java 单继承特性而带来的局限；增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；适合多个相同程序代码的线程区处理同一资源的情况。
   - 继承 Thread 类和实现 Runnable 方法启动线程都是使用 start() 方法，然后 JVM 虚拟机将此线程放到就绪队列中，如果有处理机可用，则执行 run() 方法。
   - 实现 Callable 接口要实现 call() 方法，并且线程执行完毕后会**有返回值**。其他的两种都是重写 run() 方法，没有返回值。

