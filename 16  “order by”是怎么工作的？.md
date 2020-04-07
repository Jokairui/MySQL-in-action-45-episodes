### Cue

全字段排序

Row id 排序

差异、利弊

最优解：不走排序过程，走索引！

### Notes

#### 全字段排序

将select部分中的所有字段都放到sort_buffer中进行排序

#### Row id排序

max_length_for_sort_data表示排序时单行允许的最大长度，单行长度超过这个值，就不用全字段排序而使用Row  id排序

这种排序只在sort_buffer中放id和对应排序列，排好序之后在通过ID再回一次表

#### 全字段排序 VS rowid 排序

**对于InnoDB表**，MySQL实在担心内存太小，才会使用row id 排序，通常row id排序回多回一次表，回表造成更多的随机磁盘读

同时，如果排序字段可以使用索引的话，就不会使用sort_buffer，所以应该尽量让索引覆盖排序字段。

### Summary

理解了不走索引情况下MySQL的排序过程，对比了全字段排序和Row id 排序的差异，以及权衡。

