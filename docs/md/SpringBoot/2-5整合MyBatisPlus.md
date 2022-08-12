# 2.5整合MyBatisPlus

**官网地址：[MyBatis-Plus](https://baomidou.com/)**

## **前言：**

mybatis使用方式是直接在xml中通过SQL语句操作数据库，包括简单的CRUD操作都必须要写SQL语句，而mybatis-plus的出现很好的解决了这个问题，很好的提高基于MyBatis 的项目开发效率。

## 特性：

**无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑

**损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作

**强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求

**支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错

**支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题

**支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作

**支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）

**内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用

**内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询

**分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库

**内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询

**内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

* * *

#### SpringBoot中快速使用

### 引入pom.xml依赖

```
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.3.1.tmp</version>
</dependency>
```

### 配置

**jdbc配置请查看：2.1/2.2**

```
mybatis-plus:
  mapper-locations: classpath:/mybatis-mappers/*Mapper.xml
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.abc.jx
  configuration:
    map-underscore-to-camel-case: true
		default-statement-timeout: 300
```
**service层继承IService**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LWUyYzc1MTViZDgwYWU5NWEucG5n?x-oss-process=image/format,png)

**impl层继承ServiceImpl**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LTFjYmRiYjdkODdkNGE3ZTEucG5n?x-oss-process=image/format,png)

**mapper层继承BaseMapper**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LWNkMTdmMjc4ODY3MGI5OTUucG5n?x-oss-process=image/format,png)

此时配置结束,下面开始使用

**分页**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LTMxZTJlMjBlODM2ZTFjYWQucG5n?x-oss-process=image/format,png)

Page为页码参数

QueryWrapper为条件构造器,下面简单说一下里面的方法

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LTFlY2VhOGZjMjVhZDVhZTAucG5n?x-oss-process=image/format,png)

**官方示例：**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LWFkMWY3MGVkMDNlMTFhYjcucG5n?x-oss-process=image/format,png)

具体可跳转到官方链接详看：[条件构造器 | MyBatis-Plus](条件构造器 | MyBatis-Plus)

**根据ID查询单条数据**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LTM0NWQ4MTZlMTEyOTM0ZjMucG5n?x-oss-process=image/format,png)

**插入一条数据**

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy82NzQ3NDM1LWYyNjBlZmQ5YzdmN2Y4OGIucG5n?x-oss-process=image/format,png)

service方法中还提供了批量插入,批量更新等接口

```
// 插入一条记录（选择字段，策略插入）
boolean save(T entity);

// 插入（批量）
boolean saveBatch(Collection<T>entityList);

// 插入（批量）
boolean saveBatch(Collection<T>entityList,intbatchSize);
```

具体请跳转官方链接：[CRUD 接口 | MyBatis-Plus](https://baomidou.com/pages/49cc81/)

中文官方对接口有很详细的介绍