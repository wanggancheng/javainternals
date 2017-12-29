# Java Lambda及InvokeDynamic调用探秘

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



