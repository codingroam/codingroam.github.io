# MySQL总结--MVCC（read view和undo log）

MySQL总结--MVCC（read view和undo log）
<!--more-->




  MVCC(Multi-Version Concurrency Control)，即多版本并发控制，数据库通过它能够做到遇到并发读写的时候，在不加锁的前提下实现安全的并发读操作，是一种乐观锁的实现方式，能大大提高数据库的并发性能。

- **当前读**：读取的是记录的最新版本，需要保证其它事务不能修改读取记录，所以会对记录进行加锁。比如 select for update、select lock in share mode、update等，都属于当前读。 
- **快照读**：基于MVCC实现的读，不对读操作加任何锁，读取的时候根据版本链和read view进行可见性判断，所以读取的数据不一定是数据库中的最新值。注意在串行化隔离级别下，读操作也会加锁，所以属于当前读。


## undo log日志版本链


  undo log是一种逻辑日志，当一个事务对记录做了变更操作就会产生undo log，也就是说undo log记录了记录变更的逻辑过程。当一个事务要更新一行记录时，会把当前记录当做历史快照保存下来，多个历史快照会用两个隐藏字段trx_id和roll_pointer串起来（关于隐藏字段，这里不用考虑隐式主键id:DB_ROW_ID），形成一个历史版本链。可以用于MVCC和事务回滚。   比如多个事务对id为1的数据做了更新，会形成如下图这种历史版本链： ![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/20210402040422901.png)

>注：更多关于undo log的内容，在<a href="https://blog.csdn.net/huangzhilin2015/article/details/115396599">MySQL–buffer pool、redo log、undo log、binlog</a>中进行了较为详细的介绍，这里不再赘述。

## read view（读视图）与可见性判断

  在MySQL中，一个事务开启(注意这里说的不是传统意义上的开启，在InnoDB中，begin/start一个事务，并不会立即分配事务id，而是真正执行了操作才会分配事务id)的时候会被分配一个递增的ID。在事务执行快照读的时候，会将**此时**数据库中所有的活跃事务（未提交事务）id列表和已创建的最大事务id(+1)生成一个视图快照，如果是在**可重复读**隔离级别下，这个快照在此事务活跃期间不会变化，如果是**读已提交**隔离级别下，每次快照读都会重新生成。我们从read view中获取两个属性：

- min_trx_id：read view生成时，活跃事务id列表中的最小id 
- max_trx_id：read view生成时，数据库即将分配的事务id，也就是当前已创建最大事务id+1

  一个事务在对一行数据做读取操作的时候，会从undo log历史版本链中从最新版本开始往前比对，通过一系列的规则，根据快照版本中的trx_id字段和read view来确定该版本对于当前事务是否可见，如果当前比对版本不可见，那么就通过roll_pointer找到上一个版本进行比对，直到找到可见版本或找不到任何一个可见版本。这些规则定义如下：

-  如果 trx_id < min_trx_id，则说明该版本对于当前事务(read view)来说，是已提交事务生成的，那么对于当前事务可见。  
-  如果trx_id >= max_trx_id：则说明该版本对于当前事务(read view)来说，是"将来"的事务生成的，那么对于当前事务不可见。  
-  如果min_trx_id <= trx_id < max_trx_id： 
 <ul> 
  - 如果trx_id在read view的**活跃事务id列表**中，则说明该版本对于当前事务(read view)来说，是已开始但未提交的事务生成的，那么对于当前事务**不可见**。 
  - 如果trx_id不在read view的**活跃事务id列表**中，则说明该版本对于当前事务(read view)来说，是已提交的事务生成的，那么对于当前事务**可见**。 
 </ul> 
 <blockquote> 
  <p>注：当前事务id(current_trx_id)也会在活跃事务id列表中，如果undo log是由当前事务生成的，也就是trx_id == current_trx_id，那么此版本对于当前事务来说当然可见</p> 
 </blockquote>  

  另外，在前面undo log的文章中提到过，InnoDB对于删除操作，其实并不是直接删除数据，而是“相当于”一个update操作，也会生成一个对应删除事务的update undo log，只是将delete mark设置为1，之后会由purge线程清理。当根据上述规则比对时发现delete mark为1，就代表该记录已被删除，没有数据。

## 事务id和可见性


  前面提到了，在InnoDB中，begin/start一个事务并不会立即分配事务id，而是真正执行了操作才会分配事务id。这会有什么现象呢？现假设事务A和事务B根据下图时间线执行： ![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/20210411235304921.png)   虽然事务A先begin，但是它在执行do select的时候仍然能看到事务B提交的数据，因为事务在begin的时候并没有真正开始一个事务，事务A的read view是在do select的时候生成的，此时事务B对数据修改的版本快照按照规则来说就属于：trx_id &lt; min_trx_id，属于已提交事务生成，也就对于事务A来说可见。

参考：https://www.jianshu.com/p/8845ddca3b23



---

> 作者: wang  
> URL: https://codingroam.github.io/post/mysql%E6%80%BB%E7%BB%93--mvccread-view%E5%92%8Cundo-log/  

