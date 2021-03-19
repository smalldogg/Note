## spring中的常用配置

### 1. scope

​	(1)  singleton默认的单例

​	(2)  prototype 每次调用都会创建一个bean的实例

​	(3) request

​	(4) session

​	(5)globalsession

### 2. EL

​	spring开发经常涉及调用各种资源的情况，我们使用spring的表达式语言实现资源的注入

​	 (1) 普通字符串 (常用)

​	 (2) 属性文件（常用）

### 3. Bean的初始化和销毁

生命周期的方法

#### init-method 构造函数执行完之后执行  @PostConstrcut

#### destory-method 在bean销毁之前执行 @PreDestory

### 4. Profile

在不同环境下使用不同的配置提供支持

**通过设定environment的activeprofiles**来设定当前context需要使用的配置环境

1. 在开发中使用**@profile**注解
2. 同归设定jvm的spring.profiles.active来设定
3. web项目设置servlet的context parameter中

### 5. 事件

当一个bean处理完一个任务之后，希望另外一个bean知道并能做相应的处理，这时我们需要让另外一个bean监听当前bean所发送的事件

1. 自定义时间，集成applicationEvent
2. 定义事件监听器，实现Applicationlistener
3. 使用容器发布事件





## spring高级换题

spring的依赖注入最大的亮点就是你所有的bean对spring容器的存在是没有意识的

当你要使用spring容器本身的功能资源，这时你的bean必须要意识到spring容器的存在

![](D:\MyWork\MarkDownPicture\spring\aware.png)

### 多线程

spring通过任务执行器来实现多线程和并发编程。

1. @EnablesAsync 开启对异步任务的支持

2. @Async注解声明其是一个异步任务

```java
@Configuration
@EnableAsync//开启异步调用
public class ThreadExecutorConfig {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    /** 核心线程数 */
    private int corePoolSize = 10;
    /** 最大线程数 */
    private int maxPoolSize = 200;
    /** 队列数 */
    private int queueCapacity = 10;

    /**
     * @Configuration = <beans></beans>
     * @Bean = <bean></bean>
     * 返回值类型为<bean></bean>中的属性"class"对应的value
     * 方法名为<bean></bean>中的属性"id"对应的value
     * @return
     */
    @Bean
    public ExecutorService testFxbDrawExecutor(){
        logger.info("start executor testExecutor ");
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.setThreadNamePrefix("test-fxb-draw-service-");

        // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 执行初始化
        executor.initialize();
        return executor.getThreadPoolExecutor();
    }

}
```

### 条件注解@Conditional

在满足某一个特定条件创建一个特定的Bean

@Conditional

### 组合注解与元注解

![](D:\MyWork\MarkDownPicture\spring\enable.png)





## SpringMvc

### 拦截器 HandleInterceptorAdapter

```java
public class Handler implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }
}
```

### @ControllerAdvice

![](D:\MyWork\MarkDownPicture\spring\ControllerAdvice.png)

​	   

## Spring Mvc 的高级配置

### 文件上传

![](D:\MyWork\MarkDownPicture\spring\文件上传.png)

## springboot

