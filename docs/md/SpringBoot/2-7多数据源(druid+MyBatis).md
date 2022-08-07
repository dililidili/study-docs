### 2.7使用druid+MyBatis配置多数据源

#### 1.配置

##### pom依赖

```
		<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.6</version>
        </dependency>
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- alibaba的druid数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.20</version>
        </dependency>

```

##### **application.yml** 配置

```
spring:
  datasource:
    test1: 
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/cherish?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
      username: root
      password: root
     test2:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/ceshi?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
      username: root
      password: root
```

##### 创建test1数据源配置类

```
import com.alibaba.druid.pool.DruidDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;
//com.abc.test1.dao包下使用该数据源
@Configuration
@MapperScan(basePackages = "com.abc.test1.dao", sqlSessionFactoryRef = "oneSqlSessionFactory")
public class DateSourceConfig {
    @Value("${spring.datasource.test1.driver-class-name}")
    private String driverClassName;

    @Value("${spring.datasource.test1.url}")
    private String url;

    @Value("${spring.datasource.test1.username}")
    private String username;

    @Value("${spring.datasource.test1.password}")
    private String password;

    @Bean(name = "oneDataSource")
    @Primary
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(this.driverClassName);
        dataSource.setUrl(this.url);
        dataSource.setUsername(this.username);
        dataSource.setPassword(this.password);
        return dataSource;
    }

    @Bean(name = "oneSqlSessionFactory")
    @Primary
    public SqlSessionFactory sqlSessionFactory(@Qualifier("oneDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(
                new PathMatchingResourcePatternResolver().getResources("classpath*:/mapper/cherish/*Mapper.xml"));
        return bean.getObject();
    }

    @Bean(name = "oneTransactionManager")
    @Primary
    public DataSourceTransactionManager transactionManager(@Qualifier("oneDataSource") DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
    @Bean(name = "oneSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("oneSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```

##### 创建test2数据源配置类

```
import com.alibaba.druid.pool.DruidDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

//com.abc.test2.dao包下使用该数据源
@Configuration
@MapperScan(basePackages = "com.abc.test2.dao", sqlSessionFactoryRef = "twoSqlSessionFactory")
public class SubDateSourceConfig {
    @Value("${spring.datasource.test2.driver-class-name}")
    private String driverClassName;

    @Value("${spring.datasource.test2.url}")
    private String url;

    @Value("${spring.datasource.test2.username}")
    private String username;

    @Value("${spring.datasource.test2.password}")
    private String password;

    @Bean(name = "twoDataSource")
    @Primary
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(this.driverClassName);
        dataSource.setUrl(this.url);
        dataSource.setUsername(this.username);
        dataSource.setPassword(this.password);
        return dataSource;
    }

    @Bean(name = "twoSqlSessionFactory")
    @Primary
    public SqlSessionFactory sqlSessionFactory(@Qualifier("twoDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(
                new PathMatchingResourcePatternResolver().getResources("classpath*:/mapper/ceshi/*Mapper.xml"));
        return bean.getObject();
    }

    @Bean(name = "twoTransactionManager")
    @Primary
    public DataSourceTransactionManager transactionManager(@Qualifier("twoDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "twoSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("twoSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```

**注意点：**
1.@Primary：优先方案，被注解的实现，优先被注入。
2.@Qualifier：先声明后使用，相当于多个实现起多个不同的名字，注入时候告诉我你要注入哪个。
3.该配置是对某个包下的dao层和mapper.xml文件绑定数据源。
到此处MyBatis多数据源配置就算结束了，只需要在对应包下dao层和mapper编写就可以访问对应的数据源了。



> 推荐使用MyBatisPlus做多数据源 配置简单、使用方便

参考文献:
[Spring Boot2.X MyBatis 多数据源配置](https://www.cnblogs.com/zwqh/p/11660645.html)
