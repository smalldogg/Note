## 一：spring常用配置

### 1. spring el

支持在xml和注解中使用表达式

主要是使用@Value

使用的地方



```java
@Data
@Component
public class Book {
    @Value("${book.name}")//配置文件中的值
    private String name;
    @Value("${book.author}")
    private String author;
    @Value("other")//普通字符串
    private String other;
    
}
```

### 2. bean的初始化和销毁

initMethod 和 destroyMethod

使用注解的方式

```java
@PostConstruct //在构造函数执行完之后执行
public void init() {
    System.out.println("init......");
}

@PreDestroy//在bean销毁之前执行
public void destroy() {
    System.out.println("destroy");
}
```

### 3. profile

使用配置的方式

![](D:\MyWork\MarkDownPicture\spring\profile.png)

配合使用

```yml
profiles:
  active: dev
```

### 4. 事件

## 二：spring高级话题

### 1. Aware

<img src="D:\MyWork\MarkDownPicture\spring\aware.png" style="zoom:67%;" />

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```

### 2. 多线程

Spring通过任务执行器（TaskEXecutor）来实现多线程和并发编程。使用ThreadPoolTaskExecutor可实现一套基于线程池的TaskExecutor。在配置类中通过@EnableAsync开启对异步任务的支持，并通过在实际执行的Bean的方法中使用@Async注解来声明是一个异步任务。

```java
@Configuration
@EnableAsync
public class AsyncTask implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("myThreadPool");
        /*
           rejection-policy：当pool已经达到max size的时候，如何处理新任务
           CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
        */
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```



### 3. 计划任务

```java
@EnableScheduling
@Component
public class MySchedule {

    @Scheduled(cron = "0/10 * * * * ?")
    public void log() {
        System.out.println("cron");
    }

    @Scheduled(fixedDelay = 5000)
    public void test1() {

    }

    @Scheduled(fixedRate = 1000)
    public void test2() {

    }
}
```

fixedDelay非常好理解，它的间隔时间是根据上次的任务结束的时候开始计时的。比如一个方法上设置了fixedDelay=5*1000，那么当该方法某一次执行结束后，开始计算时间，当时间达到5秒，就开始再次执行该方法。

fixedRate理解起来比较麻烦，它的间隔时间是根据上次任务开始的时候计时的。比如当方法上设置了fiexdRate=5*1000，该执行该方法所花的时间是2秒，那么3秒后就会再次执行该方法。
但是这里有个坑，当任务执行时长超过设置的间隔时长，那会是什么结果呢。打个比方，比如一个任务本来只需要花2秒就能执行完成，我所设置的fixedRate=5*1000，但是因为网络问题导致这个任务花了7秒才执行完成。当任务开始时Spring就会给这个任务计时，5秒钟时候Spring就会再次调用这个任务，可是发现原来的任务还在执行，这个时候第二个任务就阻塞了（这里只考虑单线程的情况下，多线程后面再讲），甚至如果第一个任务花费的时间过长，还可能会使第三第四个任务被阻塞。被阻塞的任务就像排队的人一样，一旦前一个任务没了，它就立马执行

### 4. 条件注解 @Conditional

@Conditional根据满足某一特定条件创建哟个特定的Bean。我们可以利用这个特性进行一些自动配置

```java
public class WindowsCondition implements Condition {

    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata metadata) {
        //获取ioc使用的beanFactory
        ConfigurableListableBeanFactory beanFactory = conditionContext.getBeanFactory();
        //获取类加载器
        ClassLoader classLoader = conditionContext.getClassLoader();
        //获取当前环境信息
        Environment environment = conditionContext.getEnvironment();
        //获取bean定义的注册类
        BeanDefinitionRegistry registry = conditionContext.getRegistry();
        //获得当前系统名
        String property = environment.getProperty("os.name");
        //包含Windows则说明是windows系统，返回true
        if (property.contains("Windows")) {
            return true;
        }
        return false;
    }
}



@Configuration
@Conditional({WindowsCondition.class})
public class BeanConfig {

    @Bean
    public Book book1() {
        Book book = new Book();
        book.setAuthor("windows");
        return book;
    }
}
```

### @Enable*注解

## 三：springmvc