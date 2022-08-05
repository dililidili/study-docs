### 2.4、整合MyBatis

#### MyBatis介绍

​		MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Ordinary Java Object,普通的 Java对象)映射成数据库中的记录。

#### MyBatis 使用

##### pom依赖

```
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

##### application.yml配置文件

```
mybatis:
 	#实体类
	type-aliases-package: com.demo.pojo 
	#接口的配置文件的位置 
	mybatis.mapper-locations: classpath:mybatis/mapper/*.xml  
```

#### Application.class启动类添加

```
//扫描mapper/dao层
@MapperScan({"com.demo.mapper"})
```

