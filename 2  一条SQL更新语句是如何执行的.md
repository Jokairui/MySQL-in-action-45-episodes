##2  一条SQL更新语句是如何执行的

### Cue

什么是redo log？

什么是binlog？

什么是两阶段提交？

两阶段提交的意义是什么？保证任何情况下binlog和redo log的一致性

MySQL是如何保证crash-safe的？

一条SQL更新语句是如何执行的？

### Notes

##### redo log

* 是InnoDB引擎特有的的日志文件

* 是硬盘上的一组文件，用来记录数据库做了什么修改

* 采用了WAL（Write-Ahead Logging）技术，即先记录redo log，再在内存中作数据的更新，最后在适当的时候，将数据写到硬盘上。

* 可以将redo log想象成一个圆，当redo log写满时，就必须擦除最早的log，同时将这些log所代表的改动写到硬盘上。

* 由于MySQL先写redo log，再更新数据的这个特性，就保证了它crash-safe的这一特性
* innodb_flush_log_at_trx_commit这个参数推荐设成1，表示每次写redo log都会直接写在磁盘上

##### binlog

* Server层特有的日志文件
* redo log是物理日志，记录的是“在某个数据页上具体的修改”，binlog是逻辑日志，记录的是SQL语句的原始逻辑——“给ID为2的行的c字段+1“。可以类比为形容一栋房子，物理日志是指”XX栋XX号“，逻辑日志是指”这是Khirye买的房子“
* binlog没有大小限制，可以一直写下去
* sync_binlog推荐设置成1，表示每次事务的binlog都会持久到磁盘上

##### 两阶段提交&UPDATE语句的执行流程

1. 执行器通过引擎获取到对应行
2. 执行器执行对应操作，例如+1，再调用引擎接口写入这行新数据
3. 引擎将改动记录到redo log中，同时在内存中更改了对应的数据，此时redo log便处于prepare状态，表示事务随时可以提交
4. 执行器生成这个事务的binlog，并将binlog写入磁盘
5. 执行器调用引擎层的提交事务接口，引擎将redo log转变为commit状态，更新完成

##### 两阶段提交的意义

* redo log只有进入commit状态，这个事务才被引擎承认，crash重启后引擎会将redo log中记录的更改写到磁盘上，**同时，重启后处于prepare状态的事务会被回滚，对应的binlog也会被回滚掉**
* binlog常用的作用有两个，作为备份用来恢复数据库，以及作为数据源拉起从库



### Summary

redo log是InnoDB特有的引擎层日志，它的通过WAL机制，保证了MySQL拥有crash-safe的特性；binlog是server层的日志，引擎无关的，它与redo log配合，实现了两阶段提交，保证了数据库从crash恢复的情况下，binlog和redo log的一致性。



