## Index Nested-Loop Join

#### 驱动表

可以理解为MySQL先查询的表

#### 被驱动表

可以理解为MySQL后查询的表

例：SELECT * FROM t1 STRAIGHT_JOIN t2 ON t1.a = t2.a

此时， t1是驱动表， t2是被驱动表

连接过程如下

1. 全表扫描t1
2. 根据条件a = t1.a查询t2中的对应数据，如果t2上的a有索引，则可以通过索引查询
3. 满足的结果组成结果集返回