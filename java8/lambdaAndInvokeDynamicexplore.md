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

从上面可知，bootstrap\_method\_ref为\#32，它在常量池的内容如下：

```java
#30 = NameAndType #9:#10 // "<init>":()V
#31 = Utf8 BootstrapMethods
#32 = MethodHandle #6:#46 // invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
#33 = MethodType #47 // (Ljava/lang/Object;)V
#34 = MethodHandle #6:#48 // invokestatic com/github/wanggancheng/LambdaDemo.lambda$main$0:(Ljava/lang/String;)V
#35 = MethodType #25 // (Ljava/lang/String;)V
#36 = NameAndType #49:#50 // accept:()Ljava/util/function/Consumer;
#37 = Utf8 Lambda demo
#38 = Class #51 // java/util/function/Consumer
#39 = NameAndType #49:#47 // accept:(Ljava/lang/Object;)V
#40 = Class #52 // java/lang/System
#41 = NameAndType #53:#54 // out:Ljava/io/PrintStream;
#42 = Class #55 // java/io/PrintStream
#43 = NameAndType #56:#25 // println:(Ljava/lang/String;)V
#44 = Utf8               com/github/wanggancheng/LambdaDemo
#45 = Utf8               java/lang/Object
#46 = Methodref          #57.#58        // java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
#47 = Utf8               (Ljava/lang/Object;)V
#48 = Methodref          #7.#59         // com/github/wanggancheng/LambdaDemo.lambda$main$0:(Ljava/lang/String;)V
#49 = Utf8               accept
#50 = Utf8               ()Ljava/util/function/Consumer;
#51 = Utf8               java/util/function/Consumer
#52 = Utf8               java/lang/System
#53 = Utf8               out
#54 = Utf8               Ljava/io/PrintStream;
#55 = Utf8               java/io/PrintStream
#56 = Utf8               println
#57 = Class              #60            // java/lang/invoke/LambdaMetafactory
#58 = NameAndType        #61:#65        // metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
#59 = NameAndType        #24:#25        // lambda$main$0:(Ljava/lang/String;)V
#60 = Utf8               java/lang/invoke/LambdaMetafactory
#61 = Utf8               metafactory
#62 = Class              #67            // java/lang/invoke/MethodHandles$Lookup
#63 = Utf8               Lookup
#64 = Utf8               InnerClasses
#65 = Utf8               (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
```

其实是一个MethodHandle。一个MethodHandle主要由方法句柄类型reference_kind及对应的引用类型reference\_index。在这里引用类型为invoke\_static，引用索引则响应的指向一个静态方法。这个静态方法为_/LambdaMetafactory.metafactory。

第1个bootstrap\_method需要三个参数，分别为\#33，\#34，\#35指向的类型。这是与LambdaMetaFactory.metafactory的最后三个参数一致的。

