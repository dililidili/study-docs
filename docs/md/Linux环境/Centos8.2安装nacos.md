# Centos8.2 安装 NACOS

#### 一、下载

1.地址：[nacos下载地址](https://github.com/alibaba/nacos/releases)

2.选中文件下载

##### 二、安装

 1.下载完成后传输到centos8下并解压

2.chmod 777 -R * 授权

##### 三、配置

##### 1.创建配置文件 配置开机自启

```
vim /lib/systemd/system/nacos.service
```

内容：

```
[Unit]
Description=nacos
After=network.target

[Service]
Type=forking
ExecStart=/opt/nacos/bin/startup.sh -m standalone
ExecReload=/opt/nacos/bin/shutdown.sh
ExecStop=/opt/nacos/bin/shutdown.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

其中/opt/nacos为本机安装的nacos文件路径，-m standalone表示作为单机启动，不加的话表示集群启动，目前先作为单机启动。

##### 2.配置nacos数据库配置

**修改application.properties配置文件**

```
cd /opt/nacos/config
vim application.properties
```

![img](https://pic1.zhimg.com/80/v2-689cbb36d05cf31d2bf40ff283f0302c_1440w.webp)

##### 3.在数据库中创建`nacos`数据库并执行config目录下nacos-mysql.sql 初始化数据

##### 4.在config目录下创建`cluster.conf`空文件(不创建会报jmenv.tbsite.net错误)

##### 5.设置开机自启

systemctl daemon-reload        #先进行文件生效配置
systemctl enable nacos.service #设置为开机启动
systemctl start nacos.service  #启动nacos服务
systemctl stop nacos.service

##### 6.启动并查看是否成功

启动成功后进行查看：systemctl status nacos.service

开放端口：firewall-cmd --zone=public --add-port=8848/tcp --permanent

立即生效：firewall-cmd --reload

##### 7.输入ip:8848/nacos进入登录页。默认账号密码为nacos/nacos。