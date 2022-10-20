# 1.docker-compose单机多部署工具

## 1.1、安装使用

**1.docker-compose安装**

[docker-compose官方文档](https://docs.docker.com/compose/install/linux/)

第一种方式

```
yum update
yum install docker-compose-plugin
```

第二种方式

```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.11.2/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

查看安装是否成功

```
docker compose version
```

**2.创建配置**

```
cd /usr/
mkdir workpress
cd workpress
vim docker-compose.yml 
```

[配置文件官方文档](https://docs.docker.com/samples/wordpress/)

复制其内容并保存

```
services:
  db:
  
    image: mariadb:10.6.4-focal

    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
  wp_data:
```

**执行**

```
docker compose up -d
```

执行完后会自动创建2个容器

 Container workpress-wordpress-1、Container workpress-db-1

然后查看所有容器

```
docker container ls 
#查看workpress-wordpress-1 映射端口
#然后用浏览器直接访问进行初始化配置
```

## 1.2、项目容器

**1.创建项目镜像**

```
#进入项目目录
cd /usr/image/testproject/java-test
# 目录下有java-test.jar 和配置文件
#创建docker脚本
vim Dockerfile
```

Dockerfile文件内容

```
#使用jdk基准镜像
FROM openjdk:8u222-jre
#创建文件夹
WORKDIR /usr/local/java-test
#复制文件
ADD java-test.jar
ADD application.yml
ADD application-dev.yml
#设置80端口
EXPOSE 80
#启动jar包
CMD ["java","-jar","java-test.jar"]
```

创建镜像

```
docker build -t dililidili/java-test
```

**2.创建db镜像**

```
#进入目录
cd /usr/image/testproject/java-test-db
#目录下有java-test-db.sql
#创建docker脚本
vim Dockerfile
```

Dockerfile文件内容

```
FROM mysql:5.7
WORKDIR /docker-entrypoint-initdb.d
ADD java-test-db.sql
```

[docker-mysql官方文档](https://hub.docker.com/_/mysql)

创建镜像

```
docker build -t dililidili/java-test-db
```

mysql创建容器命令可以添加环境变量

```
#添加环境变量 MYSQL_ROOT_PASSWORD:root账户默认密码 
#其它环境变量 上方给出的官方文档中都有展示
-e MYSQL_ROOT_PASSWORD=my-secret-pw
```

**3.整合项目镜像和db镜像**

```
#进入项目目录
cd /usr/image/testproject
#创建docker-compose.yml
vim docker-compose.yml
```

docker-compose.yml文件内容

```
#docker-compose版本
version: '3.3'
#描述部署容器相关信息
services:
	#服务名/主机名
  db: 
    #包含Dockerfile文件目录
  	build: ./java-test-db/
  	#只要出现容器当机 总是会重启
    restart: always
    #环境变量
    environment:
    	MYSQL_ROOT_PASSWORD: my-secret-pw
  app:
  	build: ./java-test/
  	#设置依赖 依赖上面的db
  	depends_on:
  		- db
  	#端口映射
    ports:
    	- "9003:80"
    restart: always
```

修改application-dev.yml 数据库配置

**该用主机名连接 db:3306**

启动

```
#前台
docker-compose up
#后台
docker-compose up -d 
#查看后台启动时日志
docker-compose logs 主机名
#关闭
docker-compose down
```

