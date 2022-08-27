# 3.2、SQL优化-索引优化

## 1、索引优点

1. 大大减少了服务器需要扫描的数据量。
2. 帮助服务器避免排序和临时表。
3. 将随机IO转为顺序IO

## 2、索引用处

1. 快速匹配查找where条件
2. 从considereration中消除行，如果可以在多个索引间选择，mysql通常会使用找到最少行的索引。
3. 如果表具有多列索引，则优化器可以使用索引的任何最左前缀来查找行。
4. 当有表连接的时候，从其他表检索行数据。
5. 查找特定索引列的最小值和最大值。
6. 如果排序或分组时在可用索引最左前缀完成的，则对表进行排序和分组。
7. 可以优化查询索引值而无需查询索引值。

## 3、索引匹配方式

##### 条件：表A有索引id(主键)、组合索引(name,age,opt)。

##### 1.全值匹配:使用所有的索引列进行匹配。

```
select * from A where name = 'ss' and age = 19 and opt = 'C'
```

##### 2.匹配最左前缀:只匹配前几列索引。

```
select * from A where name ='ss' and age = 19
```

##### 3.匹配列前缀:匹配列的开头部分

```
select * from A where name like 's%';
select * from A where name like '%s';
```

##### 4.匹配范围值：可以查找某个数据的范围

```
select * from A where name > 20;
```

##### 5.精准匹配某一列，范围匹配另外一列:可以查询第一列全部 第二列部分

```
select * from A where name = 'ss' and age > 20;
```

##### 6.只查询索引列:查询的时候只需要访问索引，不需要数据行(索引覆盖)

```
select name,age,opt from A where name = 'ss' and age = 20 and opt = 'A'
```

> 索引下推:组合索引当第一个索引列匹配上后 在存储引擎里对第二个索引列进行筛选(满足条件下是否是索引下推与版本有关)

## 4、哈希索引

​		基于哈希表的表现，只有精准匹配索引所有列的查询才有效；在mysql中只有memory存储引擎显示支持哈希索引；哈希索引自身只需存储对应的hash值，所以索引结构十分紧凑，同时让哈希索引查询速度非常快。

#### 4.1、哈希索引限制

1. 哈希索引只包含哈希值和行指针，而不存储具体的值，索引不能使用索引中的值来避免读取行。
2. 哈希索引数据并不是按照索引值顺序存储的，所以无法进行排序。
3. 哈希索引不支持部分列索引匹配，哈希索引是使用索引列全部内容进行计算哈希值存储。
4. 哈希索引支持等值比较查询，但是不支持范围查找。
5. 访问哈希索引的数据非常快，除非有很多哈希冲突，当出现哈希冲突时需要遍历链表中所有指针，逐一对比，直到找出所有符合的数据。
6. 哈希冲突比较多的时候，维护成本也很高。

> 当列需要存储大量内容时，并且根据列进行搜索查找，如果使用B+树存储内容会很大，可以将url使用CRC32做哈希
>
> (select id from A where  url_src = CRC32())此查询性能较高，使用了体积很小的索引来完成查询的

## 5、覆盖索引

		### 5.1、介绍

**1.如果一个索引包含了所有需要查询的字段的值，我们称之为覆盖索引。**

**2.不是所有类型的索引都可以称为覆盖索引，覆盖索引必须要存储索引列的值。**

**3.不同的存储的实现覆盖索引的方式不同，不是所有的存储引擎都支持索引覆盖memory就不支持。**

### 5.2、优点

**1、索引条目通常远小于数据行大小，如果只需要读取索引，那么mysql就会极大的较少数据访问量。**

**2、因为索引是按照列值顺序存储的，所以对于IO密集型的范围查询会比随机从磁盘读取每一行数据的IO要少的多。**

**3、一些存储引擎如MYISAM在内存中只缓存索引，数据则依赖于操作系统来缓存，因此要访问数据需要一次系统调用，这可能会导致严重的性能问题。**

**4、由于INNODB的聚簇索引，覆盖索引对INNODB表特别有用。**

### 5.3、例子

1、当发起一个被索引覆盖的查询时，在explain的extra列可以看到using index的信息，此时就使用了覆盖索引

```sql
mysql> explain select store_id,film_id from inventory\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: inventory
   partitions: NULL
         type: index
possible_keys: NULL
          key: idx_store_id_film_id
      key_len: 3
          ref: NULL
         rows: 4581
     filtered: 100.00
        Extra: Using index
1 row in set, 1 warning (0.01 sec)

```

2、在大多数存储引擎中，覆盖索引只能覆盖那些只访问索引中部分列的查询。不过，可以进一步的进行优化，可以使用innodb的二级索引来覆盖查询。

例如：actor使用innodb存储引擎，并在last_name字段又二级索引，虽然该索引的列不包括主键actor_id，但也能够用于对actor_id做覆盖查询

```sql
mysql> explain select actor_id,last_name from actor where last_name='HOPPER'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: actor
   partitions: NULL
         type: ref
possible_keys: idx_actor_last_name
          key: idx_actor_last_name
      key_len: 137
          ref: const
         rows: 2
     filtered: 100.00
        Extra: Using index
1 row in set, 1 warning (0.00 sec)

```

## 6、前缀索引

​		有时候需要索引很长的字符串，这会让索引变的大且慢，通常情况下可以使用某个列开始的部分字符串，这样大大的节约索引空间，从而提高索引效率，但这会降低索引的选择性，索引的选择性是指不重复的索引值和数据表记录总数的比值，范围从1/#T到1之间。索引的选择性越高则查询效率越高，因为选择性更高的索引可以让mysql在查找的时候过滤掉更多的行。

​		一般情况下某个列前缀的选择性也是足够高的，足以满足查询的性能，但是对应BLOB,TEXT,VARCHAR类型的列，必须要使用前缀索引，因为mysql不允许索引这些列的完整长度，使用该方法的诀窍在于要选择足够长的前缀以保证较高的选择性，通过又不能太长。

案例演示：

```sql
--创建数据表
create table citydemo(city varchar(50) not null);
insert into citydemo(city) select city from city;

--重复执行5次下面的sql语句
insert into citydemo(city) select city from citydemo;

--更新城市表的名称
update citydemo set city=(select city from city order by rand() limit 1);

--查找最常见的城市列表，发现每个值都出现45-65次，
select count(*) as cnt,city from citydemo group by city order by cnt desc limit 10;

--查找最频繁出现的城市前缀，先从3个前缀字母开始，发现比原来出现的次数更多，可以分别截取多个字符查看城市出现的次数
select count(*) as cnt,left(city,3) as pref from citydemo group by pref order by cnt desc limit 10;
select count(*) as cnt,left(city,7) as pref from citydemo group by pref order by cnt desc limit 10;
--此时前缀的选择性接近于完整列的选择性

--还可以通过另外一种方式来计算完整列的选择性，可以看到当前缀长度到达7之后，再增加前缀长度，选择性提升的幅度已经很小了
select count(distinct left(city,3))/count(*) as sel3,
count(distinct left(city,4))/count(*) as sel4,
count(distinct left(city,5))/count(*) as sel5,
count(distinct left(city,6))/count(*) as sel6,
count(distinct left(city,7))/count(*) as sel7,
count(distinct left(city,8))/count(*) as sel8 
from citydemo;

--计算完成之后可以创建前缀索引
alter table citydemo add key(city(7));

--注意：前缀索引是一种能使索引更小更快的有效方法，但是也包含缺点：mysql无法使用前缀索引做order by 和 group by。 
```

## 6、使用索引扫描来做排序

​		mysql有两种方式可以生成有序的结果：通过排序操作或者按索引顺序扫描，如果explain出来的type列的值为index,则说明mysql使用了索引扫描来做排序。

​		扫描索引本身是很快的，因为只需要从一条索引记录移动到紧接着的下一条记录。但如果索引不能覆盖查询所需的全部列，那么就不得不每扫描一条索引记录就得回表查询一次对应的行，这基本都是随机IO，因此按索引顺序读取数据的速度通常要比顺序地全表扫描慢。

mysql可以使用同一个索引即满足排序，又用于查找行，如果可能的话，设计索引时应该尽可能地同时满足这两种任务。

​		只有当索引的列顺序和order by子句的顺序完全一致，并且所有列的排序方式都一样时，mysql才能够使用索引来对结果进行排序，如果查询需要关联多张表，则只有当orderby子句引用的字段全部为第一张表时，才能使用索引做排序。order by子句和查找型查询的限制是一样的，需要满足索引的最左前缀的要求，否则，mysql都需要执行顺序操作，而无法利用索引排序。

```sql
只有满足最左匹配和排序都用正序或降序才会生效。
```

## 7、索引小细节

**1.union all,in,or都能够使用索引，但是推荐使用in。**

**2.范围列可以用到索引(<、<=、>、>=、between)，但是范围列后面的列无法用到索引，索引最多用于一个范围列。**

**3.强制类型转换会全表扫描。**

**4.更新十分频繁，数据区分度不高的字段上不宜建立索引：**

- 更新会变更B+树，更新频繁的字段建立索引会大大降低数据库性能
- 类似于性别这类区分不大的属性，建立索引是没有意义的，不能有效的过滤数据
- 一般区分度在80%以上的时候就可以建立索引，区分度可以使用 count(distinct(列名))/count(*) 来计算

**5.创建索引的列，不允许为null，可能会得到不符合预期的结果。**

**6.LETF JOIN表连接的时候，最好不要超过三张表，因为需要join的字段，数据类型必须一致，被关联的表必须上索引。**

**7.如果明确知道返回结果只有一个，limit 1能够提高效率;能使用limit的时候尽量使用limit。**

**8.单表索引建议控制在5个以内。**

**9.单索引字段数不允许超过5个（组合索引）。**

## 8、索引监控

```
show status like 'Handler_read%';
Handler_read_first：读取索引第一个条目的次数
Handler_read_key：通过index获取数据的次数
Handler_read_last：读取索引最后一个条目的次数
Handler_read_next：通过索引读取下一条数据的次数
Handler_read_prev：通过索引读取上一条数据的次数
Handler_read_rnd：从固定位置读取数据的次数
Handler_read_rnd_next：从数据节点读取下一条数据的次数
```



