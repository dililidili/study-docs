# 1.1、Redis安装

**1.安装**

```
yum install redis
```

**2.启动**

```
systemctl start redis
```

**3.设置开机自启动**

```
systemctl enable redis
```

**4.修改配置**

打开/etc/redis.conf文件。

1）允许远程连接

找到下面这一行，注释掉：

```
bind 127.0.0.1
```

改为：

```
#bind 127.0.0.1
```

2）启用密码（文章下方有搜索说明）

找到**# requirepass foobared**一行，删除前面的#注释，然后将foobared改为你自己的密码。

```
requirepass your_password
```

找到protected-mode yes 把yes改为no

```
protected-mode no
```


开启后台启动 找到daemonize no修改为daemonize yes

```
daemonize yes
```

重启redis

```
systemctl restart redis.service
```

**6.开放端口**

查看是否开放

```
firewall-cmd --query-port=6379/tcp
```

 开放

```
firewall-cmd --zone=public --add-port=6379/tcp --permanent
```

 关闭

```
firewall-cmd --zone=public --remove-port=6379/tcp --permanent  
```

配置立即生效

```
firewall-cmd --reload 
```

查看开放端口

```
firewall-cmd --zone=public --list-ports
```

进入redis命令模式

```
cd /usr/bin
./redis-cli
进入后 
auth "password" //替换成自己密码
```

**注：关于redis安全 防止被挖矿入侵的建议**

1.注释掉47行的bind 127.0.0.1（这个意思是限制为只能 127.0.0.1 也就是本机登录）PS：个人更建议 将你需要连接Redis数据库的IP地址填写在此处，而不是注释掉。这样做会比直接注释掉更加安全。

2.更改第84行port  6379 为你需要的端口号（这是Redis的默认监听端口）PS：个人建议务必更改

3.更改第128行 daemonize no 为 daemonize yes（这是让Redis后台运行） PS:个人建议更改

4.取消第 480  # requirepass foobared 的#注释符（这是redis的访问密码） 并更改foobared为你需要的密码 比如 我需们需要密码为123456 则改为  requirepass 123456。PS：密码不可过长否则Python的redis客户端无法连接

然后重启redis

**------搜索------**

例如 搜索关键字the写法：/the     +回车

/+关键字 ，回车即可。此为从文档当前位置向下查找关键字，按n键查找关键字下一个位置；
?+关键字，回车即可。此为从文档挡圈位置向上查找关键字，按n键向上查找关键字；

[原文链接](https://blog.csdn.net/qq_34351177/article/details/120160864)