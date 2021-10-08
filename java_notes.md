# 几个关键字
## volatile
java 最轻量级的同步机制

Java内存模型由Java虚拟机规范定义，用来屏蔽各个平台的硬件差异。简单来说:

所有变量储存在主内存。
每条线程拥有自己的工作内存，其中保存了主内存中线程使用到的变量的副本。
线程不能直接读写主内存中的变量，所有操作均在工作内存中完成。
线程，主内存，工作内存的交互关系如图。
![image](https://user-images.githubusercontent.com/42630862/131701242-e3b22917-1573-4c10-832d-be8af5184fa5.png)

volatile 指令无法被重排

但是volatile不保证原子性，要保证原子性必须加锁，Volatile 变量具有 synchronized 的可见性特性

### volatile的适用场景
模式 #1：状态标志
也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。
volatile boolean shutdownRequested;
 
...
 
public void shutdown() { 
    shutdownRequested = true; 
}
 
public void doWork() { 
    while (!shutdownRequested) { 
        // do stuff
    }
}

线程1执行doWork()的过程中，可能有另外的线程2调用了shutdown，所以boolean变量必须是volatile。
而如果使用 synchronized 块编写循环要比使用 volatile 状态标志编写麻烦很多。由于 volatile 简化了编码，并且状态标志并不依赖于程序内任何其他状态，因此此处非常适合使用 volatile。


**这种类型的状态标记的一个公共特性是：通常只有一种状态转换；shutdownRequested 标志从false 转换为true，然后程序停止。**
这种模式可以扩展到来回转换的状态标志，但是只有在转换周期不被察觉的情况下才能扩展（从false 到true，再转换到false）。此外，还需要某些原子状态转换机制，例如原子变量。

### 模式 #2：一次性安全发布（one-time safe publication）
在缺乏同步的情况下，可能会遇到某个对象引用的更新值（由另一个线程写入）和该对象状态的旧值同时存在。

这就是造成著名的双重检查锁定（double-checked-locking）问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象。参见：【设计模式】5. 单例模式（以及多线程、无序写入、volatile对单例的影响）

//注意volatile！！！！！！！！！！！！！！！！！  
private volatile static Singleton instace;   
  
public static Singleton getInstance(){   
    //第一次null检查     
    if(instance == null){            
        synchronized(Singleton.class) {    //1     
            //第二次null检查       
            if(instance == null){          //2  
                instance = new Singleton();//3  
            }  
        }           
    }  
    return instance;        

如果不用volatile，则因为内存模型允许所谓的“无序写入”，可能导致失败。——某个线程可能会获得一个未完全初始化的实例。
考察上述代码中的 //3 行。此行代码创建了一个 Singleton 对象并初始化变量 instance 来引用此对象。这行代码的问题是：在Singleton 构造函数体执行之前，变量instance 可能成为非 null 的！
什么？这一说法可能让您始料未及，但事实确实如此。

在解释这个现象如何发生前，请先暂时接受这一事实，我们先来考察一下双重检查锁定是如何被破坏的。假设上述代码执行以下事件序列：
    线程 1 进入 getInstance() 方法。
    由于 instance 为 null，线程 1 在 //1 处进入synchronized 块。
    线程 1 前进到 //3 处，但在构造函数执行之前，使实例成为非null。
    线程 1 被线程 2 预占。
    线程 2 检查实例是否为 null。因为实例不为 null，线程 2 将instance 引用返回，返回一个构造完整但部分初始化了的Singleton 对象。
    线程 2 被线程 1 预占。
    线程 1 通过运行 Singleton 对象的构造函数并将引用返回给它，来完成对该对象的初始化。

### 模式 #3：独立观察（independent observation）
安全使用 volatile 的另一种简单模式是：定期 “发布” 观察结果供程序内部使用。【例如】假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值。

使用该模式的另一种应用程序就是收集程序的统计信息。【例】如下代码展示了身份验证机制如何记忆最近一次登录的用户的名字。将反复使用lastUser 引用来发布值，以供程序的其他部分使用。

public class UserManager {
    public volatile String lastUser; //发布的信息
 
    public boolean authenticate(String user, String password) {
        boolean valid = passwordIsValid(user, password);
        if (valid) {
            User u = new User();
            activeUsers.add(u);
            lastUser = user;
        }
        return valid;
    }
} 

### 模式 #4：“volatile bean” 模式
volatile bean 模式的基本原理是：很多框架为易变数据的持有者（例如 HttpSession）提供了容器，但是放入这些容器中的对象必须是线程安全的。
在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通——即不包含约束！

@ThreadSafe
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
 
    public void setFirstName(String firstName) { 
        this.firstName = firstName;
    }
 
    public void setLastName(String lastName) { 
        this.lastName = lastName;
    }
 
    public void setAge(int age) { 
        this.age = age;
    }
}
### 模式 #5：开销较低的“读－写锁”策略
如果读操作远远超过写操作，您可以结合使用内部锁和 volatile 变量来减少公共代码路径的开销。

如下显示的线程安全的计数器，使用 synchronized 确保增量操作是原子的，并使用 volatile 保证当前结果的可见性。如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及 volatile 读操作，这通常要优于一个无竞争的锁获取的开销。
@ThreadSafe
public class CheesyCounter {
    // Employs the cheap read-write lock trick
    // All mutative operations MUST be done with the 'this' lock held
    @GuardedBy("this") private volatile int value;
 
    //读操作，没有synchronized，提高性能
    public int getValue() { 
        return value; 
    } 
 
    //写操作，必须synchronized。因为x++不是原子操作
    public synchronized int increment() {
        return value++;
    }

使用锁进行所有变化的操作，使用 volatile 进行只读操作。
其中，锁一次只允许一个线程访问值，volatile 允许多个线程执行读操作


## synchronized

synchronized 是 Java 的一个关键字，它能够将代码块 (方法) 锁起来。

synchronized 是 互斥锁，同一时间只能有一个线程进入被锁住的代码块(方法)。

synchronized 通过监视器(Monitor)实现锁。java 一切皆对象，每个对象都有一个监视器(锁标记)，而 synchronized 就是使用对象的监视器来将代码块 (方法) 锁定的！

### 怎么用 Synchronized ？
修饰普通同步方法：对象锁：一个对象一把锁，多个对象多把锁。

修饰静态同步方法：类锁：所有对象共用一个锁

修饰同步代码块：也分类锁、对象锁

其中 synchronized(this)为对象锁，synchronized(A.Class)为类锁

通过命令看下 synchronized 关键字到底做了什么事情：首先用 cd 命令切换到 SynchronizedTest.java 类所在的路径，然后执行 javac SynchronizedTest.java，于是就会产生一个名为 SynchronizedTest.class 的字节码文件，然后我们执行 javap -c SynchronizedTest.class，就可以看到对应的反汇编内容，如下：

Z:\IDEAProject\review\review_java\src\main\java\com\nasus\thread\lock>javac -encoding UTF-8 SynchronizedTest.java

Z:\IDEAProject\review\review_java\src\main\java\com\nasus\thread\lock>javap -c SynchronizedTest.class
Compiled from "SynchronizedTest.java"
public class com.nasus.thread.lock.SynchronizedTest {
  public com.nasus.thread.lock.SynchronizedTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."":()V
       4: return
  public static synchronized void test();
    Code:
       0: return
  public synchronized void test1();
    Code:
       0: return
  public void test2();
    Code:
       0: aload_0
       1: dup
       2: astore_1
       3: monitorenter  // 监视器进入，获取锁
       4: aload_1
       5: monitorexit  // 监视器退出，释放锁
       6: goto          14
       9: astore_2
      10: aload_1
      11: monitorexit  // 监视器退出，释放锁
      12: aload_2
      13: athrow
      14: return
    Exception table:
       from    to  target type
           4     6     9   any
           9    12     9   any
}
类锁 同步代码块解析
主要看 类锁 同步代码块的反编译内容，可以看出 synchronized 多了 monitorenter 和 monitorexit 指令。把执行 monitorenter 理解为加锁，执行 monitorexit 理解为释放锁，每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为 0。

那这里为啥只有一次 monitorenter 却有两次 monitorexit ？

JVM 要保证每个 monitorenter 必须有与之对应的 monitorexit，monitorenter 指令被插入到同步代码块的开始位置，而 monitorexit 需要插入到方法正常结束处和异常处两个地方，这样就可以保证抛异常的情况下也能释放锁。
执行 monitorenter 的线程尝试获得 monitor 的所有权，会发生以下这三种情况之一：

a. 如果该 monitor 的计数为 0，则线程获得该 monitor 并将其计数设置为 1。然后，该线程就是这个 monitor 的所有者。b. 如果线程已经拥有了这个 monitor ，则它将重新进入，并且累加计数。c. 如果其他线程已经拥有了这个 monitor，那个这个线程就会被阻塞，直到这个 monitor 的计数变成为 0，代表这个 monitor 已经被释放了，于是当前这个线程就会再次尝试获取这个 monitor。

monitorexit

monitorexit 的作用是将 monitor 的计数器减 1，直到减为 0 为止。代表这个 monitor 已经被释放了，已经没有任何线程拥有它了，也就代表着解锁，所以，其他正在等待这个 monitor 的线程，此时便可以再次尝试获取这个 monitor 的所有权。

对象锁 普通同步方法
它并不是依靠 monitorenter 和 monitorexit 指令实现的，从上面的反编译内容可以看到，synchronized 方法和普通方法大部分是一样的，不同在于，这个方法会有一个叫作 ACC_SYNCHRONIZED 的 flag 修饰符，来表明它是同步方法。(在这看不出来需要看 JVM 底层实现)

当某个线程要访问某个方法的时候，会首先检查方法是否有 ACC_SYNCHRONIZED 标志，如果有则需要先获得 monitor 锁，然后才能开始执行方法，方法执行之后再释放 monitor 锁。



