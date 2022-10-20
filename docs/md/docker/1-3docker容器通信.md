# 1、docker容器通信

## 1.1、容器单向通信

**1.比如tomcat容器需要访问mysql容器，而mysql不需要访问tomcat**

获取容器的元数据

```
docker inspect 5d0da3dc9764
```

可以直接找到`NetworkSettings.IPAddress`属性`虚拟ip` 容器质检通过虚拟ip是可以ping通的

还有一种方式

```
# link指定链接名称
docker container run -it centos:latest /bin/bash --link testlink
#可以使用 ping testlink ping通该容器
```

## 1.2、双向通信

```
#查看网桥
docker network ls
#第一步 配置一个网桥
docker network create -d bridge mybridge
#第二步 容器绑定网桥
docker network connect mybridge testlink
docker network connect mybridge testjava
```

## 1.3、容器间的数据共享

**1.挂载**

```
docker run --name test2 -p9002:8080 -d -v /usr/webapps:/usr/local/tomcat/webapps tomcat:latest
```

**1.使用--volumes -from 共享容器内挂载点**

```
#创建共享容器
docker create --name webpage -v /root/zsj/webapps:/tomcat/webapps tomcat:latest /bin/true
```

**2.使用共享**

```
docker run -p9002:8080 --volumes-from webpage --name test3 -d tomcat:latest
```

