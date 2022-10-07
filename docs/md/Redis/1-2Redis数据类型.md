# 1.2、Redis数据类型

## 一、下载

1.访问redis官网：[Redis](https://redis.io/)

![img](https://img-blog.csdnimg.cn/20200622144305226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MzUxMTc3,size_16,color_FFFFFF,t_70)编辑

2.点击按钮开始下载redis最新稳定版

3.下载不下来的这里提供一个5.0.9版本的：

链接：https://pan.baidu.com/s/1Z8gD6NiZfrvriRit77hjkA  提取码：fv04

## 二、Java验证redis是否启动成功

**1.引用jar包**

```
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
```

**2.java代码**

```
public static void main(String[] args) {
    Jedis jedis = new Jedis("127.0.0.1");
    System.out.println("服务正在运行: "+jedis.ping());
}
```



**3.运行结果**

启动失败：抛出redis.clients.jedis.exceptions.JedisConnectionException: Failed connecting to 127.0.0.1:6379异常

启动成功：输出 服务正在运行: PONG

## 三、字符串(string)

**1.支持存储类型**

| **类型** | **取值范围** |
| -------- | ------------ |
| 字节串   |              |
| 整数     | long integer |
| 浮点数   | double       |



**2.整数和浮点数支持自增和自减操作**

**3.命令 提示：windows系统双击redis-cli.exe即可测试命令**

| **命令**        | **描述**                             | **用例**                | **返回**             | **JAVA方法**                                                 |
| --------------- | ------------------------------------ | ----------------------- | -------------------- | ------------------------------------------------------------ |
| SET             | 设置指定key的value                   | set name zhangsan       | ok                   | `String set(String key, String value)`                       |
| GET             | 返回指定key的value                   | get name                | zhangsan             | `String get(String key)`                                     |
| GETSET          | 返回指定key的value并赋予新value      | getset name lisi        | zhangsan             | `String getSet(String key, String value)`                    |
| GETRANGE        | 返回指定key的部分字符串              | getrange name 0 2       | lis                  | `String getrange(String key, long startOffset, long endOffset)` |
| SETRANGE        | 设置指定key的部分字符串              | getrange name 0 ll      | 7                    | `Long setrange(String key, long offset, String value)`       |
| MGET            | 返回指定多个key的value               | mget name age           | lisi 2               | `List<String> mget(String... keys)`                          |
| MSET            | 设置多个指定key的value               | mset name zhaowu age 20 | ok                   | `String mset(String... keysvalues)`                          |
| SETEX           | 设置指定key的value并设置过期时间(秒) | setex set 60 man        | ok                   | `String setex(String key, int seconds, String value)`        |
| SETNX           | 当指定key不存在时才可设置成功        | setnx set name          | 0                    | `Long msetnx(String... keysvalues)`                          |
| STRLN           | 返回指定key的value长度               | strln name              | 6                    | `Long strlen(String key)`                                    |
| INCR            | 对指定key进行自增                    | incr age                | 21                   | `Long incr(String key)`                                      |
| DECR            | 对指定key进行自减                    | decr age                | 20                   | `Long decr(String key)`                                      |
| INCRBY          | 对指定key加上指定整数                | incrby age 10           | 30                   | `Long incrBy(String key, long increment)`                    |
| DECRBY          | 对指定key减去指定整数                | decrby age 10           | 20                   | `Long decrBy(String key, long decrement)`                    |
| INCRBYFLOAT     | 对指定key加上指定浮点数              | incrbyfloat age 1.1     | 31.10000000000000142 | `Double incrByFloat(String key, double increment)`           |
| APPEND          | 对指定key的value追加value            | append name 3           | 7                    | `Long append(String key, String value)`                      |
| OBJECT encoding | 查看指定key的value类型               | object encoding  k1     | int                  |                                                              |



## 四、列表(list)



**1.命令及其java方法**

| **命令**   | **描述**                                                     | **用例**                | **返回**  | **JAVA方法**                                                 |
| ---------- | ------------------------------------------------------------ | ----------------------- | --------- | ------------------------------------------------------------ |
| RPUSH      | 将一个或者多个值插入列表右侧                                 | rpush list car1 car2    | 2         | `Long rpush(String key, String... string)`                   |
| LPUSH      | 将一个或者多个值插入列表左侧                                 | lpush list car3 car4    | 4         | `Long lpush(String key, String... string)`                   |
| RPOP       | 移除并返回最右侧的元素                                       | rpop list               | car2      | `String rpop(String key)`                                    |
| LPOP       | 移除并返回最左侧的元素                                       | lpop list               | car4      | `String lpop(String key)`                                    |
| LINDEX     | 返回列表中指定位置的元素                                     | lindex list 1           | car1      | `String lindex(String key, long index)`                      |
| LTRIM      | 将指定一段元素保留，其余删除                                 | ltrim list 0 0          | ok        | `String ltrim(String key, long start, long stop)`            |
| BLPOP      | 从第一个非空列表中弹出位于最左端的元素或者在N秒内阻塞并等待可弹出元素 | blpop list list1 2      | list car3 | `List<String> blpop(int timeout, String... keys)`            |
| BRPOP      | 从第一个非空中弹出最右侧的元素或者在N秒内阻塞并等待可弹出元素 | brpop list list1 2      | list cc2  | `List<String> brpop(int timeout, String... keys)`            |
| RPOPLPUSH  | 从第一个列表中弹出最右侧的元素然后插入第二个列表的最左侧并返回元素 | rpoplpush list list1    | cc1       | `String rpoplpush(String srckey, String dstkey)`             |
| BRPOPLPUSH | 从第一个列表弹出最右侧的元素插入第二个列表的最左端并返回元素，如果第一个列表为空那么阻塞N秒等待元素的出现 | brpoplpush list list1 3 | (nil)     | `String brpoplpush(String source, String destination, int timeout)` |



## 五、集合(set)



**1.命令及其java方法**

| **命令**    | **描述**                                                   | **用例**             | **返回** | **JAVA方法**                                                 |
| ----------- | ---------------------------------------------------------- | -------------------- | -------- | ------------------------------------------------------------ |
| SADD        | 将一个或多个元素添加到集合里面，并返回添加成功的数量       | sadd set cat1 cat2   | 2        | `Long sadd(String key, String... member)`                    |
| SREM        | 从集合移除一个或多个元素，并返回移除的数量                 | srem set cat1        | 1        | `Long srem(String key, String... member)`                    |
| SISMEMBER   | 检查元素是否存在集合中，并返回存在的数量                   | sismember set cat2   | 1        | `Boolean sismember(String key, String member)`               |
| SRANDMEMBER | 从集合中随机返回一个或多个元素，正数时不重复，负数可能重复 | srandmember set 1    | cat2     | `String srandmember(String key)；``List<String> srandmember(String key, int count)` |
| SPOP        | 随机移除集合中的一个元素，并返回元素                       | spop set             | cat2     | `String spop(String key)；``Set<String> spop(String key, long count)` |
| SMOVE       | 随机移除第一个集合的元素并插入第二个元素中，成功返回1      | smove set set1 1     | 0        | `Long smove(String srckey, String dstkey, String member)`    |
| SDIFF       | 返回存在于第一个集合但不存在于其他集合中                   | sdiff set set1       | cat1     | `Set<String> sdiff(String... keys)`                          |
| SDIFFSTORE  | 将第一个集合存在但其他不存在的元素存储到第一个集合中       | sdiff set1 set       | 1        | `Long sdiffstore(String dstkey, String... keys)`             |
| SINTER      | 返回多个集合同时存在的元素                                 | sinter set set1      | cat1     | `Set<String> sinter(String... keys)`                         |
| SUNION      | 返回多个集合所有元素                                       | sunion set set1      | cat1     | `Set<String> sunion(String... keys)`                         |
| SUNIONSTORE | 将所有集合的元素存储到第一个集合中                         | sunionstore set set1 | 0        | `Long sunionstore(String dstkey, String... keys)`            |

## 六、散列(Hash)

| **命令**     | **描述**                         | **用例**                        | **返回**             | **JAVA方法**                                                 |
| ------------ | -------------------------------- | ------------------------------- | -------------------- | ------------------------------------------------------------ |
| HMGET        | 从散列里面获取一个货多个键的值   | hmget hash name age             | zhangsan 22          | `List<String> hmget(String key, String... fields)`           |
| HMSET        | 为散列插入一个货多个键设置值     | hmset hash name zhangsan age 22 | ok                   | `String hmset(String key, Map<String, String> hash)`         |
| HDEL         | 删除一个货多个键返回删除成功数量 | hdel hash name                  | 1                    | `Long hdel(String key, String... field)`                     |
| HLEN         | 返回散列包含键值的数量           | hlen hash                       | 1                    | `Long hlen(String key)`                                      |
| HEXISTS      | 检查键是否存在散列中             | hexists hash age                | 1                    | `Boolean hexists(String key, String field)`                  |
| HKEYS        | 获取散列所有键                   | hkeys hash                      | age                  | `Set<String> hkeys(String key)`                              |
| HVALS        | 获取所有值                       | hvals hash                      | 22                   | `List<String> hvals(String key)`                             |
| HGETALL      | 获取所有键值                     | hgetall hash                    | age 22               | `Map<String, String> hgetAll(String key)`                    |
| HINCRBY      | 将键的值加上整数                 | hincrby hash age 1              | 23                   | `Long hincrBy(String key, String field, long value)`         |
| HINCRBYFLOAT | 将键的值加上浮点数               | hincrbyfloat age 2.1            | 25.10000000000000142 | `Double hincrByFloat(String key, String field, double value)` |

## **七、有序集合(sorted set)**



| **命令**         | **描述**                                       | **用例**                       | **返回** | **JAVA方法**                                                 |
| ---------------- | ---------------------------------------------- | ------------------------------ | -------- | ------------------------------------------------------------ |
| ZADD             | 将带有给定位置的成员添加到有序集合里           | zadd zset 1 b                  | 1        | `Long zadd(String key, double score, String member)`         |
| ZREM             | 从有序集合中移除给定位置的元素                 | zrem zset 1                    | 0        | `Long zrem(String key, String... members)`                   |
| ZCRAD            | 返回有序集合的数量                             | zcrad zset                     | 1        | `Long zcard(String key)`                                     |
| ZINCRBY          | 将某元素排序加N                                | zincrby zset 1 b               | 2        | `Double zincrby(String key, double increment, String member)` |
| ZCOUNT           | 获取某范围内的成员数量                         | zcount zset 1 2                | 1        | `Long zcount(String key, double min, double max)`            |
| ZRANK            | 返回成员的排名                                 | zrank zset b                   | 0        | `Long zrank(String key, String member)`                      |
| ZSCORE           | 返回成员的分值                                 | zscore zset b                  | 2        | `Double zscore(String key, String member)`                   |
| ZRANGE           | 返回有序集合排名范围内成员                     | zrange zset 0 -1               | b        | `Set<String> zrange(String key, long start, long stop)`      |
| ZREVRANK         | 返回有序集合成员所在的位置，按从大到小排序     | zrevrank zset b                | 1        | `Long zrevrank(String key, String member)`                   |
| ZREVRANGE        | 返回有序集合给定排名范围内成员，按从大到小排序 | zrevrange zset 0 -1            | b        | `Set<String> zrevrange(String key, long start, long stop)`   |
| ZREVRANGEBYSCORE | 获取有序集合分值范围内所有成员，按从大到小排序 | zrevrangebyscore zset 0 -1     | b        | `Set<String> zrevrangeByScore(String key, double max, double min)` |
| ZREMRANGEBYRANK  | 移除有序集合中排名范围内的元素                 | zremrangebyrank zset 0 -1      | 1        | `Long zremrangeByRank(String key, long start, long stop)`    |
| ZREMRANGEBYSCORE | 移除有序集合分值在范围内的元素                 | zremrangebyscore zset 0 -1     | 1        | `Long zremrangeByScore(String key, double min, double max)`  |
| ZINTERSTORE      | 对给定有序集合执行交集运算                     | zinterstore zset 2 zset1 zset2 | 0        | `Long zinterstore(String dstkey, String... sets)`            |
| ZUNIONSTORE      | d对给定有序集合执行并集运算                    | zunionstore zset 2 zset1 zset2 | 1        | `Long zunionstore(String dstkey, String... sets)`            |

## **八、发布与订阅(pub/sub)**



| **命令**     | **描述**                                         | **用例**                             |
| ------------ | ------------------------------------------------ | ------------------------------------ |
| SUBSCRIBE    | 订阅给定的一个或多个频道                         | subscribe channel [channel ...]      |
| UNSUBSCRIBE  | r退订给定一个频道，如果未指定退订全部            | unsubscribe [channel [channel ...]]  |
| PUBLISH      | 向指定频道发送消息                               | publish channel message              |
| PSUBSCRIBE   | 订阅与给定模式相匹配的所有频道                   | psubscribe pattern [pattern ...]     |
| PUNSUBSCRIBE | 退订给定的模式，如果没有给定模式那么退订所有模式 | punsubscribe [pattern [pattern ...]] |

java代码推荐阅读：[基于Redis消息的订阅发布应用场景 - JerryMouseLi - 博客园](https://www.cnblogs.com/JerryMouseLi/p/11012839.html)

## 九、过期时间

| **命令** | **描述**                 | **用例**                      |
| -------- | ------------------------ | ----------------------------- |
| PERSIST  | 移除建的过期时间         | persist key-name              |
| TTL      | 查看过期时间秒           | ttl key-name                  |
| EXPIRE   | 设置过期时间秒           | expire key-name seconds       |
| PTTL     | 查看距离过期还有多少毫秒 | pttl key-name                 |
| PEXPIRE  | 设置过期时间毫秒         | pexpire key-name milliseconds |



如有错漏，请各位大佬指正。

