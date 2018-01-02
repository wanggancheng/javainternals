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

让我们来看看main方法的字节码吧。![](/assets/consumerinvokeDynamic.png)从offset为0开始，第一条字节码信息为：CONSTANT\_\_InvokeDynamic\_info。它在常量池中的位置为2（\#2）

```java
  #2 = InvokeDynamic      #0:#36         // #0:accept:()Ljava/util/function/Consumer;
```

它主要由两部分组成。  
 第一部分为bootstra\_pmethod\_attr\_index,也就是指定了bootstrap\_methods\[\]数组中的合法索引。在这里的\#0表示指向了bootstrap\_methods数组中第1个bootstrap\_method。  
 第二部分为name\_and\_type\_index。\#36表示常量池中的位置。

```java
  #36 = NameAndType        #49:#50        // accept:()Ljava/util/function/Consumer;
```

"accept"是invokedName,"\(\)Ljava/util/function/Consumer"是invokedType。二者合为NameAndType。

BootstramMethod是一种什么结构呢。

