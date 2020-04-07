### Cue

不同引擎下的实现

如何提升性能

redis方案有什么缺陷

### Notes

**背景是基于无where条件的count(\*)查询**

####MyISAM VS InnoDB

MyISAM是把一个表的总行数存在磁盘上，查询时直接返回这个值，效率很高

InnoDB每一次要把每一行数据拉出来，计算返回值，因为InnoDB支持事务，会有不同的一致性视图，本身看到的总行数就不一样

#### show table status VS count(*)

Show table status 也是使用的采样统计，官方表示误差可能有40%到50%

count(*)自带优化，会选择最小的那颗索引数来遍历，以保证只扫描最小的数据量，但是遍历依然会带来性能问题

#### 提升性能的方案

1. redis等缓存中间件的方案

   几个缺陷：

   * 异常重启丢失数据，可以通过重庆单独count一次，或者通过高可用redis的策略来解决
   * 数据不一致性，因为redis count + 1 与插入数据无法实现事务性，就会导致另一个session查询的值有误差

2. 建立一张计数表

   利用MySQL crash-safe的特性，以及InnoDB的事务特性，解决了redis方案的两个缺陷

#### count(*) VS count(主键) VS count(1) VS count(字段)

引擎层遵循要什么给什么的原则，所以count(主键) 、count(1) 、count(字段)都会取出对应值

count(1) InnoDB 引擎遍历整张表，但不取值。server 层对于返回的每一行，放一个数字“1”进去，判断是不可能为空的，按行累加。

count(字段) 会判断是否非null，且只累加非null的行数

count(*) 做了特殊优化，只计算行数， 不取值

所以count(1) 性能跟count(*) 差不多，但是count(1)会让人误解是count第一列，所以还是count(\*)最棒

### Summary

了解count(*)的实现机制，以及不同引擎下的实现方式

了解了提升count(*)性能的方法，以及不同count的差异

