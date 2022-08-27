# 3.3、查询优化

## 1.排序优化

### 1.1、排序算法

#### 1、两次传输排序

第一次数据读取是将需要排序的字段读取出来，然后进行排序，第二次是将排好序的结果按照需要去读取数据行。
这种方式效率比较低，原因是第二次读取数据的时候因为已经排好序，需要去读取所有记录而此时更多的是随机IO，读取数据成本会比较高。
两次传输的优势，在排序的时候存储尽可能少的数据，让排序缓冲区可以尽可能多的容纳行数来进行排序操作。

#### 2、一次传输排序

​		先读取查询所需要的所有列，然后再根据给定列进行排序，最后直接返回排序结果，此方式只需要一次顺序IO读取所有的数据，而无须任何的随机IO，问题在于查询的列特别多的时候，会占用大量的存储空间，无法存储大量的数据。

#### 3、max_length_for_sort_data

​		当需要排序的列的总大小超过max_length_for_sort_data定义的字节，mysql会选择双次排序，反之使用单次排序，当然，用户可以设置此参数的值来选择排序的方式

## 2.JOIN优化

mysql的join算法叫做Nested-Loop Join（嵌套循环连接）

而这个Nested-Loop Join有三种变种:

#### 2.1、Simple Nested-Loop

这个算法相当简单、直接。即驱动表中的每一条记录与被驱动表中的记录进行比较判断（就是个[笛卡尔积](https://so.csdn.net/so/search?q=笛卡尔积&spm=1001.2101.3001.7020)）。对于两表联接来说，驱动表只会被访问一遍，但被驱动表却要被访问到好多遍

假设R为驱动表，S被驱动表，用伪代码表示一下这个过程就是这样：

```
for r in R                      # 扫描R表（驱动表）
    for s in S                   # 扫描S表（被驱动表）
        if (r and s satisfy the join condition)  # 如果r和s满足join条件
            output result    # 返回结果集
```

所以如果R有1万条数据，S有1万条数据，那么数据比较的次数`1万 * 1万 =1亿次`，这种查询效率会非常慢。

### 2.2、Index Nested-Loop

这个是基于索引进行连接的算法

它要求被驱动表上有索引，可以通过索引来加速查询。

假设R为驱动表，S被驱动表，用伪代码表示一下这个过程就是这样：

```
for r in R                  # 扫描R表
    for s in Sindex                    # 查询S表的索引（固定3~4次IO，B+树高度）
        if (s == r)                   # 如果r匹配了索引s
            output result   # 返回结果集
```

### 2.3、Block Nested-Loop

这个算法较Simple Nested-Loop Join的改进就在于可以减少被驱动表的扫描次数

因为它使用Join Buffer来减少内部循环读取表的次数

假设R为驱动表，S被驱动表，用伪代码表示一下这个过程就是这样：

```
for r in R                             # 扫描表R
    store p from R in Join Buffer    # 将部分或者全部R的记录保存到Join Buffer中，记为p
    for s in S                        # 扫描表S
        if (p and s satisfy the join condition)        # p与s满足join条件
           output result                    # 返回为结果集
```

可以看到相比Simple Nested-Loop Join算法，Block Nested-LoopJoin算法仅多了一个所谓的Join Buffer；

Join Buffer用以缓存联接需要的列，然后以Join Buffer批量的形式和被驱动表中的数据进行联接比较。

> Join Buffer会缓存所有参与查询的列而不是只有Join的列。
>
> join_buffer_size的默认值是256K

### 2.4、总结

在选择Join算法时，会有优先级：

Index Nested-LoopJoin > Block Nested-Loop Join > Simple Nested-Loop Join

当不使用Index Nested-Loop Join的时候，默认使用Block Nested-Loop Join。

使用Block Nested-Loop Join算法需要开启优化器管理配置的optimizer_switch的设置block_nested_loop为on，默认为开启。
Join优化

通过上面的简单介绍，可以总结出以下几种优化思路

1.用小结果集驱动大结果集，减少外层循环的数据量

2.如果小结果集和大结果集连接的列都是索引列，mysql在join时也会选择用小结果集驱动大结果集，因为索引查询的成本是比较固定的，这时候外层的循环越少，join的速度便越快。

3.为匹配的条件增加索引：争取使用Index Nested-Loop Join，减少内层表的循环次数

4.增大join buffer size的大小：当使用Block Nested-Loop Join时，一次缓存的数据越多，那么外层表循环的次数就越少，减少不必要的字段查询：

5.当用到Block Nested-Loop Join时，字段越少，join buffer 所缓存的数据就越多，外层表的循环次数就越少；

## 3、细节优化

#### 3.1、count优化

只有没有任何where条件的myisam下 count 比较快

在某些应用场景中，不需要完全精确的值，可以参考使用近似值来代替，比如可以使用explain来获取近似的值
其实在很多OLAP的应用中，需要计算某一个列值的基数，有一个计算近似值的算法叫hyperloglog。

一般情况下，count()需要扫描大量的行才能获取精确的数据，其实很难优化，在实际操作的时候可以考虑使用索引覆盖扫描，或者增加汇总表，或者增加外部缓存系统。

**注意：count(*) count(id) count(1)效率是一样的**

#### 3.2、join优化

1.确保on或者using子句中的列上有索引，在创建索引的时候就要考虑到关联的顺序。

2.确保任何的groupby和order by中的表达式只涉及到一个表中的列，这样mysql才有可能使用索引来优化这个过程。

3.子查询尽量使用关联查询替代，子查询会产生临时表占用IO。

当表A和表B使用列C关联的时候，如果优化器的关联顺序是B、A，那么就不需要再B表的对应列上建上索引，没有用到的索引只会带来额外的负担，一般情况下来说，只需要在关联顺序中的第二个表的相应列上创建索引

#### 3.3、limit优化

在很多应用场景中我们需要将数据进行分页，一般会使用limit加上偏移量的方法实现，同时加上合适的order by 的子句，如果这种方式有索引的帮助，效率通常不错，否则的化需要进行大量的文件排序操作，还有一种情况，当偏移量非常大的时候，前面的大部分数据都会被抛弃，这样的代价太高。要优化这种查询的话，要么是在页面中限制分页的数量，要么优化大偏移量的性能。

**优化此类查询的最简单的办法就是尽可能地使用覆盖索引，而不是查询所有的列。**

```
select name,age from A  limit 50,5;
//优化为
select name,age from A join (select id from A limit 50,5) B on A.id = B.id 
```

#### 3.4、union优化

mysql总是通过创建并填充临时表的方式来执行union查询，因此很多优化策略在union查询中都没法很好的使用。经常需要手工的将where、limit、order by等子句下推到各个子查询中，以便优化器可以充分利用这些条件进行优化。

除非确实需要服务器消除重复的行，否则一定要使用union all，因此没有all关键字，mysql会在查询的时候给临时表加上distinct的关键字，这个操作的代价很高。

