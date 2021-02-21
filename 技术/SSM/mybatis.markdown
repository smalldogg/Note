```
经常会见到以下两种写法:
1、 #{bookId}
2、 #{bookId,jdbcType=INTEGER}
一般情况下，两种写法都可以。它们都可以获取Dao层传递过来的参数。
但是，当传入的参数为null时，需要指定jdbcType的类型,否则mybatis无法解析。
```