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
javap -v -p LambdaDemo
```

可以看到第一条指令为Invokeddynamic\(实际值在“Constant pool"的\#2）。

![](/assets/consumerinvokeDynamic.png)"invokedDynamic"后面的\#2表示指向常量池的第2个符号。



