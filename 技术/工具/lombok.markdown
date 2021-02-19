在我们写controller或者Service层的时候，需要注入很多的mapper接口或者另外的service接口，这时候就会写很多的@AutoWired注解，代码看起来很乱
lombok提供了一个注解：

```java
@RequiredArgsConstructor(onConstructor=@_(@Autowired))
写在类上可以代替@AutoWired注解，需要注意的是在注入时需要用final定义，或者使用@notnull注解

private final User u;
```

- [@Data](https://projectlombok.org/features/Data) 注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、hashCode、toString 方法
- [@Setter](https://projectlombok.org/features/GetterSetter) ：注解在属性上；为属性提供 setting 方法
- [@Setter](https://projectlombok.org/features/GetterSetter) ：注解在属性上；为属性提供 getting 方法
- [@Log4j](https://projectlombok.org/features/Log4j) ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象
- [@NoArgsConstructor](https://projectlombok.org/features/constructor) ：注解在类上；为类提供一个无参的构造方法
- [@AllArgsConstructor](https://projectlombok.org/features/constructor) ：注解在类上；为类提供一个全参的构造方法
- [@Cleanup](https://projectlombok.org/features/Cleanup) : 可以关闭流
- [@Builder](https://projectlombok.org/features/Builder) ： 被注解的类加个构造者模式
- [@Synchronized](https://projectlombok.org/features/Synchronized) ： 加个同步锁
- [@SneakyThrows](https://projectlombok.org/features/SneakyThrows) : 等同于try/catch 捕获异常
- [@NonNull](https://projectlombok.org/features/NonNull) : 如果给参数加个这个注解 参数为null会抛出空指针异常
- @Value : 注解和@Data类似，区别在于它会把所有成员变量默认定义为private final修饰，并且不会生成set方法。