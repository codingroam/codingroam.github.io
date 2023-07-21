# MySQL RR级别下仍然会发生幻读和不可重复读

MySQL RR级别下仍然会发生幻读和不可重复读
<!--more-->


## 前言

InnoDB的RR级别中，MVCC解决了脏读，快照读的不可重复和幻读，但是当快照读和当前读同时使用时，仍然可能会发生不可重复读和幻读。下面就来讲这两个问题什么时候会发生以及如何解决。

## RR级别下当前读的幻读问题

回顾一下什么是幻读，幻读就是在一个事务中两次查询某个范围的数据，但查询结果的条数不一样。

### 发生幻读的场景

user表中的数据如下：

![image-20230226215902212](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230226215902212.png)

时序图如下所示，在T5时刻发生了幻读：

![image-20230226215949606](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230226215949606.png)


### 为什么会发生幻读

在T4时刻，由于update语句采用的是当前读，会对事务2新增的记录进行加锁、修改age字段值、修改DB_TRX_ID隐藏字段值。 在T5时刻使用快照读时，根据可见性算法，这条新增记录的DB_TRX_ID是当前事务，所以是可见的，所以输出了三条记录。 T5时刻输出的记录条数和T1时刻的不一样，这就表示发生了幻读。

### 如何解决

在MySQL中提供了next-key lock来解决此类幻读问题。我们要在事务中尽可能早的执行select … for update语句，MySQL会对这个查询的范围加next-key lock来防止其他事务插入新的记录。 next-key lock包含两部分：行锁（record lock），间隙锁（gap lock）。行锁的对象是索引记录项，间隙锁的对象是索引项之间的间隙。

加锁后的时序图如下：

![image-20230226220034558](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230226220034558.png)


注：101 U 102 U (102, +∞)表示 对id=101的索引项加行锁 + 对id=102的索引项加行锁 + 对id在(102,+∞)的间隙加间隙锁

## RR级别下的不可重复读问题

回顾一下什么是不可重复读，不可重复读就是在一个事务中两次查询同一条数据，但由于其他事务的修改导致两次看到的数据结果不一样。

### 发生不可重复度的场景

user表中的数据如下：

![image-20230226220054031](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230226220054031.png)

时序图如下所示，在T5时刻发生了不可重复读：

![image-20230226220117991](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230226220117991.png)


### 为什么会发生不可重复读

在T4时刻，由于update语句采用的是当前读，所以会对这条记录进行加锁、修改name字段值、修改DB_TRX_ID隐藏字段值。 在T5时刻使用快照读时，根据可见性算法，这条记录最新版本的DB_TRX_ID是当前事务，所以是可见的。 T5时刻读到了事务2的修改，这就表示发生了不可重复读。

### 如何解决

可以使用行锁来解决此类问题。我们要在事务中尽可能早的执行select … for update语句，MySQL会对这个记录加行锁来防止其他事务修改此记录。

加锁后的时序图如下：

![image-20230226220136003](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230226220136003.png)


注：如果将此场景中的id=1改为id&gt;=1则加的锁是next-key lock。更


---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/mysql-rr%E7%BA%A7%E5%88%AB%E4%B8%8B%E4%BB%8D%E7%84%B6%E4%BC%9A%E5%8F%91%E7%94%9F%E5%B9%BB%E8%AF%BB%E5%92%8C%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB/  

