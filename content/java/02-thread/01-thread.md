---
title: 线程概述
date: 2021-04-06
categories: [Thread]
toc: true
---

## 1、线程简介

多任务：看起来是多个任务都在做，其实本质上我们的大脑在同一时间依旧只做了一件事。

多线程：原来是一条路，慢慢因为车太多了，道路堵塞，效率极低。为了提高使用的效率，能够充分利用道路，于是加了多个车道。

![image-20210406212626857](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210406212626857.png)

### Process 与 Thread

- 说起进程，就不得不说下**程序**。程序是指令和数据的有序集合，其本身没有任何运行的含义，是一个静态的概念。
- 而**进程**则是执行程序的一次执行过程，它是一个动态的概念。是系统资源分配的单位
- 通常在一个进程中可以包含若干个**线程**，当然一个进程中至少有一个线程，不然没有存在的意义。线程是 CPU 调度和执行的单位

> 注意：很多多线程是模拟出来的，真正的多线程是指有多个 CPU，即多核，如服务器。如果是模拟出来的多线程，即在一个 CPU 的情况下，在同一个时间点，CPU 只能执行一个代码，因为切换的很快，所以就有同时执行的错觉。

### 核心概念

- 线程就是独立的执行路径；
- 在程序运行时，即使没有自己创建线程，后台也会有多个线程，如主线程，gc 线程；
- main() 称之为主线程，为系统的入口，用于执行整个程序；
- 在一个进程中，如果开辟了多个线程，线程的运行由调度器安排调度，调度器是与操作系统紧密相关的，先后顺序是不能人为的干预的；
- 在同一份资源操作时，会存在资源抢夺的问题，需要加入并发控制；
- 线程会带来额外的开销，如 CPU 调度时间，并发控制开销；
- 每个线程在自己的工作内存交互，内存控制不当会造成数据不一致；



## 2、线程实现（重点）

### 继承 Thread 类

- 自定义线程类继承 **Thread类**
- 重写 **run()** 方法，编写线程执行体
- 创建线程对象，调用 **start()** 方法启动线程

```java
public class StartThread extends Thread {
  // 线程入口点
  @Override
  public void run() {
    // 线程体
    for (int i = 0; i < 20; i++) {
      System.out.println("线程");
    }
  }
}

public static void main(String[] args) {
  // 创建线程对象
  StartThread t = new StartThread();
  t.start();
}
```



**线程不一定立即执行，CPU 安排调度**



### 实现 Runnable 接口

- 定义 MyRunnable 类实现 **Runnable** 接口
- **实现 run()** 方法，编写线程执行体
- 创建线程对象，调用 **start()** 方法启动线程

```java
public class StartThread implements Runnable {
  @Override
  public void run() {
    // 线程体
    for (int i = 0; i < 20; i++) {
      System.out.println("Runnable");
    }
  }
}

public static void main(String[] args) {
  // 创建实现类对象
  StartThread t = new StartThread();
  // 创建代理类对象
  Thread thread = new Thread(t);
  // 启动
  thread.start();
}
```



**推荐使用 Runnable 对象，因为 Java 单继承的局限性**



### 小结

- 继承 Thread 类
	- 子类继承 Thread 类具备多线程能力
	- 启动线程：子类对象.start()
	- **不建议使用：避免 OOP 单继承局限性**
- 实现 Runnable 接口
	- 实现接口 Runnable 具有多线程能力
	- 启动线程：传入目标对象 + Thread对象.start()
	- **推荐使用：避免单继承局限性，灵活方便，方便同一个对象被多个线程使用**



### 实现 Callable 接口

1. 实现 Callable 接口，需要返回值类型
2. 重写 call 方法，需要抛出异常
3. 创建目标对象
4. 创建执行服务：`ExecutorService ser = Executors.newFixedThreadPool(1);`
5. 提交执行：`Future<Boolean> result = ser.submit(t);`
6. 获取结果：`boolean r = result.get();`
7. 关闭服务：`ser.shutdownNow();`

优点：

1. 可以定义返回值
2. 可以抛出异常



### 静态代理

```java
public class StaticProxy {
  public static void main(String[] args) {
    WeddingCompany wc = new WeddingCompany(new You());
    wc.HappyMarry();
  }
}

interface Marry{
  void HappyMarry();
}

// 真实角色
calss You implements Marry{
  @Override
  public void HappyMarry() {
    System.out.println("Marry");
  }
}

// 代理角色
calss WeddingCompany implements Marry{
  
  private Marry target;
  
  public WeddingCompany(Marry marry) {
    this.target = marry;
  }
  
  @Override
  public void HappyMarry() {
    before();
    this.target.HappyMarry();
    after();
  }
  
  private void before() {
    System.out.println("前");
  }
  
  private void after() {
    System.out.println("后");
  }
}
```

总结：

1. 真实对象和代理对象都要实现同一个接口
2. 代理对象要代理真实对象

优点：

1. 代理对象可以做很多真实对象做不了的事情
2. 真实对象专注做自己的事



## 3、线程状态

![image-20210407210150375](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210407210150375.png)

![image-20210407211306477](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210407211306477.png)

### 方法

- 更改线程的优先级：`setPriority(int newPriority)`
- 在指定的毫秒数内让当前正在执行的线程休眠：`static void sleep(long millis)`
- 等待该线程终止：`void join()`
- 暂停当前正在执行的线程对象，并执行其他线程：`static void yield()`
- 中断线程（别用这个方式）：`void interrupt()`
- 测试线程是否处于活动状态：`bollean isAlive()`

### 停止线程

> - 不推荐使用 JDK 提供的 stop()、destroy() 方法。【已废弃】
> - 推荐线程自己停止下来
> - 建议使用一个标志位进行终止变量当 flag = false，则终止线程运行。

```java
public class TestStop implements Runnable {
  // 1.线程中定义线程体使用的标识
  private boolean flag = true;
  
  @Override
  public void run() {
    // 2.线程体使用该标识
    while(flag) {
      System.out.println("run...Thread");
    }
  }
  
  // 3.对外提供方法改变标识
  public void stop() {
    this.flag = false;
  }
  
  public static void main(String[] args) {
    TestStop ts = new TestStop();
    new Thread(ts).start();
    for(int i = 0; i < 1000; i++) {
      if (i == 900) {
        // 调用stop方法切换标识位，让线程停止
        ts.stop();
      }
    }
  }
}
```

### 线程休眠

- sleep（时间）指定当前线程阻塞的毫秒数；
- sleep 存在异常 InterruptedException；
- sleep 时间达到后线程进入就绪状态；
- sleep 可以模拟网络延时，倒计时等；
- 每一个对象都有一个锁，sleep 不会释放锁；

### 线程礼让

- 礼让线程，让当前正在执行的线程暂停，但不阻塞
- 将线程从运行状态转为就绪状态
- **让 CPU 重新调度，礼让不一定成功！看 CPU 心情**

```java
public class TeasYield {
  public static void main(String[] args) {
    MyYield myYield = new MyYield();
    new Thread(myYield, "a").start();
    new Thread(myYield, "b").start();
  }
}

class MyYield implements Runnable {
  @Override
  public void run() {
    System.out.println(Thread.currentThread().getName() + "线程开始执行");
    Thread.yield(); // 礼让
    System.out.println(Thread.currentThread().getName() + "线程停止执行");
  }
}
```

### Join

- Join 合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞
- 可以想象成插队

```java
public class TestJoin implements Runnable {
  public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(new TestJoin());
    thread..start();
    for (int i = 0; i < 100; i++) {
      if (i == 50) {
        thread.join(); // mian线程阻塞
      }
      System.out.println("mian..." + i);
    }
  }
  
  @Override
  public void run() {
    for (int i = 0; i < 1000; i++) {
      System.out.println("join..." + i);
    }
  }
}
```

### 线程状态观测（Thread.State）

线程状态，线程可以处于以下状态之一：

- NEW

	尚未启动的线程处于此状态

- RUNNABLE

	在 Java 虚拟机中执行的线程处于此状态

- BLOCKED

	被阻塞等待监视器锁定的线程处于此状态

- WAITING

	正在等待另一个线程执行特定动作的线程处于此状态

- TIMED_WAITING

	正在等待另一个线程执行动作达到指定等待时间的线程处于此状态

- TERMINATED

	已退出的线程处于此状态

一个线程可以在给定时间点处于一个状态。这些状态是不反映任何操作系统线程状态的虚拟机状态。

```java
public class TestState  {

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("......");
        });

        // 观察状态
        Thread.State state = thread.getState();
        System.out.println(state); // NEW

        // 观察启动后
        thread.start();
        state = thread.getState();
        System.out.println(state); //Run

        // 只要线程不终止，就一直输出状态
        while (state != Thread.State.TERMINATED) {
            Thread.sleep(100);
            state = thread.getState(); // 更新线程状态
            System.out.println(state);
        }
    }
}
```

### 线程优先级

- Java 提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度哪个线程来执行
- 线程的优先级用数字表示，范围从1～10
	- Thread.MIN_PRIORITY = 1;
	- Thread.MAX_PRIORITY = 10;
	- Thread.NORM_PRIORITY = 5;
- 使用以下方式改变或获取优先级
	- getPriority()
	- setPriority(int xxx);

### 守护线程

- 线程分为**用户线程**和**守护线程**
- 虚拟机必须确保用户线程执行完毕
- 虚拟机不用等待守护线程执行完毕
- 如：后台记录操作日志、监控内存、垃圾回收等

```java
public class TestDaemon {

    public static void main(String[] args) {
        God god = new God();
        You you = new You();

        Thread thread = new Thread(god);
        thread.setDaemon(true); // 默认是 false 表示是用户线程，正常的线程都是用户线程
        thread.start();

        new Thread(new You()).start(); // 用户线程启动
    }

}

// 守护线程
class God implements Runnable {

    @Override
    public void run() {
        while (true) {
            System.out.println("上帝守护你");
        }
    }
}

// 用户线程
class You implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 36500; i++) {
            System.out.println("你的一生");
        }
        System.out.println("===goodbye, world!===");
    }
}
```



## 4、线程同步（重点）

> 多个线程操作同一个资源

由于同一进程的多个线程共享一个存储空间，在带来方便的同时，也带来了访问冲突的问题，为了保证数据在方法中被访问时的正确性，在访问时加入**锁机制 synchronized**，当一个线程获得对象的排它锁，独占资源，其他线程必须等待，使用后释放锁即可，存在以下问题：

- 一个线程持有锁会导致其他所需要此锁的线程挂起；
- 在多线程竞争下，加锁，释放锁会导致比较多的上下文切换和调度延时，引起性能问题；
- 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能问题；

### 并发

> 同一个对象被多个线程同时操作

处理多线程问题时，多个线程访问同一个对象，并且某些线程还想修改这个对象，这时候我们就需要线程同步，线程同步其实就是一种等待机制，多个需要同时访问此对象的线程进入这个对象的等待池形成队列，等待前面线程使用完毕，下一个线程再使用

### 线程不安全案例

不安全买票：

```java
public class UnsafeBuTicket {
    public static void main(String[] args) {
        BuyTicket station = new BuyTicket();
        new Thread(station, "李老大").start();
        new Thread(station, "王老二").start();
        new Thread(station, "张老三").start();
    }
}

class BuyTicket implements Runnable {

    // 票
    private int ticketNums = 10;
    boolean flag = true;

    @Override
    public void run() {
        // 买票
        while (flag) {
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void buy() throws InterruptedException {
        // 判断是否有票
        if (ticketNums <= 0) {
            flag = false;
            return;
        }
        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName() + "拿到" + ticketNums--);
    }
}
```

不安全取钱：

```java
public class UnsafeBank {
    public static void main(String[] args) {
        Account account = new Account(100, "投资");
        Drawing you = new Drawing(account, 50, "王老二");
        Drawing he = new Drawing(account, 100, "张老三");
        you.start();
        he.start();
    }
}

// 账户
class Account {
    int money; // 余额
    String name; //卡名

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }
}

// 银行
class Drawing extends Thread {
    Account account; // 账户
    int drawingMoney; // 取了多少钱
    int nowMoney;

    public Drawing(Account account, int drawingMoney, String name) {
        this.account = account;
        this.drawingMoney = drawingMoney;
        this.setName(name);
    }

    @Override
    public void run() {
        // 判断有没有钱
        if (account.money - drawingMoney < 0) {
            System.out.println(Thread.currentThread().getName() + "钱不够");
            return;
        }
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 卡内余额 = 余额 - 取的钱
        account.money = account.money - drawingMoney;
        // 手里的钱
        nowMoney = nowMoney + drawingMoney;

        System.out.println(account.name + "的余额为：" + account.money);
        System.out.println(this.getName() + "的现金为：" + nowMoney);
    }
}
```

线程不安全的集合：

```java
public class UnsafeList {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10000; i++) {
            new Thread(() -> {
                list.add(Thread.currentThread().getName());
            }).start();
        }
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(list.size());
    }
}
```

### 同步方法

- 由于我们可以通过 private 关键字来保证数据对象只能被方法访问，所以我们只需要针对方法提出一套机制，这套机制就是 `synchronized` 关键字，它包括两种用法：`synchronized`方法和 `synchronized`块

	同步方法：`public synchronized void method(int args){}`

- `synchronized` 方法控制对“对象”的访问，每个对象对应一把锁，每个 `synchronized` 方法都必须获得调用该方法的对象的锁才能执行，否则线程会阻塞，方法一旦执行，就独占该锁，直到该方法返回才释放锁，后面被阻塞的线程才能获得这个锁，继续执行

	缺陷：若将一个大的方法申明为 `synchronized` 将会影响效率

### 同步方法弊端

![image-20210408201941596](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408201941596.png)

### 同步块

- 同步块：synchronized (Obj) {}
- Obj 称之为 同步监视器
	- Obj 可以是任何对象，但是推荐使用共享资源作为同步监视器
	- 同步方法中无需指定同步监视器，因为同步方法的同步监视器就是 this，就是这个对象本身，或者是 class [反射中讲解]
- 同步监视器的执行过程
	- 第一个线程访问，锁定同步监视器，执行其中代码
	- 第二个线程访问，发现同步监视器被锁定，无法访问
	- 第一个线程访问完毕，解锁同步监视器
	- 第二个线程访问，发现同步监视器没有锁，然后锁定并访问

安全的取票：

```java
public class UnsafeBuTicket {
    public static void main(String[] args) {
        BuyTicket station = new BuyTicket();
        new Thread(station, "李老大").start();
        new Thread(station, "王老二").start();
        new Thread(station, "张老三").start();
    }
}

class BuyTicket implements Runnable {

    // 票
    private int ticketNums = 10;
    boolean flag = true;

    @Override
    public void run() {
        // 买票
        while (flag) {
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

  	// synchronized
    private synchronized void buy() throws InterruptedException {
        // 判断是否有票
        if (ticketNums <= 0) {
            flag = false;
            return;
        }
        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName() + "拿到" + ticketNums--);
    }
}
```

安全的银行取钱：

```java
public class UnsafeBank {
    public static void main(String[] args) {
        Account account = new Account(100, "投资");
        Drawing you = new Drawing(account, 50, "王老二");
        Drawing he = new Drawing(account, 100, "张老三");
        you.start();
        he.start();
    }
}

// 账户
class Account {
    int money; // 余额
    String name; //卡名

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }
}

// 银行
class Drawing extends Thread {
    Account account; // 账户
    int drawingMoney; // 取了多少钱
    int nowMoney;

    public Drawing(Account account, int drawingMoney, String name) {
        this.account = account;
        this.drawingMoney = drawingMoney;
        this.setName(name);
    }

    @Override
    public void run() {
        synchronized (account) {
            // 判断有没有钱
            if (account.money - drawingMoney < 0) {
                System.out.println(Thread.currentThread().getName() + "钱不够");
                return;
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 卡内余额 = 余额 - 取的钱
            account.money = account.money - drawingMoney;
            // 手里的钱
            nowMoney = nowMoney + drawingMoney;

            System.out.println(account.name + "的余额为：" + account.money);
            System.out.println(this.getName() + "的现金为：" + nowMoney);
        }
    }
}
```

### 死锁

多个线程各自占有一些共享资源，并且互相等待其他线程占有的资源才能运行，而导致两个或者多个线程都在等待对方释放资源，都停止执行的情形，某一个同步块同时拥有“两个以上对象的锁”时，就可能发生“死锁”的问题。

```java
// 死锁：多个线程互相抱着对方需要的资源，然后形成僵持
public class DeadLock {
    public static void main(String[] args) {
        Makeup g1 = new Makeup(0, "红姑娘");
        Makeup g2 = new Makeup(1, "粉姑娘");
        g1.start();
        g2.start();
    }
}

// 口红
class Lipstick {}

// 镜子
class Mirror {}

class Makeup extends Thread {

    // 需要的资源，只有一份，用static来保证只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();

    int choice; // 选择
    String girlName; // 使用的人

    public Makeup(int choice, String girlName) {
        this.choice = choice;
        this.girlName = girlName;
    }

    @Override
    public void run() {
        // 化妆
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 化妆，互相持有对方的锁，就是需要拿到对方的资源
    private void makeup() throws InterruptedException {
        if (choice == 0) {
            synchronized (lipstick) { // 获得口红的锁
                System.out.println(this.girlName + "口红的锁" );
                Thread.sleep(1000);
                synchronized (mirror) { // 一秒钟后获得镜子
                    System.out.println(this.girlName + "镜子的锁");
                }
            }
        } else {
            synchronized (mirror) { // 获得镜子的锁
                System.out.println(this.girlName + "镜子的锁" );
                Thread.sleep(2000);
                synchronized (lipstick) { // 二秒钟后获得镜子
                    System.out.println(this.girlName + "口红的锁");
                }
            }
        }
    }
}
```

### 死锁避免方法

产生死锁的四个必要条件：

- 互斥条件：一个资源每次只能被一个进程使用。
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件：进程已获得的资源，在未使用之前，不能强行剥夺。
- 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

> 上面列出来死锁的四个必要条件，我们只要想办法破其中的任意一个或多个条件就可以避免死锁发生

### Lock（锁）

- 从 JDK 5.0开始，Java 提供了更强大的线程同步机制---通过显示定义同步锁对象来实现同步。同步锁使用 Lock 对象充当
- java.util.concurrent.locks.Lock 接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对 Lock 对象加锁，线程开始访问共享资源之前应先获得 Lock 对象
- ReentrantLock 类实现了 Lock，它拥有与 synchronized 相同的并发性和内存语义，在实现线程安全的控制中，比较常见的是 ReentrantLock，可以显式加锁、释放锁

```java
public class TestLock {
    public static void main(String[] args) {
        TestLock2 lock2 = new TestLock2();
        new Thread(lock2).start();
        new Thread(lock2).start();
        new Thread(lock2).start();
    }
}

class TestLock2 implements Runnable {

    int ticketNums = 10;

    // 定义 lock 锁
    private final ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            try {
                lock.lock(); // 加锁
                if (ticketNums > 0) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(ticketNums--);
                } else {
                    break;
                }
            } finally {
                // 解锁
                lock.unlock();
            }
        }
    }
}
```

### synchronized 与 Lock 的对比

- Lock 是显式锁（手动开启和关闭，别忘记关闭锁），synchronized 是隐式锁，出了作用域自动释放
- Lock 只有代码块锁，synchronized 有代码块锁和方法锁
- 使用 Lock 锁，JVM 将花费较少的时间来调度线程，性能更好，并且具有更好的扩展性（提供更多的子类）
- 优先使用顺序：
	- Lock > 同步代码块（已经进入了方法体，分配了相应资源）> 同步方法（在方法体之外）



## 5、线程通信问题

![image-20210408220115333](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408220115333.png)

![image-20210408220238264](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408220238264.png)

![image-20210408220405794](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408220405794.png)

![image-20210408220542458](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408220542458.png)

```java
// 测试：生产者消费者模型 --> 利用缓冲区解决：管程法
// 生产者，消费者，产品，缓冲区
public class TestPC {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Productor(container).start();
        new Consumer(container).start();
    }
}

// 生产者
class Productor extends Thread {
    SynContainer container;

    public Productor(SynContainer container) {
        this.container = container;
    }
    // 生产
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            System.out.println("生产了" + i + "只鸡");
        }
    }
}

// 消费者
class Consumer extends Thread {
    SynContainer container;

    public Consumer(SynContainer container) {
        this.container = container;
    }
    // 消费
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了-->" + container.pop().id + "只鸡");
        }
    }
}

// 产品
class Chicken {
    int id; // 产品编号
    public Chicken(int id) {
        this.id = id;
    }
}

// 缓冲区
class SynContainer {
    // 需要一个容器大小
    Chicken[] chickens = new Chicken[10];
    // 容器技术器
    int count = 0;

    // 生产者放入产品
    public synchronized  void push(Chicken chicken) {
        // 如果容器满了，就需要等待消费者
        if (count == chickens.length) {
            // 通知消费者消费，生产等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果没有满，我们就需要丢入产品
        chickens[count] = chicken;
        count ++;
        // 可以通知消费者消费了
        this.notifyAll();
    }

    // 消费者消费产品
    public synchronized Chicken pop(){
        // 判断是否消费
        if (count == 0) {
            // 等待生产者生产，消费者等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 如果可以消费
        count--;
        Chicken chicken = chickens[count];
        // 吃完了，通知生产者生产
        this.notifyAll();
        return chicken;
    }
}
```



![image-20210408224755573](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408224755573.png)

```java
// 测试生产者消费者问题2：信号灯法，标志位解决
public class TestPC2 {
    public static void main(String[] args) {
        TV tv = new TV();
        new Player(tv).start();
        new Watcher(tv).start();
    }

}

// 生产者-->演员
class Player extends Thread{
    TV tv;

    public Player(TV tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i % 2 == 0) {
                this.tv.play("快乐大本营播放中");
            } else {
                this.tv.play("广告播放中");
            }
        }
    }
}

// 消费者-->观众
class Watcher extends Thread {
    TV tv;

    public Watcher(TV tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

// 产品-->节目
class TV {
    // 演员表演，观众等待
    // 观众观看，演员等待
    String voice;
    boolean flag = true;

    // 表演
    public synchronized void play(String voice) {
        if (!flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("演员表演了：" + voice);
        // 通知观众观看
        this.notifyAll();
        this.voice = voice;
        this.flag = !this.flag;
    }

    // 观看
    public synchronized void watch() {
        if (flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观众观看了：" + voice);
        // 通知演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }
}
```



## 6、线程池

![image-20210408231720634](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408231720634.png)

![image-20210408231900904](https://cdn.jsdelivr.net/gh/wangtao-Allen/images//img/image-20210408231900904.png)

```java
// 测试线程池
public class TestPool {
    public static void main(String[] args) {
        // 1.创建服务，创建线程池
        ExecutorService service = Executors.newFixedThreadPool(10);
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        // 2.关闭链接
        service.shutdown();
    }
}

class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + i);
    }
}
```

