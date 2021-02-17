 ## 配置文件

​	在之前的项目开发中，我们可以使用xml,properties进行相关的配置，这种配置方式比较简单，但是在应对复杂的商业需求下，多环境和编程化的配置无法得到满足，因此springboot为我们提供了YAML的配置方式丰富功能和简化开发。

### properties

```properties
person.name=zhangsan
person.age=12
person.gender=男
person.desc=my name is ${person.name},my age is ${person.age}
```

**读取**

```java
@Component
@Data
public class Person {
    @Value("${person.age}")
    private Integer age;
    @Value("${person.name}")
    private String name;
    @Value("${person.gender}")
    private String gender;
    @Value("${person.desc}")
    private  String desc;

}
```

如果这里有很多属性的话，那么这样写太麻烦了

### 多环境配置

```properties
application-dev.properties
application-test.properties
application-prod.properties
application.properties
spring.profiles.active=dev/test/prod
```



### yaml

基本原则：
1、大小写敏感 
2、使用缩进表示层级关系 
3、禁止使用tab缩进，只能使用空格键 
4、缩进长度没有限制，只要元素对齐就表示这些元素属于一个层级。 
5、使用#表示注释 
6、字符串可以不用引号标注

### 配置读取

```yaml
person:
  age: 10
  name: zs
  gender : m
  desc : desc
```

```java
@Component
@Data
@ConfigurationProperties(prefix = "person")
public class Person {

    private Integer age;
    private String name;
    private String gender;
    private  String desc;

}
```

### 多环境配置

```yaml
spring:
  profiles:
    active: test
---
spring:
  profiles: dev
server:
  port: 8080
---
spring:
  profiles: test
server:
  port: 8081
```

## 对比

### 

| 功能       | @ConfigurationProperties | @Value |
| ---------- | ------------------------ | ------ |
| 松散绑定   | 是                       | 否     |
| 元数据支持 | 是                       | 否     |
| spEL表达式 | 否                       | 是     |

## 加载静态资源

​	当查找静态资源的时候能够发现静态资源的路径是/**，会去ResourceProperties这个类，可以看到对应的资源目录。

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
	private String[] appendSlashIfNecessary(String[] staticLocations) {
		String[] normalized = new String[staticLocations.length];
		for (int i = 0; i < staticLocations.length; i++) {
			String location = staticLocations[i];
			normalized[i] = location.endsWith("/") ? location : location + "/";
		}
		return normalized;
	}
```

可以看到静态资源的目录一共有如下几个：
		"classpath:/resources/"	
		"classpath:/static/"	
		"classpath:/public/" 

### springmvc的扩展

​		springmvc框架是一个mvc的web框架，springmvc允许创建@controller和@RestController bean来处理传入的HTTP请求，控制器种的方法通过@RequestMapping注解映射到HTTP。

​		Springboot提供了适用于大多数Springmvc应用的自动配置。

```tex
引入 ContentNegotiatingViewResolver 和 BeanNameViewResolver bean视图解析器。
支持服务静态资源，包括对 WebJar 的支持。
自动注册 Converter（网页传入的数据封装成对象，完成数据类型的转化）、GenericConverter 和 Formatter bean（将日期转换成规定的格式）。
支持 HttpMessageConverter，用来转换http请求和响应。
自动注册 MessageCodesResolver，定义错误代码生成规则。
支持静态 index.html。
支持自定义 Favicon。
自动使用 ConfigurableWebBindingInitializer bean，将请求树绑定到javaBean中。
```

自动配置了ViewResolver，就是我们之前学习的SpringMVC的视图解析器：即根据方法的返回值取得视图对象（View），然后由视图对象决定如何渲染（转发，重定向）

## springboot配置数据源

​		Spring Framework 为 SQL 数据库提供了广泛的支持。从直接使用 JdbcTemplate 进行 JDBC 访问到完全的对象关系映射（object relational mapping）技术，比如 Hibernate。Spring Data 提供了更多级别的功能，直接从接口创建的 Repository 实现，并使用了约定从方法名生成查询。

### 1、JDBC

1、创建项目，导入需要的依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
 		<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

2、配置数据源

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.85.111:3306/sakila?serverTimezone=UTC&useUnicode=true@characterEncoding=utf-8
    driver-class-name: com.mysql.jdbc.Driver
```

### 2、自定义数据源DruidDataSource

通过源码查看DataSourceAutoConfiguration.java

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })
public class DataSourceAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	@Conditional(EmbeddedDatabaseCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import(EmbeddedDataSourceConfiguration.class)
	protected static class EmbeddedDatabaseConfiguration {

	}

	@Configuration(proxyBeanMethods = false)
	@Conditional(PooledDataSourceCondition.class)
	@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
	@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
			DataSourceJmxConfiguration.class })
	protected static class PooledDataSourceConfiguration {

	}

	/**
	 * {@link AnyNestedCondition} that checks that either {@code spring.datasource.type}
	 * is set or {@link PooledDataSourceAvailableCondition} applies.
	 */
	static class PooledDataSourceCondition extends AnyNestedCondition {

		PooledDataSourceCondition() {
			super(ConfigurationPhase.PARSE_CONFIGURATION);
		}

		@ConditionalOnProperty(prefix = "spring.datasource", name = "type")
		static class ExplicitType {

		}

		@Conditional(PooledDataSourceAvailableCondition.class)
		static class PooledDataSourceAvailable {

		}

	}
```

1、添加druid的maven配置

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.12</version>
</dependency>
```

2、添加数据源的配置

```yaml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.85.111:3306/demo?serverTimezone=UTC&useUnicode=true@characterEncoding=utf-8
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
```

```properties
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.85.111:3306/demo?serverTimezone=UTC&useUnicode=true@characterEncoding=utf-8
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true

    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址： https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

### 配置参数需要配置类

```java
package com.mashibing.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DruidConfig {
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }
}

```

## 多数据源

​	创建一个数据源切换处理类，有对数据源变量的获取、设置和情况的方法，其中threadlocal用于保存某个线程共享变量。

```java
package com.mashibing.mult;

public class DynamicDataSourceContextHolder {

    /**
     * 使用ThreadLocal维护变量，ThreadLocal为每个使用该变量的线程提供独立的变量副本，
     *  所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。
     */
    private static final ThreadLocal<String> CONTEXT_HOLDER = new ThreadLocal<>();

    /**
     * 设置数据源变量
     * @param dataSourceType
     */
    public static void setDataSourceType(String dataSourceType){
        System.out.printf("切换到{%s}数据源", dataSourceType);
        CONTEXT_HOLDER.set(dataSourceType);
    }

    /**
     * 获取数据源变量
     * @return
     */
    public static String getDataSourceType(){
        return CONTEXT_HOLDER.get();
    }

    /**
     * 清空数据源变量
     */
    public static void clearDataSourceType(){
        CONTEXT_HOLDER.remove();
    }
}
```

4、继承AbstractRoutingDataSource

​		动态切换数据源主要依靠AbstractRoutingDataSource。创建一个AbstractRoutingDataSource的子类，重写determineCurrentLookupKey方法，用于决定使用哪一个数据源。这里主要用到AbstractRoutingDataSource的两个属性defaultTargetDataSource和targetDataSources。defaultTargetDataSource默认目标数据源，targetDataSources（map类型）存放用来切换的数据源。

```java
package com.mashibing.mult;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import javax.sql.DataSource;
import java.util.Map;

public class DynamicDataSource extends AbstractRoutingDataSource {

    public DynamicDataSource(DataSource defaultTargetDataSource, Map<Object, Object> targetDataSources) {
        super.setDefaultTargetDataSource(defaultTargetDataSource);
        super.setTargetDataSources(targetDataSources);
        // afterPropertiesSet()方法调用时用来将targetDataSources的属性写入resolvedDataSources中的
        super.afterPropertiesSet();
    }

    /**
     * 根据Key获取数据源的信息
     *
     * @return
     */
    @Override
    protected Object determineCurrentLookupKey() {
        return DynamicDataSourceContextHolder.getDataSourceType();
    }
}
```

5、注入数据源

```java
package com.mashibing.mult;

import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties("spring.datasource.remote")
    public DataSource remoteDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.local")
    public DataSource localDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean(name = "dynamicDataSource")
    @Primary
    public DynamicDataSource dataSource(DataSource remoteDataSource, DataSource localDataSource) {
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put(DataSourceType.REMOTE.name(), remoteDataSource);
        targetDataSources.put(DataSourceType.LOCAL.name(), localDataSource);
        return new DynamicDataSource(remoteDataSource, targetDataSources);
    }
}
```

6、自定义多数据源切换注解

​		设置拦截数据源的注解，可以设置在具体的类上，或者在具体的方法上

```java
package com.mashibing.mult;

import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    /**
     * 切换数据源名称
     */
    DataSourceType value() default DataSourceType.REMOTE;
}
```

7、AOP拦截类的实现

​		通过拦截上面的注解，在其执行之前处理设置当前执行SQL的数据源的信息，CONTEXT_HOLDER.set(dataSourceType)这里的数据源信息从我们设置的注解上面获取信息，如果没有设置就是用默认的数据源的信息。

```java
package com.mashibing.mult;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

@Aspect
@Order(1)
@Component
public class DataSourceAspect {

    @Pointcut("@annotation(com.mashibing.mult.DataSource)")
    public void dsPointCut() {

    }

    @Around("dsPointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        DataSource dataSource = method.getAnnotation(DataSource.class);
        if (dataSource != null) {
            DynamicDataSourceContextHolder.setDataSourceType(dataSource.value().name());
        }
        try {
            return point.proceed();
        } finally {
            // 销毁数据源 在执行方法之后
            DynamicDataSourceContextHolder.clearDataSourceType();
        }
    }
}
```

8、使用切换数据源注解

```java
package com.mashibing.mult;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;
import java.util.Map;


@RestController
public class EmpController {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @GetMapping("/local")
    @DataSource(value = DataSourceType.LOCAL)
    public List<Map<String, Object>> local(){
        List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from emp");
        return maps;
    }
    @GetMapping("/remote")
    @DataSource(value = DataSourceType.REMOTE)
    public List<Map<String, Object>> remote(){
        List<Map<String, Object>> maps = jdbcTemplate.queryForList("select * from emp");
        return maps;
    }

}
```

9、在启动项目的过程中会发生循环依赖的问题，直接修改启动类即可

```java
package com.mashibing;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class SpringbootDataApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDataApplication.class, args);
    }
}
```

