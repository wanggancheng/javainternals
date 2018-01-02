# Java Lambda及InvokedDynamic调用探秘

我们先来看看一个java Lambda的简单实例。

```java
package com.github.wanggancheng;
import java.util.function.Consumer;
public class LambdaDemo {


    public static void main(String[] args) {

        Consumer<String> consumer=(s)->System.out.println(s);
        consumer.accept("Lambda demo");
    }
}
```

这个程序编译后运行时会输出“Lambda demo"。我们先来通过命令查看class字节码文件：

```bash
javap  -p LambdaDemo
```

可以看到此类有下列方法：

```java
public class com.github.wanggancheng.LambdaDemo {
  public com.github.wanggancheng.LambdaDemo();
  public static void main(java.lang.String[]);
  private static void lambda$main$0(java.lang.String);
}
```

   除了类的构造函数及main方法外，多了一个名为lambda$main$0的静态方法。此静态方法的参数为java.lang.String类型。

   下面我们通过对字节码的分析来理解背后的密码。

可以看到第一条指令为Invokeddynamic\(实际值在“Constant pool"的\#2）。

![](/assets/consumerinvokeDynamic.png)"\#0"表明指向的是第一个BootstrapMethod。"accept"是invokedName,"\(\)Ljava/util/function/Consumer"是invokedType。二者合为NameAndType。

