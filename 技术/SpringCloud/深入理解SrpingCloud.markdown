## eureka

1. eureka client 的注册延迟

   启动之后，不是立即向eureka server注册的，默认是40秒

2. eureka server的响应缓存

   每30秒更新一次缓存，所以刚刚注册的实例，也不会立即出现在注册列表中