### Cue

会刷脏页的场景

控制脏页比例要考虑的因素

如何计算刷脏页的速度

如何避免刷脏页影响性能

有趣的“连坐机制”

### Notes

#### 脏页

内存页和磁盘页有差异的数据页，称为脏页

#### MySQL会进行刷脏页的场景

1. redo log写满

   此时，会移动redo log的checkpoint，同时，把对应覆盖掉的redo log中包含的全部脏页刷到磁盘中。出现这种情况时，系统所以的更新都会被堵住，需要极力避免这种情况

2. 内存不足

   会淘汰最久不使用的内存页，如果是脏页，就把脏页刷到磁盘上在使用改内存页。如果一个查询淘汰的脏页太多，就会导致查询时间变慢

3. 空闲时间

4. 正常关闭

#### 设置控制脏页的策略要考虑的因素

innodb_io_capacity：表示你磁盘的能力，可以通过fio这个工具来测试磁盘的性能 ` fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest `

脏页比例：innodb_max_dirty_pages_pct表示脏页比例上限，默认值75%，

redo log写盘速度：当前写入位置到checkpoint的差值

####如何计算刷脏页的速度

根据脏页比例M，通过算法F1(M)得到一个1-100之间的值

根据redo log写盘速度N，通过算法F2(N)得到一个1-100之间的值

取F1(M)和F2(N)中较大的值认作R，再用innodb_io_capacity*R%即InnoDB刷脏页的速度

#### 如何避免刷脏页影响性能

合理的设置innodb_io_capacity的值

多关注脏页比例，不要让它接近75%

脏页比例通过Innodb_buffer_pool_pages_dirty/Innodb_buffer_pool_pages_total得到

具体代码如下：

```
mysql> select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;
```

####一个有趣的策略

MySQL flush掉一个脏页时，可能会连带‘邻居’脏页一起flush掉，有时后这个机制会导致一个普通的脏页连带很多张脏页一起被flush掉，导致性能问题。     
这个方法在机械硬盘时代是非常有意义的，可以减少随机IO次数。
固态硬盘主机建议将`innodb_flush_neighbors`设置为0来取消这个机制
MySQL 8.0之后`innodb_flush_neighbors`的默认值已经为0了

### Summary

理解了MySQL为什么偶尔会抖一下的原因，了解了脏页的概念，以及如何设计一个合适的刷脏页速度

