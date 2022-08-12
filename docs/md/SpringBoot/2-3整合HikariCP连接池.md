# 2.3、整合HikariCP连接池

## HikariCP介绍

​		HikariCP是由日本程序员开源的一个数据库连接池组件，代码非常轻量，并且速度非常的快。根据官方提供的数据，在i7,开启32个线程32个连接的情况下，进行随机数据库读写操作，HikariCP的速度是现在常用的C3P0数据库连接池的数百倍。在SpringBoot2.0中，官方也是推荐使用HikariCP。
![banner文件位置](https://raw.githubusercontent.com/dililidili/study-docs/main/docs/img/SpringBoot/HikariCP1.png)

## 为什么HikariCP会那么快

1. 字节码更加精简，所以可以加载更多代码到缓存。
2. 实现了一个无锁的集合类型，来减少并发造成的资源竞争。
3. 使用了自定义的数组类型，相对与ArrayList极大地提升了性能。
4. 针对CPU的时间片算法进行优化，尽可能在一个时间片里面完成各种操作。

## 与Druid对比

​		在github上有网友贴出了阿里巴巴Druid与hikari的对比，认为hikari在性能上是完全秒杀阿里巴巴的Druid连接池的。对此，阿里的工程师也做了一定的回应，说Druid的性能稍微差点是锁机制的不同，并且Druid提供了更丰富的功能，两者的侧重点不一样。

## 如何选择：

​		选择哪一款就见仁见智了，不过两款都是开源产品，阿里的Druid有中文的开源社区，交流起来更加方便，并且经过阿里多个系统的实验，想必也是非常的稳定，而Hikari是SpringBoot2.0默认的连接池，全世界使用范围也非常广，对于大部分业务来说，使用哪一款都是差不多的，毕竟性能瓶颈一般都不在连接池。大家可根据自己的喜好自由选择。

### pom依赖

```
 <dependency>
   <groupId>com.zaxxer</groupId>
   <artifactId>HikariCP</artifactId>
   <version>5.0.1</version>
</dependency>
```

### application.yml配置

```
# 配置数据源信息
spring:
  datasource:                                           # 数据源的相关配置
    type: com.zaxxer.hikari.HikariDataSource          # 数据源类型：HikariCP
    driver-class-name: com.mysql.jdbc.Driver          # mysql驱动
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
    username: root
    password: 123456
    hikari:
      connection-timeout: 30000        # 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 默认:30秒
      minimum-idle: 5                  # 最小连接数
      maximum-pool-size: 20            # 最大连接数
      auto-commit: true                # 事务自动提交
      idle-timeout: 600000             # 连接超时的最大时长（毫秒），超时则被释放（retired），默认:10分钟
      pool-name: DateSourceHikariCP     # 连接池名字
      max-lifetime: 1800000             # 连接的生命时长（毫秒），超时而且没被使用则被释放（retired），默认:30分钟 1800000ms
      connection-test-query: SELECT 1  # 连接测试语句
```

