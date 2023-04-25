# 1.4、MongoDB复制集原理和配置

### 1、MongoDB复制集原理

#### 1.1、MongoDB复制集的实现原理

数据写入时迅速将数据复制到另一个独立节点上，接受写入的节点发生故障时自动选举出一个新的替代节点。
#### 1.2、复制集结构

一个复制集由3个以上有投票权的节点组成，一个主节点（primary）接受写入操作和选举时投票，两个或多个从节点（secondary）复制主节点上的新数据和选举时投票。
#### 1.3、数据复制过程

当一个修改操作到达主节点时，它对数据的操作会被记录下来（经过一些必要的转换），这些记录成为oplog。
从节点通过在主节点上打开一个tailable游标不断获取进入主节点的oplog，在自己的数据上回访，从而保持和主节点的数据一致。
#### 1.4、通过选举完成故障恢复的过程

- 具有投票权的节点之间两两互发心跳；
- 当5次未收到心跳时判断为节点失联；
- 若为主结点失效，从节点会发起选举，选出新的主节点；
- 若为从节点失效，不发起选举；
- 选举是基于RAFT一致性算法实现的，大多数投票节点存活时才会发起选举；
- 复制集中最多可以有50个节点，但有投票权的节点最多只有7个；
- 被选举为主节点的条件：能与多数节点建立连接，具有较新的oplog，具有较高优先级。

#### 1.5、复制集节点常见配置选项

- 投票权（v参数）：有此参数即可参与投票；
- 优先级（priority参数）：优先级越高越优先成为主节点，优先级为0的节点不能成为主节点。
- 隐藏（hidden参数）：隐藏节点优先级为0，用于复制数据，对应用不可见，但可以具有投票权。
- 延迟（slaveRelay参数）：复制n秒之前的数据，保持与主节点的时间差。用于防止误操作。

#### 1.6、复制集注意事项

**硬件**：主从节点使用一样的硬件配置。各节点使用的硬件独立。

**软件**：各节点软件版本要一致。

**性能不够时，不要盲目增加节点。增加从节点不会增加系统写性能，但是可以增加系统读性能。**

### 2.MongoDB复制集配置 主从配置

#### 2.1、创建数据目录

##### 创建文件

```
mkdir -p /data/db{1,2,3}
```

##### 配置文件 每个目录下都放一个(需要把端口号和目录改成对应的)

```
systemLog:
  destination: file
  path: /data/db1/mongod.log ## 日志
  logAppend: true
storage:
  dbPath: /data/db1 ## 数据目录
net:
  bindIp: 0.0.0.0 ## 本机所有网卡连接都提供对外服务
  port: 28017 ## 端口号
replication:
  replSetName: rs0  ## 复制集的名字为 rs0
processManagement:
  fork: true ## 进程在后台运行
```

##### 启动

```
mongod -f /data/db1/mongod.conf
```

#### 2.2、配置

```
rs.initiate({
  _id: "rs0",
  members: [
      {
        _id: 0,
        host: "localhost:28017" 
      },
      {
        _id: 1,
        host: "localhost:28018" 
      },
      {
        _id: 2,
        host: "localhost:28019" 
      }
  ]
})
```

#### 2.3、测试

##### 20817

```
rs0:PRIMARY> db.test.findOne()
null
rs0:PRIMARY> db.test.insert({a:1})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.test.findOne()
{ "_id" : ObjectId("5e03695cbed9486c92da1acd"), "a" : 1 }
rs0:PRIMARY> db.test.insert({a:2})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.test.find()
{ "_id" : ObjectId("5e03695cbed9486c92da1acd"), "a" : 1 }
{ "_id" : ObjectId("5e03696cbed9486c92da1ace"), "a" : 2 }
```

##### 20818

```
rs0:SECONDARY> rs.slaveOk() ## 从结点可以读数据
rs0:SECONDARY> db.test.find() ## 否则这条报错
rs0:SECONDARY> db.test.find()
{ "_id" : ObjectId("5e03695cbed9486c92da1acd"), "a" : 1 }
rs0:SECONDARY> db.test.find()
{ "_id" : ObjectId("5e03695cbed9486c92da1acd"), "a" : 1 }
{ "_id" : ObjectId("5e03696cbed9486c92da1ace"), "a" : 2 }
```

