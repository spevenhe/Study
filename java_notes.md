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

## JMM的内存屏障
上面了解了CPU的内存屏障分类，在JMM中把内存屏障分为四类：

1. LoadLoad Barriers：示例，Load1；LoadLoad；Load2，确保Load1数据的装载先于Load2及所有后续指令的装载；
2. StoreStore Barriers：示例，Store1；StoreStore；Store2，确保Store1数据对其他处理器可见（刷新到内存）先于Store2及所有后续存储指令的存储；
3. LoadStore Barriers：示例，Load1；LoadStore；Store2，确保Load1数据装载先于Store2及所有后续存储指令刷新到内存；
4. StoreLoad Barriers：示例，Store1；StoreLoad；Load2，确保Store1数据对其他处理器变得可见（刷新到内存）先于Load2及所有后续装载指令的装载。StoreLoad Barriers会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。
其中，StoreLoad Barriers同时具有前3个的屏障的效果，但性能开销很大。

为了实现volatile内存语义，JMM会分别限制这两种类型的重排序类型。下图是JMM针对编译器制定的volatile重排序规则表。

![image](https://user-images.githubusercontent.com/42630862/136945540-a01cf8bb-3258-4bcd-81f0-d1069eb2f734.png)

从图中可以得出一个基本规则：

1. 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
2. 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。
3. 当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。
为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能。为此，JMM采取保守策略。下面是基于保守策略的JMM内存屏障插入策略：

1. 在每个volatile写操作的前面插入一个StoreStore屏障。
2. 在每个volatile写操作的后面插入一个StoreLoad屏障。
3. 在每个volatile读操作的后面插入一个LoadLoad屏障。
4. 在每个volatile读操作的后面插入一个LoadStore屏障。

保守策略下volatile写插入内存屏障后生成的指令序列示意图：
![image](https://user-images.githubusercontent.com/42630862/136945746-6d1893c8-4bcb-4a73-86e5-2d9f6357d456.png)
保守策略下volatile读插入内存屏障后生成的指令序列示意图：
![image](https://user-images.githubusercontent.com/42630862/136945799-78c19d7a-38ee-4b21-8a2a-f7f8cf6fb2e1.png)

### JMM对volatile的特殊规则定义
JVM内存指令与volatile相关的操作有：

read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用；

load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中；

use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作；

assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作；

store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作；

write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中；

在对volatile修饰的变量进行操作时，需满足以下规则：

规则1：线程对变量执行的前一个动作是load时才能执行use，反之只有后一个动作是use时才能执行load。线程对变量的read，load，use动作关联，必须连续一起出现。这保证了线程每次使用变量时都需要从主存拿到最新的值，保证了其他线程修改的变量本线程能看到。

规则2：线程对变量执行的前一个动作是assign时才能执行store，反之只有后一个动作是store时才能执行assign。线程对变量的assign，store，write动作关联，必须连续一起出现。这保证了线程每次修改变量后都会立即同步回主内存，保证了本线程修改的变量其他线程能看到。

规则3：有线程T，变量V、变量W。假设动作A是T对V的use或assign动作，P是根据规则2、3与A关联的read或write动作；动作B是T对W的use或assign动作，Q是根据规则2、3与B关联的read或write动作。如果A先与B，那么P先与Q。这保证了volatile修饰的变量不会被指令重排序优化，代码的执行顺序与程序的顺序相同。


### **volatile无法保证原子性**
volatile虽然保证了共享变量的可见性和有序性，但并不能够保证原子性。

以常见的自增操作（count++）为例来进行说明，通常自增操作底层是分三步的：

第一步：获取变量count；
第二步：count加1；
第三步：回写count；
我们来分析一下在这个过程中会有的线程安全问题：

第一步，线程A和B同时获得count的初始值，这一步没什么问题；

第二步，线程A自增count并回写，但线程B此时也已经拿到count，不会再去拿线程A回写的值，因此对原始值进行自增并回写，这就导致了线程安全的问题。有人可能要问了，线程A自增之后不是应该通知其他CPU缓存失效吗，并重新load吗？我们要知道，重新获取的前提操作是读，在线程A回写时，线程B已经拿到了count的值，并不存在再次读的场景。也就是说，线程B的缓存行的确会失效，但线程B中count值已经运行在加法指令中，不存在需要再次从缓存行读的场景。

volatile关键字只保证可见性，所以在以下情况中，需要使用锁来保证原子性：

运算结果依赖变量的当前值，并且有不止一个线程在修改变量的值。
变量需要与其他状态变量共同参与不变约束
所以，想要使用volatile变量提供理想的线程安全，必须同时满足两个条件：

对变量的写操作不依赖于当前值。
该变量没有包含在具有其他变量的不变式中。
也就是说被修饰的变量值独立于任何程序的状态，包括变量的当前状态。



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

```
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
```
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

# java类加载 与jar 包加载

### 类加载器的隔离问题
每个类装载器都有一个自己的命名空间用来保存已装载的类。当一个类装载器装载一个类时，它会通过保存在命名空间里的类全局限定名(Fully Qualified Class Name) 进行搜索来检测这个类是否已经被加载了。

JVM 对类唯一的识别是 ClassLoader id + PackageName + ClassName，所以一个运行程序中是有可能存在两个包名和类名完全一致的类的。并且如果这两个类不是由一个 ClassLoader 加载，是无法将一个类的实例强转为另外一个类的，这就是 ClassLoader 隔离性。

为了解决类加载器的隔离问题，JVM引入了双亲委派机制。

### 双亲委派机制
双亲委派机制的核心有两点：第一，自底向上检查类是否已加载；其二，自顶向下尝试加载类。
![image](https://user-images.githubusercontent.com/42630862/136877679-ec969892-6b00-49ab-b8bd-898149ffa535.png)
类加载器通常有四类：启动类加载器、拓展类加载器、应用程序类加载器和自定义类加载器。

暂且不考虑自定义类加载器，JDK自带类加载器具体执行过程如下：

第一：当AppClassLoader加载一个class时，会把类加载请求委派给父类加载器ExtClassLoader去完成；

第二：当ExtClassLoader加载一个class时，会把类加载请求委派给BootStrapClassLoader去完成；

第三：如果BootStrapClassLoader加载失败（例如在%JAVA_HOME%/jre/lib里未查找到该class），会使用ExtClassLoader来尝试加载；

第四：如果ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

### ClassLoader的双亲委派实现
ClassLoader通过loadClass()方法实现了双亲委托机制，用于类的动态加载。

该方法的源码如下：
```
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
loadClass方法本身是一个递归向上调用的过程，上述代码中从parent.loadClass的调用就可以看出。

在执行其他操作之前，首先通过findLoadedClass方法从最底端的类加载器开始检查是否已经加载指定的类。如果已经加载，则根据resolve参数决定是否要执行连接过程，并返回Class对象。

而Jar包冲突往往发生在这里，当第一个同名的类被加载之后，在这一步检查时就会直接返回，不会再加载真正需要的类。那么，程序用到该类时就会抛出找不到类，或找不到类方法的异常。

## Jar包的加载顺序
上面已经看到一旦一个类被加载之后，全局限定名相同的类可能就无法被加载了。而Jar包被加载的顺序直接决定了类加载的顺序。

决定Jar包加载顺序通常有以下因素：

第一，Jar包所处的加载路径。也就是加载该Jar包的类加载器在JVM类加载器树结构中所处层级。上面讲到的四类类加载器加载的Jar包的路径是有不同的优先级的。
第二，文件系统的文件加载顺序。因Tomcat、Resin等容器的ClassLoader获取加载路径下的文件列表时是不排序的，这就依赖于底层文件系统返回的顺序，当不同环境之间的文件系统不一致时，就会出现有的环境没问题，有的环境出现冲突。

## Jar包冲突的通常表现
Jar包冲突往往是很诡异的事情，也很难排查，但也会有一些共性的表现。

抛出java.lang.ClassNotFoundException：典型异常，主要是依赖中没有该类。导致原因有两方面：第一，的确没有引入该类；第二，由于Jar包冲突，Maven仲裁机制选择了错误的版本，导致加载的Jar包中没有该类。
抛出java.lang.NoSuchMethodError：找不到特定的方法。Jar包冲突，导致选择了错误的依赖版本，该依赖版本中的类对不存在该方法，或该方法已经被升级。
抛出java.lang.NoClassDefFoundError，java.lang.LinkageError等，原因同上。
没有异常但预期结果不同：加载了错误的版本，不同的版本底层实现不同，导致预期结果不一致。


###  Tomcat启动时Jar包和类的加载顺序
最后，梳理一下Tomcat启动时，对Jar包和类的加载顺序，其中包含上面提到的不同种类的类加载器默认加载的目录：


1. $java_home/lib 目录下的java核心api；
2. $java_home/lib/ext 目录下的java扩展jar包；
3. java -classpath/-Djava.class.path所指的目录下的类与jar包；
4. $CATALINA_HOME/common目录下按照文件夹的顺序从上往下依次加载；
5. $CATALINA_HOME/server目录下按照文件夹的顺序从上往下依次加载；
6. $CATALINA_BASE/shared目录下按照文件夹的顺序从上往下依次加载；
7. 项目路径/WEB-INF/classes下的class文件；
8. 项目路径/WEB-INF/lib下的jar文件；**

上述目录中，同一文件夹下的Jar包，按照顺序从上到下一次加载。如果一个class文件已经被加载到JVM中，后面相同的class文件就不会被加载了。


# Maven Jar包管理机制
在Maven项目中，想要了解Jar冲突必须得先了解一下Maven是如何管理的Jar包的。这涉及到Maven的一些特性，比如依赖传递、最短路径优先原则、最先声明原则等。

### 依赖传递原则
当在Maven项目中引入A的依赖，A的依赖通常又会引入B的jar包，B可能还会引入C的jar包。这样，当你在pom.xml文件中添加了A的依赖，Maven会自动的帮你把所有相关的依赖都添加进来。

比如，在Spring Boot项中，当引入了spring-boot-starter-web：

<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
此时Maven的依赖结构可能是这样的：

![image](https://user-images.githubusercontent.com/42630862/136886323-d1dcb59b-0790-4534-b29e-e08be861bdd0.png)

spring-boot-web依赖
上面这种关系，我们就可以理解为依赖的传递性。即当一个依赖需要另外一个依赖支撑时，Maven会帮我们把相应的依赖依次添加到项目当中。

这样的好处是，使用起来就非常方便，不用自己挨个去找依赖Jar包了。坏处是会引起Jar包冲突，我们后面会讲到。

### 最短路径优先原则
依赖链路一：主要根据依赖的路径长短来决定引入哪个依赖（两个冲突的依赖）。

举例说明：

依赖链路一：A -> X -> Y -> Z(21.0)
依赖链路二：B -> Q -> Z(20.0)
项目中同时引入了A和B两个依赖，它们间接都引入了Z依赖，但由于B的依赖链路比较短，因此最终生效的是Z(20.0)版本。这就是最短路径优先原则。

此时如果Z的21.0版本和20.0版本区别较大，那么就会发生Jar包冲突的表现。

### 最先声明优先原则
如果两个依赖的路径一样，最短路径优先原则是无法进行判断的，此时需要使用最先声明优先原则，也就是说，谁的声明在前则优先选择。

举例说明：

依赖链路一：A -> X -> Z(21.0)
依赖链路二：B -> Q -> Z(20.0)
A和B最终都依赖Z，此时A的声明（pom中引入的顺序）优先于B，则针对冲突的Z会优先引入Z(21.0)。

如果Z(21.0)向下兼容Z(20.0)，则不会出现Jar包冲突问题。但如果将B声明放前面，则有可能会发生Jar包冲突。

## Jar包冲突产生的原因
上面讲了Maven维护Jar包的三个原则，其实每个原则会发生什么样的Jar包冲突，已经大概了解了。这里再来一个综合示例。

举例说明：

依赖链路一：A -> B -> C -> G21(guava 21.0)
依赖链路二：D -> F -> G20(guava 20.0)
假设项目中同时引入了A和D的依赖，按照依赖传递机制和默认依赖调节机制（第一：路径最近者优先；第二：第一声明优先），默认会引入G20版本的Jar包，而G21的Jar包不会被引用。

如果C中的方法使用了G21版本中的某个新方法（或类），由于Maven的处理，导致G21并未被引入。此时，程序在调用对应类时便会抛出ClassNotFoundException异常，调用对应方法时便会抛出NoSuchMethodError异常。


