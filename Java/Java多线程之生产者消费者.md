[TOC]

## 一、使用synchronized实现

#### 实现的代码

```java
public class FruitPlateDemo {

    private final static String LOCK = "lock";

    private int count = 0;

    private static final int FULL = 10;

    public static void main(String[] args) {

        FruitPlateDemo fruitDemo1 = new FruitPlateDemo();

        for (int i = 1; i <= 5; i++) {
            new Thread(fruitDemo1.new Producer(), "生产者-" + i).start();
            new Thread(fruitDemo1.new Consumer(), "消费者-" + i).start();
        }

    }

    class Producer implements Runnable {

        @Override
        public void run() {

            for (int i = 0; i < 10; i++) {

                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (LOCK) {

                    while (count == FULL) {
                        try {
                            LOCK.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    count++;
                    System.out.println("生产者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
                    LOCK.notifyAll();

                }

            }

        }

    }

    class Consumer implements Runnable {

        @Override
        public void run() {

            for (int i = 0; i < 10; i++) {

                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (LOCK) {

                    while (count == 0) {
                        try {
                            LOCK.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    count--;
                    System.out.println("消费者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
                    LOCK.notifyAll();

                }

            }

        }

    }

}
```

#### 打印的结果

```
生产者 生产者-1 总共有 1 个资源
消费者 消费者-5 总共有 0 个资源
生产者 生产者-5 总共有 1 个资源
消费者 消费者-4 总共有 0 个资源
生产者 生产者-4 总共有 1 个资源
生产者 生产者-3 总共有 2 个资源
消费者 消费者-2 总共有 1 个资源
消费者 消费者-3 总共有 0 个资源
生产者 生产者-2 总共有 1 个资源
消费者 消费者-1 总共有 0 个资源
生产者 生产者-1 总共有 1 个资源
......
```

可以看出我们使用synchronized可以实现生产者/消费者模型，但是我们得要注意一点，我们在这里使用的 notifyAll() 这个方法，为什么不能用 notify() ，也就是随便叫醒一个消费者呢？答案是不可以，使用 notify() 是叫醒 LOCK 阻塞队列里面的任意一个线程，假如此时我们的临界区域已经满了，此时唤醒的是一个生产者线程，就会导致死锁，所以我们在这里采用的是 notifyAll() 这个方法，意思就是唤醒阻塞队列里面的全部线程，这样某一个消费者就可以去取出临界区里面的产品，从而避免死锁的发生，但是很显然，从上面打印的结果可以看出，顺序是无法保证的，想要保证顺序，可以试着使用可重入锁 ReentrantLock 来实现。

## 二、可重入锁ReentrantLock的实现

#### 实现的代码

```java
public class Demo1 {

    private int count = 0;

    private final static int FULL = 10;

    private Lock lock;

    private Condition notEmptyCondition;

    private Condition notFullCondition;

    {
        lock = new ReentrantLock();
        notEmptyCondition = lock.newCondition();
        notFullCondition = lock.newCondition();

    }

    class Producer implements Runnable {

        @Override
        public void run() {

            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                lock.lock();

                try {

                    while(count == FULL) {

                        try {
                            notFullCondition.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }

                    count++;
                    System.out.println("生产者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
                    notEmptyCondition.signal();


                } finally {
                    lock.unlock();
                }

            }

        }

    }

    class Consumer implements Runnable {

        @Override
        public void run() {

            for (int i = 0; i < 10; i++) {

                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                lock.lock();

                try {

                    while(count == 0) {

                        try {
                            notEmptyCondition.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }

                    count--;
                    System.out.println("消费者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
                    notFullCondition.signal();


                } finally {
                    lock.unlock();
                }


            }

        }

    }

    public static void main(String[] args) {

        Demo1 demo1 = new Demo1();
        for (int i = 1; i <= 5; i++) {
            new Thread(demo1.new Producer(), "生产者-" + i).start();
            new Thread(demo1.new Consumer(), "消费者-" + i).start();
        }

    }

}

```

#### 打印的结果

```
生产者 生产者-1 总共有 1 个资源
生产者 生产者-2 总共有 2 个资源
消费者 消费者-1 总共有 1 个资源
消费者 消费者-2 总共有 0 个资源
生产者 生产者-4 总共有 1 个资源
消费者 消费者-3 总共有 0 个资源
生产者 生产者-3 总共有 1 个资源
消费者 消费者-4 总共有 0 个资源
生产者 生产者-5 总共有 1 个资源
消费者 消费者-5 总共有 0 个资源
......
```

这里我们使用的是 Java 提供的 ReentrantLock 来实现生产者/消费者模型，与 synchronized 相比之下，一个 lock 我们可以生成多个 condition ，换句话说 synchronized 就像是只有一个 condition 的 ReentrantLock ，所以 后者比前者更加的灵活，但是也较为麻烦，因为每次都得手动地关闭锁，所以我们每次得尝试在 finally 里面关闭锁。

## 三、阻塞队列BlockingQueue的实现

#### 实现的代码

```java
public class Demo2 {

    private int count = 0;

    private BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

    public static void main(String[] args) {

        Demo2 demo2 = new Demo2();
        for (int i = 1; i <= 5; i++) {
            new Thread(demo2.new Producer(), "生产者-" + i).start();
            new Thread(demo2.new Consumer(), "消费者-" + i).start();
        }


    }

    class Producer implements Runnable {

        @Override
        public void run() {

            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(300);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                try {
                    queue.put(1);
                    count++;
                    System.out.println("生产者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }

    class Consumer implements Runnable {

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e1) {
                    e1.printStackTrace();
                }
                try {
                    queue.take();
                    count--;
                    System.out.println("消费者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }

}

```

#### 打印的结果

```
生产者 生产者-1 总共有 3 个资源
生产者 生产者-2 总共有 1 个资源
消费者 消费者-3 总共有 0 个资源
消费者 消费者-4 总共有 2 个资源
生产者 生产者-4 总共有 2 个资源
生产者 生产者-3 总共有 1 个资源
生产者 生产者-5 总共有 2 个资源
消费者 消费者-2 总共有 0 个资源
消费者 消费者-1 总共有 1 个资源
消费者 消费者-5 总共有 1 个资源
生产者 生产者-1 总共有 1 个资源
消费者 消费者-3 总共有 1 个资源
......
```

阻塞队列可以借助它本身的性质帮我们实现生产者/消费者模型，在某些情况下，访问队列会造成阻塞，队列被阻塞分为两种情况：

1. 当队列满了的时候进行入队列操作
2. 当队列空了的时候进行出队列操作

因此，当一个线程对已经满了的阻塞队列进行入队操作时会阻塞，除非有另外一个线程进行了出队操作，当一个线程对一个空的阻塞队列进行出队操作时也会阻塞，除非有另外一个线程进行了入队操作。所以阻塞队列本身就可以完成生产者/消费者模型。

## 四、信号量

#### 实现的代码

```java
import java.util.concurrent.Semaphore;

public class test {
  private volatile int count = 0;
  private final Semaphore notFull = new Semaphore(10);
  private final Semaphore notEmpty = new Semaphore(0);
  private final Semaphore mutex = new Semaphore(1);
  public static void main(String[] args) {
    test t = new test();
    for(int i=0;i<5;i++){
      new Thread(t.new Producer(),"生产者-"+i).start();
      new Thread(t.new Customer(),"消费者-"+i).start();
    }

  }
  //生产者
  class Producer implements Runnable{
    @Override
    public void run() {
      for(int i=0;i<10;i++){
        try {
          Thread.sleep(300);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        try {
          notFull.acquire();
          mutex.acquire();
          count ++;
          System.out.println("生产者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
          mutex.release();
          notEmpty.release();
        } catch (InterruptedException e) {
          e.printStackTrace();
        }      
      }
    }
  }
  // 消费者
  class Customer implements Runnable{
    @Override
    public void run() {
      for(int i=0;i<10;i++){
        try {
          Thread.sleep(300);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        try {
          notEmpty.acquire();
          mutex.acquire();
          count --;
          System.out.println("消费者 " + Thread.currentThread().getName() + " 总共有 " + count + " 个资源");
          mutex.release();
          notFull.release();
        } catch (InterruptedException e) {
          e.printStackTrace();
        }      
      }
    }
  }
}

```

Semaphore是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做完自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。Semaphore可以用来构建一些对象池，资源池之类的，比如数据库连接池，我们也可以创建计数为1的Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量，表示两种互斥状态。计数为0的Semaphore是可以release的，然后就可以acquire（即一开始使线程阻塞从而完成其他执行。）。