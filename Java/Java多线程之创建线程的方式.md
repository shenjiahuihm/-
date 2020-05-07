## 一、Java种创建线程主要有三种方式

1. 继承Thread类创建线程类

   1. 定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。
   2. 创建Thread子类的实例，即创建了线程对象。
   3. 调用线程对象的start()方法来启动该线程。

   ```
   package com.thread;  
     
   public class FirstThreadTest extends Thread{  
       int i = 0;  
       //重写run方法，run方法的方法体就是现场执行体  
       public void run()  
       {  
           for(;i<100;i++){  
           System.out.println(getName()+"  "+i);  
           }  
       }  
       public static void main(String[] args)  
       {  
           for(int i = 0;i< 100;i++)  
           {  
               System.out.println(Thread.currentThread().getName()+"  : "+i);  
               if(i==20)  
               {  
                   new FirstThreadTest().start();  
                   new FirstThreadTest().start();  
               }  
           }  
       }  
   }
   ```

   

2. 通过Runnable接口创建线程类

   1. 定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。
   2. 创建 Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。
   3. 调用线程对象的start()方法来启动该线程。

   ```
   package com.thread;  
     
   public class RunnableThreadTest implements Runnable  
   {  
     
       private int i;  
       public void run()  
       {  
           for(i = 0;i <100;i++)  
           {  
               System.out.println(Thread.currentThread().getName()+" "+i);  
           }  
       }  
       public static void main(String[] args)  
       {  
           for(int i = 0;i < 100;i++)  
           {  
               System.out.println(Thread.currentThread().getName()+" "+i);  
               if(i==20)  
               {  
                   RunnableThreadTest rtt = new RunnableThreadTest();  
                   new Thread(rtt,"新线程1").start();  
                   new Thread(rtt,"新线程2").start();  
               }  
           }  
       }   
   }
   ```

   线程的执行流程很简单，当执行代码start()时，就会执行对象中重写的void run();方法，该方法执行完成后，线程就消亡了。

3.  通过Callable和Future创建线程

   1. 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
   2. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。（FutureTask是一个包装器，它通过接受Callable来创建，它同时实现了Future和Runnable接口。）
   3. 使用FutureTask对象作为Thread对象的target创建并启动新线程。
   4. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值

```
package com.thread;  
  
import java.util.concurrent.Callable;  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.FutureTask;  
  
public class CallableThreadTest implements Callable<Integer>  
{  
  
    public static void main(String[] args)  
    {  
        CallableThreadTest ctt = new CallableThreadTest();  
        FutureTask<Integer> ft = new FutureTask<>(ctt);  
        for(int i = 0;i < 100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);  
            if(i==20)  
            {  
                new Thread(ft,"有返回值的线程").start();  
            }  
        }  
        try  
        {  
            System.out.println("子线程的返回值："+ft.get());  
        } catch (InterruptedException e)  
        {  
            e.printStackTrace();  
        } catch (ExecutionException e)  
        {  
            e.printStackTrace();  
        }  
  
    }  
  
    @Override  
    public Integer call() throws Exception  
    {  
        int i = 0;  
        for(;i<100;i++)  
        {  
            System.out.println(Thread.currentThread().getName()+" "+i);  
        }  
        return i;  
    }  
  
}
```

## 创建线程的三种方式的对比

1、采用实现Runnable、Callable接口的方式创建多线程时，

优势是：

线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。

在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。

劣势是：

编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

2、使用继承Thread类的方式创建多线程时，

优势是：

编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。

劣势是：

线程类已经继承了Thread类，所以不能再继承其他父类。

3、Runnable和Callable的区别

(1) Callable规定（重写）的方法是call()，Runnable规定（重写）的方法是run()。

(2) Callable的任务执行后可返回值，而Runnable的任务是不能返回值的。

(3) call方法可以抛出异常，run方法不可以。

(4) 运行Callable任务可以拿到一个Future对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。

## 二、 创建四种线程的方式

### 1. **new Thread的弊端**

执行一个异步任务你还只是如下new Thread吗？

```java
new Thread(new Runnable() {
 
 
  @Override
 
  public void run() {
 
    // TODO Auto-generated method stub
 
    }
 
  }
 
).start();
```

那你就out太多了，new Thread的弊端如下：

1. 每次new Thread**新建对象性能差**。
2. 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或oom（out of memory）。
3. 缺乏更多功能，如定时执行、定期执行、线程中断。

相比new Thread，Java提供的四种线程池的好处在于：

1. **重用存在的线程，减少对象创建、消亡的开销，性能佳**。
2. 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。
3. 提供**定时执行、定期执行、单线程、并发数控制**等功能。

### 2. **Java 线程池**

Java通过Executors提供四种线程池，分别为：

newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

(1) newCachedThreadPool：

创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。示例代码如下：

```java
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
 
  for (int i = 0; i < 10; i++) {
 
    final int index = i;
 
  try {
 
    Thread.sleep(index * 1000);
 
   } catch (InterruptedException e) {
 
      e.printStackTrace();
 
  }
 
 
 
   cachedThreadPool.execute(new Runnable() {
 
     @Override
 
     public void run() {
 
        log.info(index);
 
      }
 
   });
 
}
```

线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

(2) newFixedThreadPool：---  需要指定线程池的大小

创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。示例代码如下：

```java

ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
 
  for (int i = 0; i < 10; i++) {
 
  final int index = i;
 
 
  fixedThreadPool.execute(new Runnable() {
 
 
  @Override
 
  public void run() {
 
      try {
         log.info(index);
         Thread.sleep(2000);
      } catch (InterruptedException e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
       }
    }
  });
}
```

因为线程池大小为3，每个任务输出index后sleep 2秒，所以每两秒打印3个数字。

定长线程池的大小最好根据系统资源进行设置。如Runtime.getRuntime().availableProcessors()。可参考PreloadDataCache。

(3)newScheduledThreadPool：

创建一个定长线程池，**支持定时及周期性任务执行**。延迟执行示例代码如下：

```java
ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
 
 scheduledThreadPool.schedule(new Runnable() {
 
 
      @Override
 
      public void run() {
 
         log.info("delay 3 seconds");
       }
 
 }, 3, TimeUnit.SECONDS);
```

表示延迟3秒执行。

定期执行示例代码如下：

```java
scheduledThreadPool.scheduleAtFixedRate(new Runnable() {
 
      @Override
 
      public void run() {
 
          log.info("delay 1 seconds, and excute every 3 seconds");
 
      }
 
}, 1, 3, TimeUnit.SECONDS);
```

表示延迟1秒后每3秒执行一次。

ScheduledExecutorService比Timer更安全，功能更强大

(4)newSingleThreadExecutor：

创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。示例代码如下：

```java
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
 
for (int i = 0; i < 10; i++) {
  final int index = i;
  singleThreadExecutor.execute(new Runnable() {
 
    @Override
    public void run() {
    try {
        log.info(index);
        Thread.sleep(2000);
     } catch (InterruptedException e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
     }
    }
  });
}
```

结果依次输出，相当于顺序执行各个任务。

**线程池的作用：**

线程池作用就是限制系统中执行线程的数量。

根据系统的环境情况，可以自动或手动设置线程数量，达到运行的最佳效果；少了浪费了系统资源，多了造成系统拥挤效率

不高。用线程池控制线程数量，其他线程排队等候。一个任务执行完毕，再从队列的中取最前面的任务开始执行。若队列中没

有等待进程，线程池的这一资源处于等待。当一个新任务需要运行时，如果线程池中有等待的工作线程，就可以开始运行了；

否则进入等待队列。

### 为什么要用线程池:

1.减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。

2.可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大

约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的

线程池接口是ExecutorService。

### 比较重要的几个类：

ExecutorService： 真正的线程池接口。

ScheduledExecutorService： 能和Timer/TimerTask类似，解决那些需要任务重复执行的问题。

ThreadPoolExecutor： ExecutorService的默认实现。

ScheduledThreadPoolExecutor： 继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此

在Executors类里面提供了一些静态工厂，生成一些常用的线程池。

## 线程池的原理

- 提交任务
- 核心线程池（corePoolSize）是否已经满，如果未满的话就创建线程执行任务
- 否则查看队列（BlockingQueue）是否已满，未满的话，将任务存储在队列里
- 如果已经满了，看线程池（maximumPoolSize）是否已满，如果满的话按照拒绝处理任务策略（handler）处理无法执行的任务
- 如果未满，创建线程执行任务

## ThreadPoolExecutor构造参数

- corePoolSize：核心池的大小，构建线程池后，并不会创建线程，当前线程数如果小于corePoolSize时，当要执行任务时，创建一个线程。当当前线程数等于corePoolSize，会将任务放入队列中
- maximumPoolSize：线程池最大数，也就是线程最多能创建的线程
- keepAliveTime：工作线程空闲后，保持存活的时间。默认情况下，如果当前线程数大于corePoolSize，那么一个线程如果没有任务，当空闲的时间大于keepAliveTime时，会终止该线程，直到线程数不超过corePoolSize
- workQueue：存储任务的队列，有几种种类型队列ArrayBlockingQueue（有界缓冲区，基于数组的队列，先进先出，必须指定大小，可以设置是否保持公平，以FIFO顺序访问），LinkedBlockingQueue（基于链表的队列，如果没有指定大小，默认为Integer.MAX_VALUE），SynchronousQueue(无界线程池，不管多少任务提交进来，直接运行)
- ThreadFactory：线程工厂，用来创建线程,通过线程工厂可以给创建的线程设置名字
- rejectedExecutionHandler：拒绝处理任务的策略AbortPolicy（直接放弃任务，抛出RejectedExecutionException异常），DiscardPolicy（放弃任务，不抛出异常），DiscardOldestPolicy（放弃最旧的未处理请求，然后重试 execute；如果执行程序已关闭，则会丢弃该任务），CallerRunsPolicy（它直接在 execute 方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务）

## ThreadPoolExecutor重要方法

- execute：在将来某个时间执行给定任务
- submit：提交一个任务用于执行，并返回一个表示该任务的 Future，和execute不同的是返回的是一个Future，可以在任务执行完毕之后得到任务执行结果
- shutdown：按过去执行已提交任务的顺序发起一个有序的关闭，但是不接受新任务。也就是说，中断没有正在执行任务的线程，等待任务执行完毕。
- shutdownnow： 尝试停止所有的活动执行任务、暂停等待任务的处理，并返回等待执行的任务列表。

## 线程池状态

```java
volatile int runState;
static final int RUNNING    = 0;
static final int SHUTDOWN   = 1;
static final int STOP       = 2;
static final int TERMINATED = 3;
```

- RUNNING:创建线程池后，初始状态为RUNNING
- SHUTDOWN：执行shutdown方法后，线程池处于SHUTDOWN状态
- STOP：执行shutdownNow方法后，线程池处于STOP状态
- TERMINATED：当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

