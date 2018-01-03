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

从第１个bootstrapmethod来看，与第１篇文章中的示例的bootstrapmethod的差异是第２个参数。先前的实例中第２个参数是调用编译器自动生成的一个静态方法ambda$main$0。

　　我们来看看一个稍复杂的例子。

```java
package com.github.wanggancheng;
import java.util.function.Consumer;
public class InvokeDynamicInstanceMethodDemo {

    public void output(String s){

        System.out.println(s);
    }

    public static void main(String[] args) {

        InvokeDynamicInstanceMethodDemo instance = new InvokeDynamicInstanceMethodDemo();
        Consumer<String> consumer=instance::output;
        consumer.accept("Lambda demo");
    }
}
```

　反编译此类的class文件，可以发现也没有添加新方法。还是重点来看看bootstrap\_method方法信息。

```java
BootstrapMethods:
  0: #41 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #42 (Ljava/lang/Object;)V
      #43 invokevirtual com/github/wanggancheng/InvokeDynamicInstanceMethodDemo.output:(Ljava/lang/String;)V
      #44 (Ljava/lang/String;)V
```

重点差别为＃43，表明调用类型为invokevirtual而非invokestatic。

invokedynamice\_info的信息如下：

```java
#7 = InvokeDynamic      #0:#45         // #0:accept:(Lcom/github/wanggancheng/InvokeDynamicInstanceMethodDemo;)Ljava/util/function/Consumer;
```

从上面的信息可以看出，与先前的示例的主要差别为invokedType中包含实例方法引用的类实例作为参数。

　我们在运行时把自动生成的类dump出来，反编译信息如下。

```java
package com.github.wanggancheng;

import java.lang.invoke.LambdaForm.Hidden;
import java.util.function.Consumer;

// $FF: synthetic class
final class InvokeDynamicInstanceMethodDemo$$Lambda$1 implements Consumer {
    private final InvokeDynamicInstanceMethodDemo arg$1;

    private InvokeDynamicInstanceMethodDemo$$Lambda$1(InvokeDynamicInstanceMethodDemo var1) {
        this.arg$1 = var1;
    }

    private static Consumer get$Lambda(InvokeDynamicInstanceMethodDemo var0) {
        return new InvokeDynamicInstanceMethodDemo$$Lambda$1(var0);
    }

    @Hidden
    public void accept(Object var1) {
        this.arg$1.output((String)var1);
    }
}

```



