HashMap是java中非常常用的容器，以前只是停留在使用阶段，对于它的底层设计更是一知半解，看到源码才知道它巧妙的设计和工作原理。在了解底层原理之前，建议先学习相关的数据结构[几种常见数据结构](https://github.com/MasterWilliamCheng/codeman/blob/main/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/%E5%87%A0%E7%A7%8D%E5%B8%B8%E8%A7%81%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.md)，可以帮助你更好的了解HashMap。

#### 数据结构

> HashMap的数据结构是**数组+链表**，默认初始大小是**16**，默认负载因子是**0.75**，当实际容量大小（容量大小是包含每一个node的大小，不是非空数组的大小）超过定义的容量Capacity乘以负载因子的值后，HashMap会进行一次**resize**扩容操作，创建一个2倍大小的新数组，同时**rehash**所有的数据（之所以需要rehash，而不是直接复制之前的数组位置，是因为hash算法中需要使用到数组长度，扩容后数组的长度变化了）。\
> **Java8进一步做了优化，不需要再重新进行hash计算，将元素原来的hash和旧数组的大小（大小为2次幂）做与运算，为0则表示数组位置不变，不为0则表示需要移位，新位置为原先位置+旧数组的大小，把原来在同一个位置上面的链表拆分成两个链表。**


![image](https://user-images.githubusercontent.com/31581862/113298031-f64d3b00-932d-11eb-854a-38e9a6e9c099.png)

![image](https://user-images.githubusercontent.com/31581862/132988108-0617320c-b8fa-463a-8068-425e6fd53ad3.png)


> HashMap初始大小是16，设置成2的幂，便于进行位运算，另外，在手动设置HashMap大小的时候，也尽量设置成2的幂数

> **链表进化和退化机制：**hashmap中的链表长度在大于8的时候会进化成**红黑树结构**，小于6的时候又会退化成链表。那么，为什么是8和6呢?那是因为在多次的实验中发现，发生8次hash碰撞的概率非常非常小，设置成8可以在小概率事件发生后，转换数据结构，优化后续查询性能，退化阈值设置成6是为了避免设置成8或者7的时候，因为hash碰撞在8附近来回的切换数据结构。

#### Hash算法

> 每一个key会经过一次hash运算（先获取key的hashcode，将hashcode无符号右移16位[hashcode >>> 16]，对右移后的值和hashcode进行异或运算（二进制里相同得0，不同得1））[hash = hashcode ^ hashcode >>> 16]，得到key的hash值，至于数组的index值，需要将hash值和数组长度减一后的值进行与运算（同时为1结果为1，否则为0）[(length - 1) & hash]得到。这里实际上是将取模运算简化成了位运算，h%n = h&(n-1)

![image](https://user-images.githubusercontent.com/31581862/113298067-fea57600-932d-11eb-9516-38568b38fb89.png)

![image](https://user-images.githubusercontent.com/31581862/113298083-02d19380-932e-11eb-9c2d-c2e0959edcb8.png)



> hashmap发生hash碰撞的数组会使用尾插法（jdk7是头插法）插入一个node，node中包含key，hash值，value以及下一个node的指针next。
> 
> tips：<<左移，右边补0 >>右移，左边初始位为正补0，为负补1 >>>无符号右移，初始位补0 对于二进制来说，初始位0为正，1为负。

#### 线程安全

> HashMap的线程不安全，在jdk7和jdk8中的情况不同

JDK7
> 在jdk7中，使用的是头插法，会有两种情况的线程不安全问题
> （1）在数组的某个索引的位置，A线程要插入一对key-value，new Entry的指针next指向桶的第一个Entry（jdk7中的Entry在jdk8中变成了Node），这个时候B线程也要插入一对key-value，因为A线程的赋值还未完成，所以线程B创建的new Entry的next也指向桶的第一个Entry，并成功添加。随后线程B结束，线程A开始进行赋值操作，这个时候就会发现，在线程A执行完之后，线程B所添加的数据被覆盖了。

> （2）另一种情况是在扩容方法transfer中，假设某个链表桶有两个相邻的Entry，Entry1和Entry2，Entry1的next指针指向Entry2，当线程A在执行transfer操作的时候，Entry1执行到图例标注之前的时候，切换到B线程，B线程操作完成了整个HashMap的resize，如果此时经过rehash后，Entry1和Entry2依旧发生了hash碰撞，那么此时扩容后的HashMap，因为头插法的存在，Entry2的next指针指向Entry1。同时切换到线程A，线程A继续执行图例红框中的指令，那么Entry1的next指针指向新数组的桶的链表头Node，也就等同于Entry1的next指针指向Entry2，又因为Entry2的next指针是指向Entry1的，所以会陷入死循环，线程A无法跳出resize这步。

![image](https://user-images.githubusercontent.com/31581862/113298110-0c5afb80-932e-11eb-9b8e-3905787afa2e.png)


JDK8
> 在jdk8中，移除了transfer方法，取而代之的是resize方法。因为尾插法的存在，不会出现死循环的情况，但是会出现数据覆盖的问题。
> 假设线程A和线程B同时在put值，它们的key的hash值经过计算后得到的index索引值相同，发生了hash碰撞，但此时线程A挂起，线程B进行了newNode操作，线程B结束之后，线程A继续执行，同样的，它也会进行newNode操作，会覆盖前面线程B的更新，线程B put的数据就消失了。

