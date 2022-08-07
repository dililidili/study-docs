### 2.6、使用JDBC实现多数据源

#### 1.配置

##### pom依赖配置

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```

##### application.properties配置文件

```
#第一个数据源指向test数据库
spring.one.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&useSSL=false&serverTimezone=Asia/Shanghai
spring.one.datasource.username=root
spring.one.datasource.password=123456
 
 #第二个数据源指向了test2数据库
spring.two.datasource.url=jdbc:mysql://127.0.0.1:3306/test2?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&useSSL=false&serverTimezone=Asia/Shanghai
spring.two.datasource.username=root
spring.two.datasource.password=123456
```

##### 启动类SpringBootApplication.class 

```
//移除DataSourceAutoConfiguration、DataSourceTransactionManagerAutoConfiguration、JdbcTemplateAutoConfiguration
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class,
		DataSourceTransactionManagerAutoConfiguration.class,
		JdbcTemplateAutoConfiguration.class})
public class SpringBootApplication {
 
	public static void main(String[] args) {
		SpringApplication.run(SpringBootApplication.class, args);
	}

}
```

##### 配置不同数据源配置类

第一个数据源配置类

```
 
@Configuration
@Slf4j
public class OneDataSourceConfig {
 
    @Bean
    @ConfigurationProperties("spring.one.datasource")
    public DataSourceProperties oneDataSourceProperties(){
        return new DataSourceProperties();
    }
 
    @Bean
    public DataSource oneDateSource(){
        DataSourceProperties oneDataSourceProperties = oneDataSourceProperties();
        log.info("oneDatasource:"+oneDataSourceProperties.getUrl());
        return oneDataSourceProperties.initializeDataSourceBuilder().build();
    }
 
    @Bean
    public JdbcTemplate oneJdbcTemplate(){
        return new JdbcTemplate(oneDateSource());
    }
 
    @Bean
    @Resource
    public PlatformTransactionManager onePlatformTransactionManager(){
        return new DataSourceTransactionManager(oneDateSource());
    }
 
}
```

第二个数据源配置类

```
 
@Configuration
@Slf4j
public class TwoDataSourceConfig {
 
    @Bean
    @ConfigurationProperties("spring.two.datasource")
    public DataSourceProperties twoDataSourceProperties(){
        return new DataSourceProperties();
    }
 
    @Bean
    public DataSource twoDateSource(){
        DataSourceProperties twoDataSourceProperties = twoDataSourceProperties();
        log.info("twoDatasource:"+twoDataSourceProperties.getUrl());
        return twoDataSourceProperties.initializeDataSourceBuilder().build();
    }
 
    @Bean
    public JdbcTemplate twoJdbcTemplate(){
        return new JdbcTemplate(twoDateSource());
    }
 
 
    @Bean
    @Resource
    public PlatformTransactionManager twoPlatformTransactionManager(){
        return new DataSourceTransactionManager(twoDateSource());
    }
}
```

#### 2.使用

```
  @Resource(name = "oneJdbcTemplate")
    JdbcTemplate jdbcTemplate;
    @Resource(name = "twoJdbcTemplate")
    JdbcTemplate jdbcTemplate2;
 
    @GetMapping("test")
    public void test(){
        Integer fpck = jdbcTemplate.queryForObject("select count(1) from tb_stockout where order_type = ? ", Integer.class, "FPCK");
        Integer fpck1 = jdbcTemplate2.queryForObject("select count(1) from tb_stockout where order_type = ? ", Integer.class, "FPCK");
 
        System.out.println(fpck);
        System.out.println(fpck1);
    }
```

