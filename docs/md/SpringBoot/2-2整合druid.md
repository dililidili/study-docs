# 2.2整合druid连接池

[druid中文官网](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)

### **pom依赖**

```
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.8</version>
</dependency>
```

### **application.yml**配置文件 只需要添加`type`,druid还带有一些其他属性

```
spring:
  datasource:
  	driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/test?allowMultiQueries=true&useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
```

### druid独有配置

```
druid:
  # 下面为连接池的补充设置，应用到上面所有数据源中
  # 初始化大小，最小，最大
  initial-size: 5
  min-idle: 5
  max-active: 20
  # 配置获取连接等待超时的时间
  max-wait: 60000
  # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
  time-between-eviction-runs-millis: 60000
  # 配置一个连接在池中最小生存的时间，单位是毫秒
  min-evictable-idle-time-millis: 300000
  validation-query: SELECT 1 FROM DUAL
  test-while-idle: true
  test-on-borrow: false
  test-on-return: false
  # 打开PSCache，并且指定每个连接上PSCache的大小
  pool-prepared-statements: true
  #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙 ; ,wall加上无法执行批量处理
  max-pool-prepared-statement-per-connection-size: 20
  filters: stat
  use-global-data-source-stat: true
  # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
  connect-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
  # 配置监控服务器
  stat-view-servlet:
  login-username: admin
  login-password: 123456
  reset-enable: false
  url-pattern: /druid/*
  # 添加IP白名单
  #allow:
  # 添加IP黑名单，当白名单和黑名单重复时，黑名单优先级更高
  #deny:
  web-stat-filter:
  # 添加过滤规则
  url-pattern: /*
  # 忽略过滤格式
  exclusions: "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*"
```

### 官网配置表格

| 配置                                      | 缺省值             | 说明                                                         |
| ----------------------------------------- | ------------------ | ------------------------------------------------------------ |
| name                                      |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this). 另外配置此属性至少在1.0.5版本中是不起作用的，强行设置name会出错。[详情-点此处](http://blog.csdn.net/lanmo555/article/details/41248763)。 |
| url                                       |                    | 连接数据库的url，不同数据库不一样。例如： mysql : jdbc:mysql://10.20.153.104:3306/druid2  oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                                  |                    | 连接数据库的用户名                                           |
| password                                  |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。[详细看这里](https://github.com/alibaba/druid/wiki/使用ConfigFilter) |
| driverClassName                           | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName |
| initialSize                               | 0                  | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                                 | 8                  | 最大连接池数量                                               |
| maxIdle                                   | 8                  | 已经不再使用，配置了也没效果                                 |
| minIdle                                   |                    | 最小连接池数量                                               |
| maxWait                                   |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements                    | false              | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxPoolPreparedStatementPerConnectionSize | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery                           |                    | 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。 |
| validationQueryTimeout                    |                    | 单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法 |
| testOnBorrow                              | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                              | false              | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testWhileIdle                             | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| keepAlive                                 | false （1.0.28）   | 连接池中的minIdle数量以内的连接，空闲时间超过minEvictableIdleTimeMillis，则会执行keepAlive操作。 |
| timeBetweenEvictionRunsMillis             | 1分钟（1.0.14）    | 有两个含义： 1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun                    | 30分钟（1.0.14）   | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis                |                    | 连接保持空闲而不被驱逐的最小时间                             |
| connectionInitSqls                        |                    | 物理连接初始化的时候执行的sql                                |
| exceptionSorter                           | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                                   |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat 日志用的filter:log4j 防御sql注入的filter:wall |
| proxyFilters                              |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |

**官网地址:** [DruidDataSource配置属性列表](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)

### JAVA代码载入druild连接池配置

```
package com.example.demo;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import com.alibaba.druid.wall.WallConfig;
import com.alibaba.druid.wall.WallFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Servlet;
import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;


@Configuration
public class DruidConfig {
    /**
     * 将自定义的 Druid数据源添加到容器		 中，不再让 Spring Boot 自动创建
     * 绑定全局配置文件中的 druid 数据源属性到  com.alibaba.druid.pool.DruidDataSource从而让它们生效
     *
     * @ConfigurationProperties(prefix = "spring.datasource.")：作用就是将 全局配置文件中
     * 前缀为 spring.datasource的属性值注入到 com.alibaba.druid.pool.DruidDataSource 的同名参数中
     */
    @ConfigurationProperties(prefix = "spring.datasource.druid")
    @Bean
    public DataSource druidSource() {
        return new DruidDataSource();
    }
		/**
			* Servlet 需要javax.servlet包
			* 多重配置方式  比如还可以用application.yml 里面的stat-view-servlet 配置
			*/
    @Bean
    public ServletRegistrationBean druidServletRegistrationBean() {
        ServletRegistrationBean<Servlet> servletRegistrationBean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        //表示谁可以访问druid监控页面 为空表示所有人都可以访问
        initParams.put("allow", "127.0.0.1");
        //表示禁止访问druid监控页面的ip 即黑名单
        initParams.put("msb", "192.168.10.1");
        servletRegistrationBean.setInitParameters(initParams);
        return servletRegistrationBean;
    }


    /**
     * 配買 Druid 监控之web监控的 filter
     * webstatFilter：用于配置web和Druid数据源之间的管理关联监控统计
     */
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        //设買哪些请求进行过滤排除掉，从而不进行统计
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*. js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        //"/*"表示过滤所有请求
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }


    @Bean
    public WallFilter wallFilter() {
        WallFilter wallFilter = new WallFilter();
        wallFilter.setConfig(wallConfig());
        return wallFilter;
    }

    @Bean
    public WallConfig wallConfig() {
        WallConfig wallConfig = new WallConfig();
        //允许一次执行多条语句
        wallConfig.setMultiStatementAllow(true);
        //允许⾮基本语句的其他语句
        wallConfig.setNoneBaseStatementAllow(true);
        return wallConfig;
    }
}

```

