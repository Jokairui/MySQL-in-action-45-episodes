### Cue

MySQL是如何统计索引信息的

影响MySQL选择索引的因素有哪些？扫描行数，排序，回表等

### Notes

####Mysql索引统计数字不准

可以通过show index from table查看当前表的索引具体行数

使用采样统计的方式，随机抽取N个数据页，取平均值*总数据页，估计出索引行数，当数据行数变化超过1/M时，重新出发采样统计

innodb_stats_persistent为on时，N为20，M为10，统计信息持久化

innodb_stats_persistent为on时，N为8，M为16，统计信息存放在内存中

**通过analyze table t命令重新采样索引**

#### 考虑回表的代价

使用二级索引要回表，全表查询不需要回表

#### 考虑order by，而采用了order by相关的字段的索引

解决方法，force index

修改sql语句，例如order by b改为order by b，a

重建更合适的索引

### Summary

了解了MySQL统计索引信息的方法，了解了几种导致MySQL选错索引的原因，以及相应的解决办法

