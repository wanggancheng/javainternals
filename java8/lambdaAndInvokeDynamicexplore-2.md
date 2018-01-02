# Java Lambda及InvokedDynamic调用探秘（二）

在第一篇中我们了解到实例中的Lambda最终会生成一个静态方法，通过invokeDynamic指令最后在执行过程中最终会调用到这个静态方法。

　　在这篇中我们将深入分析一下ｏracle jdk内部是如何构建出bootstrap\_method的，以及如何创建出自动新增加的类的。



