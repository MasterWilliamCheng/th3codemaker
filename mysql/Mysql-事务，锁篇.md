前面探究了mysql的数据结构和索引，本文我们来学习一下mysql中事务和锁方面的知识。总结了一些点，方便温故知新

## **事务**
**1. ACID四大特性**

**Atomicity：原子性**
事务中的操作要么全部成功要么全部失败

**Consistency：一致性**
一个事务在执行前后，数据库都必须保持一致性，数据库只包含成功提交事务的结果

**Isolation：隔离性**
事务的执行是相互隔离的，不被干扰的

**Durability：持久性**
事务提交后，对数据的修改必须是永久保存的

**2. 隔离级别**

**未提交读（READ UNCOMMITTED）**
事务可以读取其他未提交事务中修改的数据，会发生脏读，不可重复读，幻读

**已提交读（READ COMMITTED）**
事务只能读取其他已提交事务修改的数据，可以解决脏读（读取的数据是别的未提交事务修改的数据）

**可重复读（REPEATABLE READ）**
在事务开启时，不再允许修改操作，一个事务多次读取同一数据得到的结果是一致的，可以解决不可重复读（一个事务进行2次查询，中间有别的事务进行了修改操作，导致两次查询的结果不同）

**可串行化（SERIALIZABLE）**
事务串行执行，不能并发执行，可以解决幻读（一个事务进行2次查询，中间有别的事务进行了新增或删除操作，导致得到了不同条数的数据）

## **锁**
**共享锁（读锁）**  
允许多个事务共享一把锁，但是对于加锁的数据只能读取，不能进行UPDATE DETELE等操作
lock in share mode可以添加共享锁

**排他锁（写锁）**  
只有一个事务能拿到排他锁，其他事务不能获取到锁，会阻塞直到持有锁的事务释放锁或者等待超时，不能与其他锁共存
InnoDB会对update，insert，delete语句自动加排它锁
select ... for update也会添加排他锁
其他事务可以正常执行select语句，因为select不涉及加锁（有些文章模糊了这块的概念，强调排他锁事务未提交时，其他事务不能对锁住的数据进行任何操作，我在实验过后发现不加for update的查询操作是可以执行的）

**意向共享锁/意向排他锁**\
在事务获取共享锁或排他锁之前，会对整张表先加锁，意向锁存在的意义是支持行锁和表锁共存

**行锁、表锁**\
myisam没有行锁，都是表锁\
innodb默认是行锁，前提条件是建立在索引之上的。如果筛选条件没有建立索引，会降级到表锁

## **锁算法**
**Record Lock（记录锁）**
对记录上的索引加锁，可以理解为行锁

**Gap Lock（间隙锁）**
间隙锁，只对索引的间隙加锁，但是不包括索引本身，只有RR隔离级别以上才支持间隙锁

**Next-Key Lock（临键锁）**
Record Lock+Gap Lock，不仅对索引加锁，也对索引的间隙加锁（左开右闭），InnoDB利用Next-Key Lock来解决幻读问题

## 加锁规则
**1. next-key lock 是前开后闭区间((x,y])。**

**2. 查找过程中访问到的对象才会加锁。**

**3. 索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。（这里强调，只有唯一索引才会退化为行锁，如果是普通索引，仍然为间隙锁+行锁，也就是next-key lock）**

**4. 索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。**

**5. 唯一索引上的范围查询会访问到不满足条件的第一个值为止**

举例：
假设我们有张只有主键id的表，表的id为1,2,3,4,5,6,9,11，普通索引字段a为1,2,3,4,5,6,9,11，我们看看不同的sql下，加锁的结果是什么

> select * from user where id =8 for update
8介于索引6 9之间，next-key lock前开后闭(6,9]，又因为右边最后一个值9不等于8，所以next-key lock退化成间隙锁(6,9) 

> select * from user where id =9 for update
 因为是唯一索引等值查询，退化成行锁

> select * from user where id >6 for update
 根据第五条规则，next-key lock范围(6,+MAXVALUE]

> select * from user where id >=6 for update
因为6是等值查询，所以需要加行锁，next-key lock范围[6,MAXVALUE] 

> select * from user where id >7 for update
介于6之前，所以next-key lock范围(6,MAXVALUE]

> select * from user where id >6 and id <8 for update
因为规则5，第一个不满足查询条件的是9，所以next-key lock范围(6,9]   

> select * from user where a =6 for update
next-key lock范围(5,6]+(6,9)，因为不是唯一索引，所以间隙锁不会退化为行锁

> select * from user where a =7 for update
next-key lock范围(6,9)

> update user set name = xx where id =8 
next-key lock范围(6,9] 

> update user set name = xx where id =6
next-key lock范围(5,6] + (6,9)

> update user set name = xx where id >6
next-key lock范围(6,MAXVALUE]  


----------


#### 2021/5/11 更新

参考自[【美团技术团队】Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)

对于读取历史数据的方式，我们叫它快照读 (snapshot read)，而读取数据库当前版本数据的方式，叫当前读 (current read)。
很显然，在MVCC中：

**快照读**：就是select

select * from table ….;

**当前读**：特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。

select * from table where ? lock in share mode;
select * from table where ? for update;
insert;
update ;
delete;

**MVCC是解决快照读的幻读问题，间隙锁是解决当前读的幻读问题。他们解决不同情况下的幻读问题**
