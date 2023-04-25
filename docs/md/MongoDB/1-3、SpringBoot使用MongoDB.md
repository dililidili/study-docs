# 1.3、SpringBoot使用MongoDB

[官网地址](https://www.mongodb.com/docs/manual/reference/operator/update/positional/)

### 1.依赖与配置

##### 1.pom依赖

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

##### 2.application.yml配置

```
spring:
  data:
    mongodb:
      host: 127.0.0.1
      port: 27017
      database: testDB
```

### 2.增删改查

##### 1.使用MongoTemplate

```
  @Autowired
  MongoTemplate mongoTemplate;
```

##### 2.新增

```
StudentPo studentPo = new StudentPo();
studentPo.setName("张三");
studentPo.setAge(18)
studentPo.setId(10)
mongoTemplate.insert(studentPo);
//保存对象到指定表
mongoTemplate.insert(studentPo,"mongodb_user");
```

##### 3.修改

```
Query query = Query.query(Criteria.where("_id").is("10"));
Update update = Update.update("age",19);
//  更新一条数据
mongoTemplate.updateFirst(query,update, StudentPo.class);
mongoTemplate.updateFirst(query,update, "mongodb_user");
mongoTemplate.updateFirst(query,update, StudentPo.class,"mongodb_user");
//  更新多条数据
mongoTemplate.updateMulti(query,update, StudentPo.class);
mongoTemplate.updateMulti(query,update,"mongodb_user");
mongoTemplate.updateMulti(query,update, StudentPo.class,"mongodb_user");
//  更新数据，如果数据不存在就新增
mongoTemplate.upsert(query,update, StudentPo.class);
mongoTemplate.upsert(query,update,"mongodb_user");
mongoTemplate.upsert(query,update, StudentPo.class,"mongodb_user");
```

##### 4.删除

```
List<StudentPo> list = new ArrayList<>();
StudentPo studentPo = new StudentPo();
studentPo.setId("12");
list.add(studentPo);

Query query = Query.query(Criteria.where("_id").in("10","12"));
//  根据条件删除
mongoTemplate.remove(query);
mongoTemplate.remove(studentPo);
mongoTemplate.remove(StudentPo.class);
//  根据条件删除（可删除多条）
mongoTemplate.remove(query,StudentPo.class,"mongodb_user");
```

##### 5.查询

```
//  查询name=张三
Query query = Query.query(Criteria.where("name").is("张三"));
//指定字段不返回
query.fields().exclude("age");
mongoTemplate.find(query,StudentPo.class);
mongoTemplate.find(query,StudentPo.class,"mongodb_user");

//模糊查询
Query query = Query.query(Criteria.where("name").regex("张"));
mongoTemplate.find(query,StudentPo.class);

//  查询所有
mongoTemplate.findAll(StudentPo.class);
mongoTemplate.findAll(StudentPo.class,"mongodb_user");

//  分页查询	page页码，pageSize每页展示几个
Pageable pageable = PageRequest.of(page - 1, pageSize, Sort.by(Sort.Order.desc("age")));
Query query = new Query().with(pageable);
return this.mongoTemplate.find(query, StudentPo.class,"mongodb_user");

//  查询多个
Query query= Query.query(Criteria.where("id").in("id1","id2","id3")).with(Sort.by(Sort.Order.desc("age")));
List<StudentPo> list= this.mongoTemplate.find(query, StudentPo.class);

//  查询数量
Criteria criteria = Criteria.where("userId").is("12345")
                .and("name").is(new ObjectId("张三"))
                .and("age").is(18);
Query query = Query.query(criteria);
long count = this.mongoTemplate.count(query, StudentPo.class);

```

