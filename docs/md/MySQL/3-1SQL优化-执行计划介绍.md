# 3.1、SQL优化-执行计划介绍

## 1、介绍

​		在企业的应用场景中，为了知道优化SQL语句的执行，需要查看SQL语句的具体执行过程，以加快SQL语句的执行效率。可以使用explain+SQL语句来模拟优化器执行SQL查询语句，从而知道mysql是如何处理sql语句的。

## 2、执行计划列信息

|     字段      |               含义                |
| :-----------: | :-------------------------------: |
|      id       |             查询标识              |
|  select_type  |             查询类型              |
|     table     |      表明/别名/子查询虚拟名       |
|  partitions   |          匹配到的表分区           |
|     type      |             访问类型              |
| possible_keys |         可能会用到的索引          |
|      key      |          实际使用的索引           |
|    key_len    | 索引中使用的字节数,索引使用的长度 |
|      ref      |           使用索引的列            |
|     rows      |             扫描行数              |
|   filtered    |      按表条件过滤的行百分比       |
|     extra     |             额外信息              |

### 2.1、id列

select查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序。

id决定了执行顺序 `id越大优先执行,id相同从上到下顺序执行`。

### 2.2、select_type列

|       查询类型       |                含义                |
| :------------------: | :--------------------------------: |
|        SIMPLE        |     简单查询没有union和子查询      |
|       PRIMARY        |      包含子查询的外层语句类型      |
|        UNION         |           union后的语句            |
|   DEPENDENT UNION    |       子查询中union后的语句        |
|     UNION RESULT     |        union前后SQL的结果集        |
|       SUBQUERY       |               子查询               |
|  DEPENDENT SUBQUERY  | 子查询需要是一个结果集而不是单个值 |
|       DERIVED        |       衍生表即from后是子查询       |
| UNCACHEABLE SUBQUERY |           不缓存的子查询           |
|  UNCACHEABLE UNION   |         不缓存的union语句          |

#### SIMPLE:简单的查询，不包含子查询和union

```
explain select * from A;
```

#### PRIMARY:查询中若包含任何复杂的子查询，最外层查询则被标记为primary

```
explain select C.name  from (select name,count(name) from A group by name) C  
```

#### UNION:若第二个select出现在union之后，则被标记为union

```
explain select * from A where id = 10 union select * from B where id >2000;
```

##### 结果：

| id   | select_type  |
| ---- | ------------ |
| 1    | PRIMARY      |
| 2    | UNION        |
| NULL | UNION RESULT |

#### DEPENDENT UNION:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
```
explain select * from A where A.id  in ( select id from A where A.id = 10 union select id from B where B.id >2000)
```

| id   | select_type        |
| ---- | ------------------ |
| 1    | PRIMARY            |
| 2    | DEPENDENT SUBQUERY |
| 3    | DEPENDENT UNION    |
| NULL | UNION RESULT       |

#### UNION RESULT:从union表获取结果的select
```
explain select * from A where id = 10 union select * from B where id >2000;
```

##### 结果：

| id   | select_type  |
| ---- | ------------ |
| 1    | PRIMARY      |
| 2    | UNION        |
| NULL | UNION RESULT |

#### SUBQUERY:在select或者where列表中包含子查询
```
explain select * from A where A.id > (select B.id from B)
```

##### 结果：

| id   | select_type |
| ---- | ----------- |
| 1    | PRIMARY     |
| 2    | SUBQUERY    |

#### DEPENDENT SUBQUERY::subquery的子查询要受到外部表查询的影响
```
explain select * from A  where A.s_id in (select distinct s_id from B);
```

#### DERIVED: from子句中出现的子查询，也叫做派生类，
```
EXPLAIN SELECT * FROM (SELECT A.id, count(*) as cou FROM A GROUP BY id) AS C where C.id > 1;
```

##### 结果：

| id   | select_type |
| ---- | ----------- |
| 1    | PRIMARY     |
| 2    | DERIVED     |

#### UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存

```
 explain select * from A where A.id = (select id from B where id=@@sort_buffer_size);
```

##### 结果：

| id   | select_type          |
| ---- | -------------------- |
| 1    | PRIMARY              |
| 2    | UNCACHEABLE SUBQUERY |

## table

对应行正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集

1. 如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名。
2. 表名是derivedN的形式，表示使用了id为N的查询产生的衍生表。
3. 当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

### **type**

​		type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下，访问的类型有很多，

##### 效率从最好到最坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL 

一般情况下，得保证查询至少达到range级别，最好能达到ref

##### all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。

```
explain select * from A;
```

##### index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序

```
explain  select name from A;
```

##### range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 

```
explain select * from A where s_id between 7000 and 7500;
```

##### index_subquery：利用索引来关联子查询，不再扫描全表

```
explain select * from A where A.name in (select name from B);
```

##### unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引

```
explain select * from A  where A.s_id in (select distinct B.s_id from B);
```

##### index_merge：在查询过程中需要多个索引组合使用

##### ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式

```
explain select * from A  where  a.name is null or a.name=7369;
```

##### ref：使用了非唯一性索引进行数据的查找

```
explain select * from A,B  where A.name =B.name;
```

##### eq_ref ：使用唯一性索引进行数据查找

```
explain select * from A,B where A.name = B.name;
```

##### const：这个表至多有一个匹配行，

```
explain select * from A where name = 1;
```

##### system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现

### **possible_keys** 

​        显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

```sql
explain select * from A,B where A.name = B.name and B.name = 10;
```

### **key**

​		实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠。

```sql
explain select * from A,B where A.name = B.name and B.deptno = 10;
```

### **key_len**

表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好。

```sql
explain select * from A,B where A.name = B.name and B.deptno = 10;
```

### **ref**

显示索引的哪一列被使用了，如果可能的话，是一个常数

```sql
explain select * from A,B where A.name = B.name and B.deptno = 10;
```

### **rows**

根据表的统计信息及索引使用情况，大致估算出找出所需记录需要读取的行数，此参数很重要，直接反应的sql找了多少数据，在完成目的的情况下越少越好

```sql
explain select * from A;
```

### **extra**

包含额外的信息。

```sql
--using filesort:说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
explain select * from A order by name;

--using temporary:建立临时表来保存中间结果，查询完成之后把临时表删除
explain select name,count(*) from A where s_id = 10 group by name;

--using index:这个表示当前的查询时覆盖索引的，直接从索引中读取数据，而不用访问数据表。如果同时出现using where 表名索引被用来执行索引键值的查找，如果没有，表面索引被用来读取数据，而不是真的查找
explain select name,count(*) from A group by s_id limit 10;

--using where:使用where进行条件过滤
explain select * from A where id = 1;

--using join buffer:使用连接缓存

--impossible where：where语句的结果总是false
explain select * from A where name = 7469;
```

