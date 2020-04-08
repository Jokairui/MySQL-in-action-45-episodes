### Cue

binlog的写入机制

write和fsync的概念

sync_binlog参数

innodb_flush_log_at_trx_commit参数

组提交

### Notes

#### binlog的写入机制

1. 每个线程都分配一个binlog cache
2. 当binlog cache使用大小超过了binlog_cache_size之后，就要将多余的内容暂存磁盘
3. 事务提交时，执行器将binlog cache中的完整事务写入到binlog中

#### write和fsync

write将binlog cache写到文件系统中的Page Cache中

fsync才真正写入磁盘

#### sync_binlog

1. sync_binlog = 0时，每次提交的事务都只write，不fsync
2. sync_binlog = 1时，每次提交都执行fsync
3. sync_binlog = N时，每次提交都write，累计N次事务之后才fsync。 **但是，将 sync_binlog 设置为 N，对应的风险是：如果主机发生异常重启，会丢失最近 N 个事务的 binlog 日志。**

#### redo log buffer

在一个事务执行完之前，所有的变动都先写到一块内存中，称为redo log buffer

有一个后台线程，每秒一次将redo log buffer write到Page Cache，再fsync到磁盘上

#### innodb_flush_log_at_trx_commit参数

1. 设置为0时，每次事务提交都只把redo log 留在redo log buffer中
2. 设置为1时，每次事务提交都把redo log fsync到磁盘中
3. 设置为2时，每次事务提交都只把redo log写到Page Cache中

#### 导致未提交事务的redo log写盘的两种场景

1. redo log buffer占用的空间即将达到innodb_log_buffer_size的一半的时候
2. 并行事务提交的时候，顺带把这个未提交的事务的redo log写盘

#### 组提交

n个并行事务同时处于prepare之后，第一个事务redo log进入commit状态并且执行fsync，会同时将其他prepare的事务给一起fsync掉

同样的binlog也可以实现组提交

#### 两个组提交相关的参数

1. binlog_group_commit_sync_delay，表示事务提交后binlog延迟多少秒再执行fsync
2. binlog_group_commit_sync_no_delay_count，表示累计多少次提交后再执行fsync

这两个参数是逻辑或的关系

### Summary

了解了binlog和redo log具体的写入机制，了解了write和fsync的区别，还有组提交的概念。

