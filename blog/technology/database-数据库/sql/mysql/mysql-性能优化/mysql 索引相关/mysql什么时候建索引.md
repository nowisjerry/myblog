# mysql什么时候建索引

## 索引简介

### 是什么

1. MySQL官方对索引的定义为:索引（Index）是帮助MySQL高效获取数据的数据结构。可以得到索引的本质:索引是数据结构。
2. **你可以简单理解为"排好序的快速查找数据结构"**。
3. 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上

### 优势

1. 类似大学图书馆建书目索引，提高数据**检索**的效率，降低数据库的IO成本
2. 通过索引列对数据进行**排序**，降低数据排序的成本，降低了CPU的消耗

### 劣势

1. 虽然索引大大提高了查询速度，同时却会**降低更新表的速度**，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，
   都会调整因为更新所带来的键值变化后的索引信息
2. 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要**占用空间**的



### 哪些情况需要创建索引

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引
3. 查询中与其它表关联的字段，外键关系建立索引
4. 单键/组合索引的选择问题，组合索引性价比更高
5. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
6. 查询中统计或者分组字段 (分组里面已经做了排序,所以分组更消耗性能)

### 哪些情况不要创建索引

1. 表记录太少
2. 经常增删改的表或者字段
3. where条件里用不到的字段不创建索引
4. 过滤性不好的不适合建索引 (比如:性别)