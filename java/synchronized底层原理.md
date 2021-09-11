#### **synchronized实现原理**

synchronized是java提供用于处理线程原子性操作相关问题的内置锁。对于每个对象来说，都有一个monitor，你可以把它看做是一个同步工具，synchronized需要通过操作monitor来实现锁的获取和释放。

当synchronized作用于**代码块**的时候，反编译可以看到2个指令，monitorenter和monitorexit
![image](https://user-images.githubusercontent.com/31581862/112711297-4aa58480-8f02-11eb-9878-7998c08240a2.png)


当而synchronized作用于**方法**的时候，则是在标识符flags里添加一个ACC\_SYNCHRONIZED标记，来实现同步功能
![image](https://user-images.githubusercontent.com/31581862/112711307-585b0a00-8f02-11eb-852e-db737f512d54.png)


当线程执行到monitorenter指令时，会去尝试获取monitor，如果锁计数器为0，就会将计数器+1，其他线程就无法获取当前对象的monitor，没有获取到monitor的线程将会阻塞。如果再次重入monitor，计数器会继续+1。相反的，执行monitorexit指令时，会将计数器-1，直到计数器为0的时候，当前线程会释放monitor。

**那么，monitor又是如何工作的呢**\
进一步细化，对于monitor来说，有3个队列contentionList（cxq），entryList和waitSet\
1.当有一个线程获取到锁之后，此时其他阻塞线程会全部进入cxq（已经在entrylist中的线程除外，cxq是一个头插法插入的队列，cas新增节点），等待owner线程锁释放。\
2.获取到锁的线程调用wait方法，会立刻释放锁，并进入waitSet队列等待被唤醒，entryList内的线程开始获取锁（一般是head线程）\
3.有线程调用notify方法的时候，waitSet内的某个线程会被唤醒（如果调用notifyAll会唤醒所有wait线程），此时如果entrylist是空的，线程会直接进入entryList，如果不为空，线程进入cxq，等待锁被释放加入entrylist\
4.持有锁的线程释放锁之后，cxq中的线程全部按照原有的顺序进入entrylist\
5.设置entryList的原因是，可以减少cxq上的并发访问，大大增加吞吐量。另外，进入entrylist的线程的位置不会发生变化，新创建的线程也要先进入cxq

**monitor与对象关联**

一个对象主要由三部分组成
> 对象头：包括Mark Word和Class Pointer
实例数据：存储对象实际有效信息，类属性数据信息等
对齐填充：虚拟机要求对象的起始地址必须是8字节的整数倍，对齐填充相当于占位符的作用

对象头中的Class Pointer是指向对象类元数据的指针，由此获取对象是哪个类的实例
对象头的Mark Word主要存储对象的运行时数据，包括hashcode，分代年龄，GC标记，是否偏向锁，偏向锁线程ID，轻量级锁指针，重量级锁指针等信息，对于synchronize来说，属于重量级锁，所以重量级锁指针就指向monitor的地址。

Mark Word引出了一些锁的概念，比如偏向锁，轻量级锁，重量级锁，之后更新一期关于锁的分配机制和优化机制的文章，看一看jvm是如何优化内置锁的

