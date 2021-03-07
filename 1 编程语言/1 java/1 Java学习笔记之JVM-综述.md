# JVM综述

![](https://cdn.jsdelivr.net/gh/vpdong/opt-imgs@master/data/20210307192851-JVM%E8%99%9A%E6%8B%9F%E6%9C%BA.png)

一个JVM实例，对应着多个后台线程，这些线程不包括调用main方法的main线程和创建main线程的线程，主要包括如下：

+ GC线程：对JVM不同种类的垃圾收集行为提供支持
+ 编译线程：在运行时将字节码编译成本地代码
+ 虚拟机线程：这种线程的执行类型包括“stop-the-world”的垃圾收集、线程栈收集、线程挂起、偏向锁撤销
+ 信号调度线程：接收信号并发送给JVM，在它内部通过适当的方法进行处理
+ 周期任务线程：一般用于周期性操作的调度执行

一个JVM实例，对应着一个Runtime对象(Runtime类描述了虚拟机一些信息)，该类采用了单例设计模式，可通过静态方法getRuntime()获取Runtime类实例。

+ Runtime类提供totalMemory\maxMemory\freeMemory等方法，用于监控JVM内存使用情况。
+ Runtime类提供gc()方法，用于释放JVM的一些无用空间。
+ Runtime类提供exec()方法，用于执行可执行程序，返回一个新的Process(当前JVM进程的子进程)。



