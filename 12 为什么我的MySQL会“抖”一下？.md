### 脏页和干净页
内存与磁盘数据不一致的数据页，称为脏页，反之，为干净页    
将内存数据写入磁盘的过程，称为刷脏页（flush）    
### 发生flush的场景
1. redo log写满了
2. 服务器内存不足了
3. MySQL认为系统空闲了
4. MySQL正常关闭了
### 对应场景对性能的影响
1. **非常严重**，需要尽量避免，当redo log被写满时，所以的更新都会阻塞
2. 是一种常态，InnoDB本身会维护一个Buffer Pool管理内存，为了维护一个正常的脏页比例，InnoDB在申请新的数据页时，可能会淘汰掉很久不用的一些数据页。
3. 没有性能影响
4. 没有性能影响
### InnoDB 刷脏页的控制策略
首先，要告诉InnoDB主机的IO性能，设置`innodb_io_capacity`这个参数。    
可以通过`fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest`这个命令测试磁盘的IOPS    
设计控制策略可以这么考虑，刷的慢怎么办？    
首先，脏页刷的慢，比例会上升，redo log可能会写满，所以设计策略其实要考虑两个因素：    
1. 脏页比例
2. redo log写盘速度

InnoDB会根据这两个因素先单独算出两个数字M和N    
参数`innodb_max_dirty_pages_pct`是最大脏页比例，第一个数字M拿当前脏页数与最大脏页数比较，超过当前脏页数返回100，否则返回当前值
第二个数字N，InnoDB每次写入日志有一个序号，根据该序号的值与checkpoint的差值，InnoDB会计算出N，差值越大，N越大
最后InnoDB取M和N中最大的值设为R，在用`innodb_io_capacity` 乘以R%来控制刷脏页的速度  
所以可以发现，要尽量避免脏页数达到阀值，避免MySQL全力刷脏页导致**所有更新阻塞**    
**当前脏页比例**可以通过`Innodb_buffer_pool_pages_dirty` / `Innodb_buffer_pool_pages_total`得到,具体代码如下：    
`mysql> select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty'; `    
`select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total'; `    
`select @a/@b;`
### 一个有趣的策略
MySQL flush掉一个脏页时，可能会连带‘邻居’脏页一起flush掉，有时后这个机制会导致一个普通的脏页连带很多张脏页一起被flush掉，导致性能问题。     
这个方法在机械硬盘时代是非常有意义的，可以减少随机IO次数。
固态硬盘主机建议将`innodb_flush_neighbors`设置为0来取消这个机制
MySQL 8.0之后`innodb_flush_neighbors`的默认值已经为0了

