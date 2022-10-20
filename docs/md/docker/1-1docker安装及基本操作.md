# 1、docker安装及基本操作

## 1.安装

linux环境centos8.2

卸载旧版本

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

安装包

```
yum install -y yum-utils
```

设置阿里镜像

```
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装

```
yum install docker-ce docker-ce-cli containerd.io
```

安装完成测试

```
docker version
```

查看当前docker存储驱动类型

```
 docker system info
```

(非必要)docker配置存储驱动为Device Mapper 并使用direct-lvm模式

```
{
        "storage-driver": "devicemapper",
        "storage-opts": [
        # 设置块设备存储位置
        "dm.directlvm_device=/dev/xdf",
        # 设置镜像和存储允许使用的最大存储空间占比，默认95%
        "dm.thinp_percent=95",
        # 设置元数据存储允许使用的最大存储空间占比，默认1%
        "dm.thinp_metapercent=1",
        # 设置LVM自动扩展精简池阈值，默认80%
        "dm.thinp_autoextend_threshold=80",
        # 表示当触发精简池自动扩容机制时，扩容大小占现有空间比例
        "dm.thinp_autoextend_percent=20",
        # 是否允许用户决定是否将块设备格式化为新的文件系统
        "dm.thinp_autoextend_force=false",
        ]
}

```

启动docker服务

```
service start docker
```

## 2.基本操作

查看所有镜像

```
 docker image ls
```

拉取镜像

```
 docker pull centos
```

创建容器

```
 #第一种
 docker container run -it centos:latest /bin/bash
 #第二种自定义名字
 docker container run --name zsj -it centos:latest /bin/bash
 #第三种 绑定端口
 docker container run -p 9002:8080 centos:latest /bin/bash
 #第四种 后台运行
 docker container run -p 9002:8080 centos:latest /bin/bash -d
```

退出容器 但不会杀死进程

```
ctrl+q+p
```

查看所有容器状态

```
docker container ls
```

下面相关操作(04a7a39148a6 替换成自己的镜像id)

进入容器

```
 docker container exec -it 04a7a39148a6 bash
```

停止容器

```
 docker container stop 04a7a39148a6
```

查看所有容器 包含停止的

```
docker container ls -a
```

杀死容器

```
 docker container rm 04a7a39148a6
```

查询镜像 centos为你需要的镜像名

```
docker search centos
```

查询官方镜像

```
docker search centos --filter "is-official=true"
```

查询所有镜像

[[Docker Hub 中央仓库]](https://hub.docker.com/)

删除镜像(5d0da3dc9764是镜像id 通过`docker image ls`查看)

```
docker image rm 5d0da3dc9764
```

强制删除全部容器

```
docker container rm $(docker container ls -aq) -f
```

