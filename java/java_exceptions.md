![image](https://user-images.githubusercontent.com/42630862/147538906-e819d79c-29fc-43dc-a99e-09e63fd35499.png)

# 异常分为两类 运行时异常和编译时异常。

## 运行时异常
「定义」：RuntimeException 类及其子类，表示 JVM 在运行期间可能出现的异常。

「特点」：Java 编译器不会检查它。也就是说，当程序中可能出现这类异常时，倘若既"没有通过throws声明抛出它"，也"没有用try-catch语句捕获它"，还是会编译通过。比如NullPointerException空指针异常、ArrayIndexOutBoundException数组下标越界异常、ClassCastException类型转换异常、ArithmeticExecption算术异常。此类异常属于不受检异常，一般是由程序逻辑错误引起的，在程序中可以选择捕获处理，也可以不处理。虽然 Java 编译器不会检查运行时异常，但是我们也可以通过 throws 进行声明抛出，也可以通过 try-catch 对它进行捕获处理。如果产生运行时异常，则需要通过修改代码来进行避免。例如，若会发生除数为零的情况，则需要通过代码避免该情况的发生！

RuntimeException 异常会由 Java 虚拟机自动抛出并自动捕获（「就算我们没写异常捕获语句运行时也会抛出错误」！！），此类异常的出现绝大数情况是代码本身有问题应该从逻辑上去解决并改进代码

## 编译时异常
「定义」: Exception 中除 RuntimeException 及其子类之外的异常。

「特点」: Java 编译器会检查它。如果程序中出现此类异常，比如 ClassNotFoundException（没有找到指定的类异常），IOException（IO流异常），要么通过throws进行声明抛出，要么通过try-catch进行捕获处理，否则不能通过编译。在程序中，通常不会自定义该类异常，而是直接使用系统提供的异常类。「该异常我们必须手动在代码里添加捕获语句来处理该异常」。


# 异常处理
Java异常处理

![image](https://user-images.githubusercontent.com/42630862/147550290-093e97ab-f6ae-4bd3-9256-5a98adc93601.png)

## 捕获异常
private static void readFile(String filePath) {
    try {
        // code
    } catch (FileNotFoundException | UnknownHostException e) {
        // handle FileNotFoundException or UnknownHostException
    } catch (IOException e){
        // handle IOException
}
}

# 常见问题

*1. NoClassDefFoundError 和 ClassNotFoundException 区别？*

NoClassDefFoundError 是一个 Error 类型的异常，是由 JVM 引起的，不应该尝试捕获这个异常。

引起该异常的原因是 JVM 或 ClassLoader 尝试加载某类时在内存中找不到该类的定义，该动作发生在运行期间，即编译时该类存在，但是在运行时却找不到了，可能是变异后被删除了等原因导致；

ClassNotFoundException 是一个受查异常，需要显式地使用 try-catch 对其进行捕获和处理，或在方法签名中用 throws 关键字进行声明。当使用 Class.forName, ClassLoader.loadClass 或 ClassLoader.findSystemClass 动态加载类到内存的时候，通过传入的类路径参数没有找到该类，就会抛出该异常；另一种抛出该异常的可能原因是某个类已经由一个类加载器加载至内存中，另一个加载器又尝试去加载它。







