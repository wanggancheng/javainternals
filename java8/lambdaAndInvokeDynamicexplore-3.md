# Java Lambda及InvokedDynamic调用探秘（三）

　　这次我们来探究一下其它情况。

```java
package com.github.wanggancheng;
import java.util.function.Consumer;
public class InvokeDynamicDemo {

    public static void output(String s){

        System.out.println(s);
    }

    public static void main(String[] args) {

        Consumer<String> consumer=InvokeDynamicDemo::output;
        consumer.accept("Lambda demo");
    }
}
```

 从上面的代码可以看出，与先前的主要差异为：用类静态方法来替代java Lambda。我们反编译class文件的方法信息如下：

```java
public class com.github.wanggancheng.InvokeDynamicDemo {
  public com.github.wanggancheng.InvokeDynamicDemo();
  public static void output(java.lang.String);
  public static void main(java.lang.String[]);
}
```

从上面的信息来看，并没有自动添加什么方法。我们来看看ｍain方法的字节码信息。

```java
 public static void main(java.lang.String[]);
    Code:
       0: invokedynamic #4,  0              // InvokeDynamic #0:accept:()Ljava/util/function/Consumer;
       5: astore_1
       6: aload_1
       7: ldc           #5                  // String Lambda demo
       9: invokeinterface #6,  2            // InterfaceMethod java/util/function/Consumer.accept:(Ljava/lang/Object;)V
      14: return
```

从上面可以看出第一条指令还是invokedynamic,有先前的例子没有什么差别。我们来看更具体的字节码信息。

```java
BootstrapMethods:
  0: #36 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #37 (Ljava/lang/Object;)V
      #38 invokestatic com/github/wanggancheng/InvokeDynamicDemo.output:(Ljava/lang/String;)V
      #39 (Ljava/lang/String;)V
```

从第１个bootstrapmethod来看，与先前的

