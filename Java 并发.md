# Java 并发

## 一、基本概念

### 1. 线程状态转换

![img](Java 并发.assets/adfb427d-3b21-40d7-a142-757f4ed73079.png)

1. 新建（New）：创建后尚未启用

2. 可运行（Runnable）：可能正在运行，也可能正在等待CPU时间片(包含了操作系统的线程状态的Running和Ready)

3. 阻塞（Block）:等待获取一个排它锁，如果其线程释放了锁就会结束此状态

4. 无限等待（Waiting）：等待其他线程显式地唤醒，否则不会被分配CPU时间片

   | 进入方法                                       | 退出方法                                 |
   | ---------------------------------------------- | ---------------------------------------- |
   | 没有设置 `timeOut` 参数的 `Object.wait()` 函数 | `Object.notify()` / `Object.notifyAll()` |
   | 没有设置 `timeOut` 参数的 `Thread.join()` 函数 | 被调用的线程执行完毕                     |
   | `LockSupport.park()` 方法                      | `LockSupport.unPark(Thread)`             |

5. 限期等待（Timed Waiting）：无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒

   调用 `Thread.sleep()` 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。

   调用 `Object.wait()` 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。

   睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

   阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 `Thread.sleep()` 和 `Object.wait()` 等方法进入。

   | 进入方法                                     | 退出方法                                            |
   | -------------------------------------------- | --------------------------------------------------- |
   | `Thread.sleep()` 方法                        | 时间结束                                            |
   | 设置了 `TimeOut` 参数的 `Object.wait()` 函数 | 时间结束 / `Object.notify()` / `Object.notifyAll()` |
   | 设置了 `TimeOut` 参数的 `Thread.join()` 函数 | 时间结束 / 线程执行结束                             |
   | `LockSupport.parkNanos()` 方法               | `LockSupport.unpark(Thread)`                        |
   | `LockSupport.parkUntil()` 方法               | `LockSupport.unpark(Thread)`                        |

6. 死亡（Terminated）：线程执行结束 或者 线程发生异常而结束

### 2. 使用线程

有三种使用线程的方法：

- 实现 `Runnable` 接口
- 实现 `Callable` 接口
- 继承 `Thread` 类

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

1. 实现 `Runnable` 接口

   需要实现 `Run()` 方法，通过 `Thread` 调用 `start()` 方法来启动线程

   ```java
package basicJava;
   
   public class ImpRunnable implements Runnable {
       public void run() {
           System.out.println("132");
       }
   
       public static void main(String[] args) {
           ImpRunnable impRunnable = new ImpRunnable();
           Thread thread = new Thread(impRunnable);
   
           thread.start();
       }
   }
   ```
   
2. 实现 `Callable` 接口

   与 `Runnable` 接口相比， `Callable` 接口可以有返回值，返回值通过 `FutureTask` 进行封装

   ```java
   package basicJava;
   
   import java.util.concurrent.Callable;
   import java.util.concurrent.ExecutionException;
   import java.util.concurrent.FutureTask;
   
   public class ImpCallable implements Callable<Integer> {
       public Integer call() throws Exception {
           System.out.println("123");
           return 123;
       }
   
       public static void main(String[] args) throws ExecutionException, InterruptedException {
           ImpCallable impRunnable = new ImpCallable();
   
           FutureTask<Integer> futureTask = new FutureTask<Integer>(impRunnable);
           Thread thread = new Thread(futureTask);
   
           thread.start();
           System.out.println(futureTask.get());
       }
   }
   ```

3. 继承 `Thread` 类

   同样需要实现 `run()` 方法，因为 `Thread` 也实现了 `Runnable` 接口

   当调用 `start()` 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被执行时会执行该线程的 `run()` 方法

   ```java
   package basicJava;
   
   public class ExtThread extends Thread{
   
       @Override
       public void run(){
           System.out.println("123");
       }
   
       public static void main(String[] args) {
           ExtThread extThread = new ExtThread();
   
           extThread.start();
       }
   }
   ```

4. 实现接口 vs 继承 `Thread`

   - 实现接口会好一些，因为Java不允许多重继承，继承了 `Thread` 类后无法继承其他的类，但是java支持实现多个接口
   - 类可能只要求可执行就行，继承整个 `Thread` 类开销过大

### 3. 基础线程机制

1. Executor

   Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。（这里的异步是指多个任务的执行互不干扰，不需要进行同步操作）

   主要有三种 Executor：

   - `CachedThreadPool` :一个任务创建一个线程
   - `FixedThreadPool` :所有任务只能使用固定大小的线程
   - `SingleThreadExecutor`  :相当于大小为 1 的 `FixedThreadPool`

   ```java
   package basicJava;
   
   import java.util.concurrent.ExecutorService;
   import java.util.concurrent.Executors;
   
   public class ExtThread extends Thread{
   
       @Override
       public void run(){
           System.out.println("123");
       }
   
       public static void main(String[] args) {
           ExecutorService executorService = Executors.newCachedThreadPool();
           for(int i = 0 ; i < 5; i++){
               executorService.execute(new ExtThread());
           }
           executorService.shutdown();
       }
   }
   ```

   

2. Daemon