# 单例模式

就是采取一定的方法保证在整个的软件系统中，对某个类只能存在一个对象实例，并且该类只提供一个取得其对象实例的方法(静态方法)。

1) 饿汉式(静态常量)
```
class Singleton {
private static Singleton instance;
private Singleton() {}
//提供一个静态的公有方法，当使用到该方法时，才去创建instance
//即懒汉式
public static Singleton getInstance() {
if(instance == null) {
instance = new Singleton();
}
return instance;
}
}
```
2) 饿汉式（静态代码块）
3) 懒汉式(线程不安全)
4) 懒汉式(线程安全，同步方法)
5) 懒汉式(线程安全，同步代码块)
6) 双重检查 **推荐**
```
public class DoubleCheckedLocking {                     //1
    private static Instance instance;                   //2
    public  static Instance getInstance(){              //3
        if(instance ==null) {                           //4:第一次检查
            synchronized (DoubleCheckedLocking.class) { //5：加锁
                if (instance == null)                   //6：第二次检查
                    instance = new Instance();          //7：问题的根源处在这里
            }                                           //8
        }                                               //9
        return instance;                                //10
    }                                                   //11
}
```
7) 静态内部类 **推荐**

```
class Singleton {
private static volatile Singleton instance;
//构造器私有化
private Singleton() {}
//写一个静态内部类,该类中有一个静态属性Singleton
private static class SingletonInstance {
private static final Singleton INSTANCE = new Singleton();
}
//提供一个静态的公有方法，直接返回SingletonInstance.INSTANCE
public static synchronized Singleton getInstance() {
return SingletonInstance.INSTANCE;
}
}
```
8) 枚举
```
enum Singleton {
INSTANCE; //属性
public void sayOK() {
System.out.println("ok~");
}
}
....
Singleton instance = Singleton.INSTANCE;
```
 优缺点说明：
1) 这借助JDK1.5 中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建
新的对象。
2) 这种方式是Effective Java 作者Josh Bloch 提倡的方式

# 工厂模式
1. 简单工厂模式：
简单工厂模式的设计方案: 定义一个可以实例化Pizaa 对象的类，封装创建对象的代码。

2. 工厂模式：
1) 工厂方法模式设计方案：将披萨项目的实例化功能抽象成抽象方法，在不同的口味点餐子类中具体实现。
2) 工厂方法模式：定义了一个创建对象的抽象方法，由子类决定要实例化的类。工厂方法模式将对象的实例
化推迟到子类。


![image](https://user-images.githubusercontent.com/42630862/147737778-dd1e5806-7363-4c29-be35-f6d57f993229.png)


3. 抽象工厂模式
1) 抽象工厂模式：定义了一个interface 用于创建相关或有依赖关系的对象簇，而无需指明具体的类
2) 抽象工厂模式可以将简单工厂模式和工厂方法模式进行整合。
3) 从设计层面看，抽象工厂模式就是对简单工厂模式的改进(或者称为进一步的抽象)。
4) 将工厂抽象成两层，AbsFactory(抽象工厂) 和具体实现的工厂子类。程序员可以根据创建对象类型使用对应
的工厂子类。这样将单个的简单工厂类变成了工厂簇，更利于代码的维护和扩展。


![image](https://user-images.githubusercontent.com/42630862/147737801-a3786b64-09e2-40e1-8220-d5200405b37f.png)



# 代理模式

1) 代理模式：为一个对象提供一个替身，以控制对这个对象的访问。即通过代理对象访问目标对象.这样做的好处
是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能。
2) 被代理的对象可以是远程对象、创建开销大的对象或需要安全控制的对象
3) 代理模式有不同的形式, 主要有三种静态代理、动态代理(JDK 代理、接口代理)和Cglib 代理(可以在内存
动态的创建对象，而不需要实现接口， 他是属于动态代理的范畴) 。
4) 代理模式示意图

![image](https://user-images.githubusercontent.com/42630862/147738132-758c83a8-3033-42a7-b6f7-e64309ba7661.png)


## 1.静态代理

 具体要求
1) 定义一个接口:ITeacherDao
2) 目标对象TeacherDAO 实现接口ITeacherDAO
3) 使用静态代理方式,就需要在代理对象TeacherDAOProxy 中也实现ITeacherDAO
4) 调用的时候通过调用代理对象的方法来调用目标对象.
5) 特别提醒：代理对象与目标对象要实现相同的接口,然后通过调用相同的方法来调用目标对象的方法
![image](https://user-images.githubusercontent.com/42630862/147739178-c481c9a4-407d-42bf-95ad-7d9b6407a696.png)

15.2.3 静态代理优缺点
1) 优点：在不修改目标对象的功能前提下, 能通过代理对象对目标功能扩展
2) 缺点：因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类
3) 一旦接口增加方法,目标对象与代理对象都要维护

## 动态代理模式的基本介绍

1) 代理对象,不需要实现接口，但是目标对象要实现接口，否则不能用动态代理
2) 代理对象的生成，是利用JDK 的API，动态的在内存中构建代理对象
3) 动态代理也叫做：JDK 代理、接口代理
15.3.2 JDK 中生成代理对象的API
1) 代理类所在包:java.lang.reflect.Proxy
2) JDK 实现代理只需要使用newProxyInstance 方法,但是该方法需要接收三个参数,完整的写法是:
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )

![image](https://user-images.githubusercontent.com/42630862/147739296-12a64af7-f4d6-4f84-b6f4-0c3aca86f04f.png)


![image](https://user-images.githubusercontent.com/42630862/147739555-64300719-1956-41ec-8c77-552932692512.png)


![image](https://user-images.githubusercontent.com/42630862/147739573-57d059ae-ed7f-4f6c-8687-16de04649b0a.png)







