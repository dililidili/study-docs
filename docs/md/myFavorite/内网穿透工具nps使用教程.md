[官网链接](https://github.com/cnlh/nps/)

## 一、使用条件

1.需要有一个外网ip的服务器。

## 二、下载地址

[Releases · ehang-io/nps (github.com)](https://github.com/ehang-io/nps/releases)

分为服务端和客户端 有外网ip的使用服务端

## 三、中文文档

[[nps/README_zh.md at master · ehang-io/nps (github.com)](https://github.com/ehang-io/nps/blob/master/README_zh.md)](https://github.com/ehang-io/nps/blob/master/README_zh.md)

## 四、配置

```
vim conf/nps.conf

web_host= 服务器IP或者域名
web_username= admin（登录用户名）
web_password= 你的密码
web_port=8080（web管理端口）
```

**nps操作**

```
./nps test|start|stop|restart|status 测试配置文件|启动|停止|重启|状态
```

**访问地址: ip:8080**

![image-20221007162011029](/Users/zhangshaojie/WebstormProjects/study-docs/docs/img/ELasticsearch/image-20221007162011029.png)

![image-20221007162151187](/Users/zhangshaojie/WebstormProjects/study-docs/docs/img/ELasticsearch/image-20221007162151187.png)

## 五、服务端操作

**启动**

```
./npc -server=你的IP:8024 -vkey=唯一验证密码 -type=tcp
```

