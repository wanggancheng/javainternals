# Java Lambda及InvokedDynamic调用探秘（二）

在第一篇中我们了解到实例中的Lambda最终会生成一个静态方法，通过invokeDynamic指令最后在执行过程中最终会调用到这个静态方法。

在这篇中我们将深入分析一下ｏracle jdk内部是如何构建出bootstrap\_method的，以及如何创建出自动新增加的类的。

我们可在LambdaMetaFactory的metafactory方法中添加一个断点。

![](/assets/metafactory断点.png)

运行时信息如下面两图。

![](/assets/metafactory运行参数信息.png)

![](/assets/metafactory执行时参数信息2.png)

这些参数中，除了ｃaller参数外，其它都是从java类文件中可以直接获取到的或包装了而已。没有印象可以查看第一篇文章。

跟踪到InnerClassLambdaMetaFactory的实例创建过程。

![](/assets/InnerClassMetaFactorystaticinit.png)

上篇文章中Dump出InnerClass的开关参数就是在InnerClassLambdaMetaFactory的类静态初始化语句块中设置的。

继续跟踪到可以看到执行到下列代码块。

![](/assets/lambdaClassName.png)　　从上图可以看出ＬambdaClasName的值是如何生成的。ＩmplMethodName就是

跟进到ＩnnerClassLambdaFactory的ｂuildCallSite方法中，可以看到如下逻辑：

![](/assets/Constructor.png)继续跟踪，可以看到如下代码。

![](/assets/sambuildview.png)

最终，dump出InnerClass的地方是在InvokerBytecodeGenerator的maybeDump方法中。

![](/assets/maybedump.png)

