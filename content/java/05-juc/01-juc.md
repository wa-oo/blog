---
title: JUC 并发编程
date: 2021-05-15
categories: [JUC]
toc: true
---

## 1.什么是 JUC

> java util concurrent
>
> java util concurrent atomic
>
> java util concurrentLocks

**Runnable**：没有返回值、效率相对于 Callable 较低！

## 2.进程和线程回顾

**进程**：一个程序

一个进程可以包含多个线程，至少包含一个！

Java 默认有几个线程？2 个，main，GC

**线程**：开了个进程 Typora，写字自动保存（线程负责）

对于 Java 而言：Thread、Runnable、Callable

**Java 真的可以开启线程吗？**开不了

```java
		public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
		// 本地方法，调用了底层的C++，Java无法操作硬件
    private native void start0();
```

> 并发、并行

并发：多线程操作同一个资源

并行：多个人一起行走（多个线程同时进行）

```java
package com.allen.juc;

/**
 * Created by Allen on 2021/05/15
 */
public class test {
    public static void main(String[] args) {
        // 获取cpu的核数
        // CPU 密集型，IO 密集型
        System.out.println(Runtime.getRuntime().availableProcessors());
    }
}

```

并发编程的本质：**充分利用 CPU 的资源**

> 线程有几个状态

```java
public enum State {
    // 新生
    NEW,

    // 运行
    RUNNABLE,

    // 阻塞
    BLOCKED,

    // 等待，死死的等
    WAITING,

    // 超时等待
    TIMED_WAITING,

    // 终止
    TERMINATED;
}

```

> wait/sleep 区别

1. 来自不同的类

   wait => Object

   sleep => Thread

2. 关于锁的释放

   wait 会释放锁，sleep 不释放锁

3. 使用的范围是不同的

   wait：必须在同步代码块中

   sleep：可以在任何地方用

## 3.Lock 锁（重点）

> 传统 Synchronized

```java
/**
 * 线程就是一个单独的资源类，没有任何附属的操作
 * 属性、方法
 * Created by Allen on 2021/05/15
 */
public class SaleTicketDemo01 {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        // 并发，多线程操作同一个资源类，把资源类丢入线程
        new Thread(() -> {
            for (int i = 0; i < 60; i++) {
                ticket.sale();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 60; i++) {
                ticket.sale();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 60; i++) {
                ticket.sale();
            }
        }, "C").start();
    }
}

// 资源类 OOP
class Ticket {
    // 属性、方法
    private int number = 50;

    // 买票方式
    // synchronized 本质是队列，主要锁对象和Class
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了" + number-- + "票，剩余了" + number);
        }
    }
}
```

> Lock

![image-20210517195420806](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210517195420806.png)

![image-20210517195520961](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210517195520961.png)

![image-20210517195725802](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210517195725802.png)

公平锁：十分公平，先来后到

非公平锁：不公平，可以插队（默认）

```java
public class SaleTicketDemo02 {
    public static void main(String[] args) {
        Ticket2 ticket = new Ticket2();
        // 并发，多线程操作同一个资源类，把资源类丢入线程
        new Thread(() -> { for (int i = 0; i < 60; i++) ticket.sale(); }, "A").start();
        new Thread(() -> { for (int i = 0; i < 60; i++) ticket.sale(); }, "B").start();
        new Thread(() -> { for (int i = 0; i < 60; i++) ticket.sale(); }, "C").start();

    }
}

// 资源类 OOP
class Ticket2 {
    // 属性、方法
    private int number = 50;

    Lock lock = new ReentrantLock();

    // 买票方式
    public  void sale() {
        lock.lock(); // 加锁

        try {
            // 业务代码
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出了" + number-- + "票，剩余了" + number);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock(); // 解锁
        }
    }
}
```

> Synchronized 和 Lock 区别

1. Synchronized 内置的 java 关键字，Lock 是一个 Java 类
2. Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
3. Synchronized 会自动释放锁，Lock 必须要手动释放锁，如果不释放锁，死锁
4. Synchronized 线程 1（获得锁，等待）、线程 2（等待，傻傻的等），Lock 不一定会一直等待（tryLock()）
5. Synchronized 可重入锁，不可以中断的，非公平；Lock，可重入锁，可以判断锁，非公平（可以自己设置）
6. Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码

## 4.生产者和消费者

> 生产者和消费者 Synchronized 版

```java
/**
 * 线程之间的通信问题：生产者和消费者问题; 等待唤醒，通知唤醒
 * 线程交替执行 A B 操作同一个变量 num = 0
 * A num + 1
 * B num - 1
 * Created by Allen on 2021/05/17
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

// 等待，业务，通知
class Data { // 数字 资源类

    private int number = 0;

    // +1
    public synchronized void increment() throws InterruptedException {
        if (number != 0) {
            // 等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + " ==> " + number);
        // 通知其他线程，+1 完毕
        this.notifyAll();
    }

    // -1
    public synchronized void decrement() throws InterruptedException {
        if (number == 0) {
            // 等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + " ==> " + number);
        // 通知其他线程，-1 完毕
        this.notifyAll();
    }
}
```

> 问题存在，A B C D 个线程！虚假唤醒

![image-20210517210042668](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210517210042668.png)

**if 改为 while 判断**

```java
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

// 等待，业务，通知
class Data { // 数字 资源类

    private int number = 0;

    // +1
    public synchronized void increment() throws InterruptedException {
        while (number != 0) {
            // 等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + " ==> " + number);
        // 通知其他线程，+1 完毕
        this.notifyAll();
    }

    // -1
    public synchronized void decrement() throws InterruptedException {
        while (number == 0) {
            // 等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + " ==> " + number);
        // 通知其他线程，-1 完毕
        this.notifyAll();
    }
}
```

> JUC 版生产者和消费者问题

![image-20210517220218085](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210517220218085.png)

通过 Lock 找到 Conduction

```java
public class B {
    public static void main(String[] args) {
        Data2 data = new Data2();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.increment();
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.decrement();
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.increment();
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) data.decrement();
        }, "D").start();
    }
}

// 等待，业务，通知
class Data2 { // 数字 资源类

    private int number = 0;

    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    // condition.await(); // 等待
    // condition.signalAll(); // 唤醒全部

    // +1
    public void increment() {
        lock.lock();
        try {
            while (number != 0) {
                // 等待
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + " ==> " + number);
            // 通知其他线程，+1 完毕
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    // -1
    public void decrement() {
        lock.lock();
        try {
            while (number == 0) {
                // 等待
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + " ==> " + number);
            // 通知其他线程，-1 完毕
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

> Condition 精准的通知和唤醒线程

![image-20210517214042368](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210517214042368.png)

代码测试：

```java
/**
 * A 执行完调用 B，B 执行完调用 C，C 执行完调用 A
 * Created by Allen on 2021/05/17
 */
public class C {
    public static void main(String[] args) {
        Data3 data = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                data.printC();
            }
        }, "C").start();
    }
}

class Data3{
    private int num = 1; // 1A 2B 3C
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    public void printA(){
        lock.lock();
        try {
            // 业务，判断 -> 执行 -> 通知
            while (num != 1){
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + " ==> AAA");
            // 唤醒指定的人，B
            num = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB(){
        lock.lock();
        try {
            // 业务，判断 -> 执行 -> 通知
            while (num != 2){
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName() + " ==> BBB");
            // 唤醒指定的人，C
            num = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC(){
        lock.lock();
        try {
            // 业务，判断 -> 执行 -> 通知
            while (num != 3){
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName() + " ==> CCC");
            // 唤醒指定的人，A
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    // 生产线：下单 -> 支付 -> 交易 -> 物流
}
```

## 5.8 锁的现象

锁是什么，如何判断锁的是谁？

```java
/**
 * 8锁就是关于锁的8个问题
 * 1、标准情况下，两个线程先打印 发短信 还是 打电话？1发短信，2打电话
 * 2、sendSms 延迟4秒，两个线程先打印 发短信 还是 打电话？1发短信，2打电话
 * Created by Allen on 2021/05/17
 */
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();

        // 锁的存在
        new Thread(() -> {
            phone.sendSms();
        }, "A").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
            phone.call();
        }, "B").start();
    }
}

class Phone {

    // synchronized 锁的对象是方法的调用者！
    // 两个方法用的是同一个锁，谁先拿到谁执行
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

```java
/**
 * 3、增加了一个普通方法后，先执行 发短信 还是 hello？1hello，2发短信
 * 4、两个对象，两个同步方法，先执行 发短信 还是 打电话？1打电话，2发短信
 * Created by Allen on 2021/05/17
 */
public class Test2 {
    public static void main(String[] args) throws InterruptedException {

        Phone2 phone1 = new Phone2();
        Phone2 phone2 = new Phone2();

        // 锁的存在
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}

class Phone2 {

    // synchronized 锁的对象是方法的调用者！
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }

    // 这里没有锁！不是同步方法，不受锁的影响！
    public void hello() {
        System.out.println("hello");
    }
}
```

```java
/**
 * 5、增加两个静态的同步方法，只有一个对象，先打印 发短信 还是 打电话？1发短信，2打电话
 * 6、两个对象，增加两个静态的同步方法，先打印 发短信 还是 打电话？1发短信，2打电话
 * Created by Allen on 2021/05/17
 */
public class Test3 {
    public static void main(String[] args) throws InterruptedException {

        // 两个对象的Class类模版只有一个，static，锁的是Class
        Phone3 phone1 = new Phone3();
        Phone3 phone2 = new Phone3();

        // 锁的存在
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}

// Phone3唯一的一个class对象
class Phone3 {

    // synchronized 锁的对象是方法的调用者！
    // static 静态方法
    // 类一加载就有了！锁的是Class
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public static synchronized void call() {
        System.out.println("打电话");
    }

}
```

```java
/**
 * 7、一个静态的同步方法，一个普通的同步方法，一个对象，先打印 发短信 还是 打电话？1打电话，2发短信
 * 8、一个静态的同步方法，一个普通的同步方法，两个对象，先打印 发短信 还是 打电话？1打电话，2发短信
 * Created by Allen on 2021/05/17
 */
public class Test4 {
    public static void main(String[] args) throws InterruptedException {

        // 两个对象的Class类模版只有一个，static，锁的是Class
        Phone4 phone1 = new Phone4();
        Phone4 phone2 = new Phone4();

        // 锁的存在
        new Thread(() -> {
            phone1.sendSms();
        }, "A").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
            phone2.call();
        }, "B").start();
    }
}

// Phone3唯一的一个class对象
class Phone4 {

    // 静态的同步方法，锁的是 Class 类模版
    public static synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 普通的同步方法，锁的是调用者
    public synchronized void call() {
        System.out.println("打电话");
    }

}
```

> 小结

new：this，具体的一个对象

static：Class，唯一的一个模版

## 6.集合类不安全

> List 不安全

```java
/**
 * java.util.ConcurrentModificationException 并发修改异常
 * Created by Allen on 2021/05/18
 */
public class ListTest {
    public static void main(String[] args) {
        // 并发下ArrayList不安全
        /**
         * 解决方案：
         * 1、List<String> list = new Vector<>();
         * 2、List<String> list = Collections.synchronizedList(new ArrayList<>());
         * 3、List<String> list = new CopyOnWriteArrayList<>();
         */
        // CopyOnWrite 写入时复制 COW 计算机程序设计领域的一种优化策略
        // 多个线程调用的时候，list，读取的时候，固定的，写入（覆盖）
        // 在写入的时候避免覆盖，造成数据问题
        // 读写分离
        /**
         * CopyOnWriteArrayList 比 Vector 好在哪里？
         * Vector 使用 synchronized，效率低
         * CopyOnWriteArrayList 使用 Lock，效率高
         */
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 1; i < 10; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(list);
            }).start();
        }
    }
}
```

![image-20210518081418293](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210518081418293.png)

> Set 不安全

```java
/**
 * 同理可证：ConcurrentModificationException
 * Created by Allen on 2021/05/18
 */
public class SetTest {
    public static void main(String[] args) {
        /**
         * 解决方案：
         * 1、Set<Object> set = Collections.synchronizedSet(new HashSet<>());
         * 2、Set<Object> set = new CopyOnWriteArraySet<>();
         */
        Set<Object> set = new CopyOnWriteArraySet<>();
        for (int i = 1; i < 10; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0, 5));
            }, String.valueOf(i)).start();
        }
    }
}
```

### HashSet 底层时什么？

```java
public HashSet() {
    map = new HashMap<>();
}
// add set 本质就是map key是无法重复的！
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
private static final Object PRESENT = new Object(); // 常量，不变的值
```

> Map 不安全

```java
public class MapTest {
    public static void main(String[] args) {
        // map 是这样用的吗？不是
        // 默认等价于什么？new HashMap<>(16, 0.75);
        // 加载因子，初始化容量
        Map<String, String> map = new ConcurrentHashMap<>();
        for (int i = 1; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,5));
                System.out.println(map);
            }).start();
        }
    }
}
```

## 7.Callable

![image-20210518203631579](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210518203631579.png)

1. 可以有返回值
2. 可以抛出异常
3. 方法不同，run() / call()

> 代码测试

![image-20210518211753084](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210518211753084.png)

```java
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        MyThread myThread = new MyThread();
        // 适配类
        FutureTask futureTask = new FutureTask(myThread);
        new Thread(futureTask, "A").start();
        new Thread(futureTask, "B").start(); // 结果会被缓存，效率高
        // 获取 callable 返回结果
      	// 这个get方法可能会产生阻塞！（把它放到最后或者使用异步通信处理）
        Integer o = (Integer)futureTask.get();
        System.out.println(o);
    }
}

class MyThread implements Callable<Integer> {

    @Override
    public Integer call() {
        System.out.println("call()");
        return 1024;
    }
}
```

细节：

1. 有缓存
2. 结果可能需要等待，会阻塞

## 8.常用的辅助类

### CountDownlatch（重点）

![image-20210518212838301](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210518212838301.png)

```java
/**
 * 计数器
 * Created by Allen on 2021/05/18
 */
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 总数是6，必须要执行任务的时候，再使用
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " Go out");
                countDownLatch.countDown(); // 数量-1
            }, String.valueOf(i)).start();
        }
        // 等待技术器归零，然后再向下执行
        countDownLatch.await();
        System.out.println("Close Door");
    }
}
```

**原理**：

countDownLatch.countDown(); // 数量-1

countDownLatch.await(); // 等待技术器归零，然后再向下执行

每次有线程调用 countDown() 数量 -1，假设计数器变为 0，countDownLatch.await() 就会被唤醒，继续执行！

### CyclicBarrier

![image-20210518214544591](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210518214544591.png)

```java
public class CyclicBarrierDemo {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("召唤神龙成功！");
        });
        for (int i = 1; i <= 7; i++) {
            final int temp = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "收集" + temp + "个龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### Semaphore

Semaphore：信号量

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        // 线程数量
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    // acquire() 得到
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "抢到");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName() + "离开");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // release() 释放
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```

**原理**：

semaphore.acquire(); 获得，假设如果已经满了，等待，等待被释放为止！

semaphore.release(); 释放，会将当前的信号释放 +1，然后唤醒等待的线程！

作用：多个共享资源互斥的使用！并发限流，控制最大的线程数！

## 9.读写锁

![image-20210525205615906](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210525205615906.png)

```java
/**
 * 独占锁（写锁）一次只能被一个线程占有
 * 共享锁（读锁）多个线程可以同时占有
 * ReadWriteLock
 * 读-读：可以共存
 * 读-写：不能共存
 * 写-写：不能共存
 * Created by Allen on 2021/05/25
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCacheLock myCache = new MyCacheLock();

        // 写入
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.put(temp + "", temp);
            }, String.valueOf(i)).start();
        }

        // 读取
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCache.get(temp + "");
            }, String.valueOf(i)).start();
        }
    }
}

// 加锁的
class MyCacheLock {
    private volatile Map<String, Object> map = new HashMap<>();
    // 读写锁：更加细粒度的控制
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    // 存，写，写入的时候只希望同时只有一个线程写
    public void put(String key, Object value){
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "写入" + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写入OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }

    // 取，读，所有人都可以读
    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取" + key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}


class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();

    // 存，写
    public void put(String key, Object value){
        System.out.println(Thread.currentThread().getName() + "写入" + key);
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "写入OK");
    }

    // 取，读
    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "读取" + key);
        Object o = map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取OK");
    }
}
```

## 10.阻塞队列

**FIFO**：先进先出

![image-20210525210307977](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210525210307977.png)

阻塞队列：

![image-20210525210508799](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210525210508799.png)

**BlockingQueue**：不是新的东西

![image-20210526205541588](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210526205541588.png)

什么情况下会使用阻塞队列：多线程并发处理，线程池！

**学会使用队列**

添加、移除

**四组 API**

|     方式     | 抛出异常 | 有返回值，不抛出异常 | 阻塞等待 | 超时等待  |
| :----------: | :------: | :------------------: | :------: | :-------: |
|     添加     |   add    |        offer         |   put    | offer(,,) |
|     移除     |  remove  |         poll         |   take   |  poll(,)  |
| 检测队首元素 | element  |         peek         |    -     |     -     |

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        test3();
    }

    /**
     * 抛出异常
     */
    public static void test1() {
        // 队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("b"));
        System.out.println(blockingQueue.add("c"));
        // java.lang.IllegalStateException: Queue full 抛出异常！
        // System.out.println(blockingQueue.add("d"));

        System.out.println(blockingQueue.element()); // 查看队首元素是谁

        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        // java.util.NoSuchElementException 抛出异常！
        System.out.println(blockingQueue.remove());
    }

    /**
     * 有返回值，没有异常
     */
    public static void test2() {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("c")); // false 不抛出异常！

        System.out.println(blockingQueue.peek()); // 查看队首元素是谁

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll()); // null 不抛出异常！
    }

    /**
     * 等待，阻塞（一直阻塞）
     */
    public static void test3() throws InterruptedException {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        // 一直阻塞
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        // blockingQueue.put("d"); // 队列没有位置了，一直阻塞

        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take()); // 没有元素，一直阻塞
    }

    /**
     * 等待，阻塞（等待超时）
     */
    public static void test4() throws InterruptedException {
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("a"));
        // 等待超过2秒就退出
        System.out.println(blockingQueue.offer("d", 2, TimeUnit.SECONDS));

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        // 等待超过2秒就退出
        System.out.println(blockingQueue.poll(2, TimeUnit.SECONDS));
    }
}
```

> SynchronousQueue 同步队列

没有容量

进去一个元素，必须等待取出来之后，才能再往里面放一个元素。

```java
package com.allen.bq;

import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

/**
 * 同步队列
 * 和其他的 BlockingQueue 不一样，SynchronousQueue 不存储元素
 * put了一个元素，必须从里面先take取出来，否则不能再put进去值
 * Created by Allen on 2021/7/14
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        SynchronousQueue<String> synchronousQueue = new SynchronousQueue<>();

        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + " put 1");
                synchronousQueue.put("1");
                System.out.println(Thread.currentThread().getName() + " put 2");
                synchronousQueue.put("2");
                System.out.println(Thread.currentThread().getName() + " put 3");
                synchronousQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "T1").start();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + " => " + synchronousQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + " => "  + synchronousQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + " => "  + synchronousQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "T2").start();
    }
}
```

## 11.线程池(重点)

线程池：**三大方法、七大参数、四种拒绝策略**

> 池化技术

程序的运行，本质：占用系统的资源！优化资源的使用！=> 池化技术

线程池、连接池、内存池、对象池......（创建、销毁。十分浪费资源）

池化技术：事先准备好一些资源，有人要用，就来池里拿，用完之后还给池

**线程池的好处**：

1、降低资源的消耗

2、提高响应的速度

3、方便管理

**线程复用、可以控制最大并发数、管理线程**

### 三大方法

![](D:\SynologyDrive\allen\img\Snipaste_2021-08-12_15-15-22.png)

```java
package com.allen.pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Executors 工具累、3大方法
 * Created by Allen on 2021/7/14
 */
public class Demo01 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();// 单个线程
//        ExecutorService threadPool = Executors.newFixedThreadPool(5); // 创建一个固定的线程池的大小
        ExecutorService threadPool = Executors.newCachedThreadPool(); // 可伸缩的，遇强则强，遇弱则弱

        try {
            for (int i = 0; i < 10; i++) {
                // 使用了线程池之后，使用线程池创建线程
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

### 七大参数

源码分析：

```java
public static ExecutorService newSingleThreadExecutor() {
    return new Executors.FinalizableDelegatedExecutorService(
        new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue()));
}
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue());
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, 2147483647, 60L, TimeUnit.SECONDS, new SynchronousQueue());
}

// 本质ThreadPoolExecutor()
public ThreadPoolExecutor(int corePoolSize,  //核心线程池大小
                          int maximumPoolSize,  // 最大核心线程池大小
                          long keepAliveTime,  // 超时了没有人调用就会释放
                          TimeUnit unit,  // 超时单位
                          BlockingQueue<Runnable> workQueue,  // 阻塞队列
                          ThreadFactory threadFactory,  // 线程工厂，创建线程的，一般不用动
                          RejectedExecutionHandler handler // 拒绝策略) {
    this.ctl = new AtomicInteger(ctlOf(-536870912, 0));
    this.mainLock = new ReentrantLock();
    this.workers = new HashSet();
    this.termination = this.mainLock.newCondition();
    if (corePoolSize >= 0 &&
        maximumPoolSize > 0 &&
        maximumPoolSize >= corePoolSize &&
        keepAliveTime >= 0L) {
        if (workQueue != null && threadFactory != null && handler != null) {
            this.corePoolSize = corePoolSize;
            this.maximumPoolSize = maximumPoolSize;
            this.workQueue = workQueue;
            this.keepAliveTime = unit.toNanos(keepAliveTime);
            this.threadFactory = threadFactory;
            this.handler = handler;
        } else {
            throw new NullPointerException();
        }
    } else {
        throw new IllegalArgumentException();
    }
}
```

![Snipaste_2021-08-12_15-26-35](D:\SynologyDrive\allen\img\Snipaste_2021-08-12_15-26-35.png)

![](D:\SynologyDrive\allen\img\Snipaste_2021-08-12_15-31-09.png)

### 手动创建线程池

```java
package com.allen.pool;

import java.util.concurrent.*;

public class Demo01 {
    public static void main(String[] args) {
        // 自定义线程池 ThreadPoolExecutor
        ExecutorService threadPool = new ThreadPoolExecutor(2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy()); // 队列满了，抛掉最早的那个，代替它的位置进入队列里

        try {
            // 最大承载：Deque + max
            // 超过抛异常：RejectedExecutionException
            for (int i = 0; i < 10; i++) {
                // 使用了线程池之后，使用线程池创建线程
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 线程池用完，程序结束，关闭线程池
            threadPool.shutdown();
        }
    }
}
```

### 四种拒绝策略

![image-20210830143048415](D:\SynologyDrive\allen\img\image-20210830143048415.png)

```java
/**
 * Executors 工具累、3大方法
 *
 * new ThreadPoolExecutor.AbortPolicy(); // 满了，还有人今来，不处理这个人，抛出异常
 * new ThreadPoolExecutor.CallerRunsPolicy(); // 哪来的去哪里
 * new ThreadPoolExecutor.DiscardPolicy(); // 队列满了，丢掉任务，不会抛出异常！
 * new ThreadPoolExecutor.DiscardOldestPolicy(); // 队列满了，抛掉最早的那个，代替它的位置进入队列里
 *
 * Created by Allen on 2021/7/14
 */
```

### 小结和拓展

1. 最大线程池到底该如何定义？(调优)
   - CPU 密集型，几核，就是几，可以保持 CPU 的效率最高！
   - IO 密集型 > 判断程序中十分耗 IO 的线程

```java
// 获取CPU核数
Runtime.getRuntime().availableProcessors();
```

## 12.四大函数式接口（必须掌握）

新时代的程序员：lambda 表达式、链式编程、函数式接口、Stream 流式计算

> 函数式接口：只有一个方法的接口

```java
@FunctionalInterface
public interface Runnable {
    void run();
}

// 超级多FunctionalInterface
// 简化编程模型，在新版本的框架底层大量应用！
// foreach(消费者类的函数式接口)
```

### 代码测试：

> Function 函数式接口：有一个输入参数，有一个输出参数

![](D:\SynologyDrive\allen\img\Snipaste_2021-08-30_15-08-29.png)

```java
package com.allen.function;

import java.util.function.Function;

/**
 * Function 函数型接口，有一个输入参数，有一个输出参数
 * 只要是函数式接口，可以用lambda表达式简化
 */
public class Demo01 {
    public static void main(String[] args) {
        /*Function function = new Function<String, String>() {
            @Override
            public String apply(String o) {
                return o;
            }
        };*/

        /*Function function = (str) -> {
            return str;
        };*/

        Function function = str -> str;
        function.apply("sss");
    }
}
```

> Predicate 断定型接口：有一个输入参数，返回值只能是布尔值

![](D:\SynologyDrive\allen\img\Snipaste_2021-08-30_15-17-15.png)

```java
package com.allen.function;

import java.util.function.Predicate;

/**
 * 断定型接口，有一个输入参数，返回值只能是布尔值
 */
public class Demo02 {
    public static void main(String[] args) {
        // 判断字符串是否为空
        Predicate<String> objectPredicate01 = new Predicate<String>() {
            @Override
            public boolean test(String str) {
                return str.isEmpty();
            }
        };
        Predicate<String> objectPredicate02 = (str) -> {
            return str.isEmpty();
        };

        Predicate<String> objectPredicate03 = (str) -> str.isEmpty();
    }
}
```

> Consumer 消费型接口

> Supplier 供给型接口

## 13.Steam 流式计算

## 14.分支合并

## 15.异步回调

## 16.JMM

## 17.volatile

## 18.深入单例模式

## 19.深入理解 CAS
