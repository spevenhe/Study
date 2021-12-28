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

