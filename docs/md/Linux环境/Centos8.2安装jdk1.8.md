# Centos8.2安装 JDK1.8

##### 查看是否已安装

```
java -version
```

##### 查看系统是否自带 jdk

```
rpm -qa |grep java
rpm -qa |grep jdk
rpm -qa |grep gcj
```

##### 如果有先卸载

```
rpm -qa | grep java | xargs rpm -e --nodeps
```

##### 安装

```
yum install java-1.8.0-openjdk* -y
```

##### 查看安装是否成功

```
java 
javac
```

