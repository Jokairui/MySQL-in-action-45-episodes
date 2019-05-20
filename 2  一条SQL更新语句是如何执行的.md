## 2  一条SQL更新语句是如何执行的

####redo log（重做日志）： 

* InnoDB引擎层特有日志, 数据更新会先写redo log，再更新内存，在适当的时候，将这个操作记录到磁盘 

* 有大小，一般是一组文件，从头开始写，写到尾时就要持久化并擦除之前的记录。 

* write position ： 文件写入点；checkpoint：文件擦除点；当writepostion追上checkpoint就说明文件写满了。 

* 记录的是***做了什么改动***，而不是***更新了什么状态***

####binlog（归档日志）： 

* Server层日志，在redo log写完之后写入 

* 记录的是原始逻辑，有两种模式，statement时记录SQL语句，rows时记录行的内容，两条，更新前和更新后
* 不同与redo log，空间是不固定的 

####WAL（Write-Ahead Logging）

即先写redo log，再写磁盘 



####Crash-safe

有了redo log，即使mysql异常重启了，也可以通过磁盘恢复状态 

####两阶段提交

引擎层写完redo log后，redo log处于prepare状态，告知执行器，执行器生成对应binlog之后， 调用引擎提交事务接口，事务提交成功后，redo log处于commit状态。 



####为什么要有两阶段提交？ 

假设没有它，且在两次写日志之间数据库crash了： 

* 先写redo log，再写binlog情况：数据库恢复后，通过binlog恢复时，失去了一个事务，导致对应值与原库不同了 

* 先写binlog，再写redo log：数据库恢复后，多了一个事务，导致对应值与原库不同 

**在写完binlog之后和事务提交成功之前这段时间内数据库crash了的话，通过binlog恢复之后，会认可这个事务，并提交这个事务。**



####sync_binlog

这个值设为1，可以让每次写完binlog都flush到磁盘

####innodb_flush_log_at_trx_commit

这个值设为1，可以让每次写完的redo log都flush到磁盘