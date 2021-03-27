#### **synchronized实现原理**

synchronized是java提供用于处理线程原子性操作相关问题的内置锁。对于每个对象来说，都有一个monitor，你可以把它看做是一个同步工具，synchronized需要通过操作monitor来实现锁的获取和释放。

当synchronized作用于**对象**的时候，反编译可以看到2个指令，monitorenter和monitorexit
![image](https://user-images.githubusercontent.com/31581862/112711297-4aa58480-8f02-11eb-9878-7998c08240a2.png)


当而synchronized作用于**方法**的时候，则是在标识符flags里添加一个ACC\_SYNCHRONIZED标记，来实现同步功能
![image](https://user-images.githubusercontent.com/31581862/112711307-585b0a00-8f02-11eb-852e-db737f512d54.png)


当线程执行到monitorenter指令时，会去尝试获取monitor，如果锁计数器为0，就会将计数器+1，其他线程就无法获取当前对象的monitor，没有获取到monitor的线程将会阻塞。如果再次重入monitor，计数器会继续+1。相反的，执行monitorexit指令时，会将计数器-1，直到计数器为0的时候，当前线程会释放monitor。

**那么，monitor又是如何工作的呢**
进一步细化，对于monitor来说，有2个队列entryList和waitSet，线程在请求获取monitor之前都会进入entryList
1.当有一个线程获取到monitor之后会进入一个Owner区域，同时对计数器进行+1，此时对于entryList里的线程来说都是阻塞的，等待锁释放。
2.进入Owner的线程调用wait方法，会释放monitor，同时计数器-1，并进入waitSet队列等待被唤醒，entryList内的线程开始新一轮的竞争
3.有线程调用notify方法的时候，waitSet内的某个线程会被唤醒（当然，如果调用notifyAll会唤醒所有wait线程），线程会重新进入entryList，去继续下一轮竞争
4.执行完同步代码的线程会退出区域，同时释放锁

**monitor与对象关联**

一个对象主要由三部分组成
> 对象头：包括Mark Word和Class Pointer
实例数据：存储对象实际有效信息，类属性数据信息等
对齐填充：虚拟机要求对象的起始地址必须是8字节的整数倍，对齐填充相当于占位符的作用

对象头中的Class Pointer是指向对象类元数据的指针，由此获取对象是哪个类的实例
对象头的Mark Word主要存储对象的运行时数据，包括hashcode，分代年龄，GC标记，是否偏向锁，偏向锁线程ID，轻量级锁指针，重量级锁指针等信息，对于synchronize来说，属于重量级锁，所以重量级锁指针就指向monitor的地址。

Mark Word引出了一些锁的概念，比如偏向锁，轻量级锁，重量级锁，之后更新一期关于锁的分配机制和优化机制的文章，看一看jvm是如何优化内置锁的

