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


3. 抽象工厂模式
1) 抽象工厂模式：定义了一个interface 用于创建相关或有依赖关系的对象簇，而无需指明具体的类
2) 抽象工厂模式可以将简单工厂模式和工厂方法模式进行整合。
3) 从设计层面看，抽象工厂模式就是对简单工厂模式的改进(或者称为进一步的抽象)。
4) 将工厂抽象成两层，AbsFactory(抽象工厂) 和具体实现的工厂子类。程序员可以根据创建对象类型使用对应
的工厂子类。这样将单个的简单工厂类变成了工厂簇，更利于代码的维护和扩展。




