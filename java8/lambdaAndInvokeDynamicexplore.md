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

让我们来看看main方法的字节码吧。

```java
  0: invokedynamic #2,  0              // InvokeDynamic #0:accept:()Ljava/util/function/Consumer;
         5: astore_1
         6: aload_1
         7: ldc           #3                  // String Lambda demo
         9: invokeinterface #4,  2            // InterfaceMethod java/util/function/Consumer.accept:(Ljava/lang/Object;)V
        14: return
```

从offset为0开始，第一条字节码信息为：CONSTANT\_\_InvokeDynamic\_info。它在常量池中的位置为2（\#2）

```java
  #2 = InvokeDynamic      #0:#36         // #0:accept:()Ljava/util/function/Consumer;
```

它主要由两部分组成。  
 第一部分为bootstrap\_method\_attr\_index,也就是指定了bootstrap\_methods\[\]数组中的合法索引。在这里的\#0表示指向了bootstrap\_methods数组中第1个bootstrap\_method。  
 第二部分为name\_and\_type\_index。\#36表示常量池中的位置。

```java
  #36 = NameAndType        #49:#50        // accept:()Ljava/util/function/Consumer;
```

"accept"是invokedName,"\(\)Ljava/util/function/Consumer"是invokedType。二者合为NameAndType。

 接下来，我们看看bootstrap_methods。它的结构定义如下：_

```java
BootstrapMethods_attribute {
u2 attribute_name_index;
u4 attribute_length;
u2 num_bootstrap_methods;
{ u2 bootstrap_method_ref;
u2 num_bootstrap_arguments;
u2 bootstrap_arguments[num_bootstrap_arguments];
} bootstrap_methods[num_bootstrap_methods];
}
```

每个bootstrap_method由两部分组成：bootstrap\_method\_ref及bootstrap\_arguments。_

在这个类中，bootstrapmethods如下：

```java
BootstrapMethods:
  0: #32 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #33 (Ljava/lang/Object;)V
      #34 invokestatic com/github/wanggancheng/LambdaDemo.lambda$main$0:(Ljava/lang/String;)V
      #35 (Ljava/lang/String;)V
```



