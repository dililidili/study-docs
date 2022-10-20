# Centos 8.2安装 MySQL5.7

#### 一、检查环境

##### 1.查看系统版本

```
cat /etc/redhat-release
```

##### 2.查看是否安装过MySQL

```
systemctl status mysqld.service
```

#### 二、下载并安装

##### 1.下载MySQL官方的Yum Repository

```
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

##### 2.安装Yum Repository

```
yum -y install mysql57-community-release-el7-10.noarch.rpm
```

##### 3.关闭mysql模块

```
yum module disable mysql
```

##### 4.安装

```
yum -y install mysql-community-server
```

##### 5.卸载Yum Repository

```
yum -y remove mysql57-community-release-el7-10.noarch
```

#### 三、启动

##### 1.查看mysql服务状态

```
systemctl status mysqld.service
```

##### 2.启动mysql服务

```
systemctl start mysqld.service
```

#### 四、mysql内部配置

##### 1.获取root管理员账户密码

```
grep "password" /var/log/mysqld.log
```

##### 2.登陆

```
mysql -uroot -p
```

##### 录入密码

##### 3.修改root账户默认密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MySQL%57';
```

##### 密码必须包含大写字母和数字和字符  否则修改失败

##### 4.开启远程访问

##### 第一种方式：授权

```
grant all privileges on *.* to 'root'@'%' identified by 'MySQL%57' with grant option;
```

##### 第二种：改表;

```
#进入mysql数据库
use mysql;
#改表
update user set host="%" where user ="root";
```

如果需要指定ip访问  用IP地址替换%即可

##### 5.执行并刷新

```
flush privileges;
```

##### 6.退出mysql

```
exit;
```

#### 五、外部配置

##### 1.添加端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

##### 2.端口配置生效

```
firewall-cmd --reload
```

##### 3.修改数据库字符编码

```
vim /etc/my.cnf
```

##### 添加下面4行

```
[client]
default-character-set=utf8

[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci
```

##### 5.保存退出 按esc

```
:wq!
```

##### 6.重启mysql

```
systemctl restart mysqld.service
```

####  六、成功

#### 错误

##### 1.mysql-community-client-5.7.39-1.el7.x86_64.rpm 的公钥尚未安装

原因:

可能是MySQL GPG 密钥已过期导致，改一下密钥。

解决:

执行一下命令解决

```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

