# 1、dockerfile

**dockerfile是一个包含用于组合镜像的命令的文本文档**

**docker build -t 镜像名:<tags> dcokerfile目录**

## 1.1、根据当前环境生成一个镜像

**1.创建Dockerfile文件** 

文件内容：

```
#来源镜像
FORM tomcat:latest
#机构
MAINTAINER: CC.COM
#切换目录，不存在则创建
WORKDIR /usr/local/tomcat/webapps
#复制目录下文件到容器目录不存在则创建 复制和Dockerfile文件夹同级目录下的source-file文件夹 到 /usr/local/tomcat/webapps下
ADD source-file ./java-web
```

**2.文件传输到linux服务器上**

**3.cd到存放文件的目录上(Dockerfile文件目录)**

**4.执行dockerfile文件创建镜像**(最后的.代表当前目录 当前在容器下)

```
docker build -t cc.com/zsjtest:1.0 .
```

**5.命令列表**

```
#Label说明信息
LABEL name="测试"
#ENV设置环境常量
ENV JAVA_HOME /usr/local/jdk1.8
#使用环境常量
RUN ${JAVA_HOME}/bin/java -jar test.jar
# run在构建镜像时执行，cmd/enterpoint在容器创建时执行
#两种书写格式 比如安装vim
RUN yum install -y vim
RUN["yum","install","-y","vim"]

```

