## Index Nested-Loop Join

#### 驱动表

可以理解为MySQL先查询的表

#### 被驱动表

可以理解为MySQL后查询的表

例：SELECT * FROM t1 STRAIGHT_JOIN t2 ON t1.a = t2.a

此时， t1是驱动表， t2是被驱动表

连接过程如下

1. 从t1中读取一条数据
2. 根据条件a = t1.a查询t2中的对应数据和t1的数据组成一行，如果t2上的a有索引，则可以通过索引查询
3. 满足的结果组成结果集返回
4. 重复1到3步，知道扫描完t1的最后一行

以上，称为NLJ，可以看出，使用join和不使用join的操作数量是一样的，但是使用join是在和执行多条SQL是不一样的，执行SQL的消耗要大的多



## 如何选择驱动表

## Simple Nested-Loop Join

以上的Join流程称为simple nested-loop join， 个人感觉只是相对后面的BNL取的一个别名叫法

## Block Nested-Loop JOIN

1. 从t1中取出一部分数据，放入join buffer
2. 全表扫描取出t2的数据
3. 将join buffer里的行作为条件去t2中查找出存在的数据，组成结果集的一部分
4. 重复1和3步直至结束



