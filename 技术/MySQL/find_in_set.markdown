首先举个例子来说：
有个文章表里面有个type字段，它存储的是文章类型，有 1头条、2推荐、3热点、4图文等等 。
现在有篇文章他既是头条，又是热点，还是图文，type中以 1,3,4 的格式存储。那我们如何用sql查找所有type中有4的图文类型的文章呢？？
这就要我们的 find_in_set 出马的时候到了。以下为引用的内容：

```sql
select * from article where FIND_IN_SET('4',type)
```

FIND_IN_SET(str,strlist)

str 要查询的字符串
strlist 字段名 参数以”,”分隔 如 (1,2,6,8)
查询字段(strlist)中包含(str)的结果，返回结果为null或记录

假如字符串str在由N个子链组成的字符串列表strlist 中，则返回值的范围在 1 到 N 之间。 一个字符串列表就是一个由一些被 ‘,’ 符号分开的子链组成的字符串。如果第一个参数是一个常数字符串，而第二个是type SET列，则FIND_IN_SET() 函数被优化，使用比特计算。 如果str不在strlist 或strlist 为空字符串，则返回值为 0 。如任意一个参数为NULL，则返回值为 NULL。这个函数在第一个参数包含一个逗号(‘,’)时将无法正常运行。

 **例子：**
mysql> SELECT FIND_IN_SET('b', 'a,b,c,d');
-> 2 因为b 在strlist集合中放在2的位置 从1开始

select FIND_IN_SET('1', '1'); 返回 就是1 这时候的strlist集合有点特殊 只有一个字符串 其实就是要求前一个字符串 一定要在后一个字符串集合中才返回大于0的数
select FIND_IN_SET('2', '1，2'); 返回2
select FIND_IN_SET('6', '1'); 返回0

**注意:**
select * from treenodes where FIND_IN_SET(id, '1,2,3,4,5');
使用find_in_set函数一次返回多条记录
id 是一个表的字段，然后每条记录分别是id等于1，2，3，4，5的时候
有点类似in （集合）
select * from treenodes where id in (1,2,3,4,5);

原来以为mysql可以进行这样的查询：

```sql
SELECT id,name,list from tb_test WHERE 'daodao' IN(list); -- (一) 
```

![](D:\MyWork\MarkDownPicture\mysql\find_in_set.png)

实际上这样是不行的，这样只有当list字段的值等于'daodao'时（和IN前面的字符串完全匹配），查询才有效，否则都得不到结果，即使'daodao'真的在list中。

```sql
SELECT id,name,list from tb_test WHERE 'daodao' IN ('libk', 'zyfon', 'daodao'); -- (二)
```

![](D:\MyWork\MarkDownPicture\mysql\find_in_set(1).png)

这样是可以的。

这两条到底有什么区别呢？为什么第一条不能取得正确的结果，而第二条却能取得结果。原因其实是（一）中 (list) list是变量， 而（二）中 ('libk', 'zyfon', 'daodao')是常量。
所以如果要让（一）能正确工作，需要用find_in_set():

**总结：**
所以如果list是常量，则可以直接用IN， 否则要用find_in_set()函数。

**find_in_set()和like的区别：**

在mysql中，有时我们在做数据库查询时，需要得到某字段中包含某个值的记录，但是它也不是用like能解决的，使用like可能查到我们不想要的记录，它比like更精准，这时候mysql的FIND_IN_SET函数就派上用场了，下面来看一个例子。

```sql
CREATE TABLE users(
    id int(6) NOT NULL AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    limits VARCHAR(50) NOT NULL, -- 权限
    PRIMARY KEY (id)
);

INSERT INTO users(name, limits) VALUES('小张','1,2,12'); 
INSERT INTO users(name, limits) VALUES('小王','11,22,32');
```

```sql
SELECT * FROM users WHERE limits LIKE '%2%';
```

![](D:\MyWork\MarkDownPicture\mysql\like.png)

这样第二条数据不具有权限'2'的用户也查出来了，不符合预期。下面利用mysql 函数find_in_set()来解决。

```sql
SELECT * FROM users WHERE FIND_IN_SET(2,limits);
```

![](D:\MyWork\MarkDownPicture\mysql\find_in_set_like.png)

这样就能达到我们预期的效果，问题就解决了！

**注意**：mysql字符串函数 find_in_set(str1,str2)函数是返回str2中str1所在的位置索引，str2必须以","分割开。

**总结：**like是广泛的模糊匹配，字符串中没有分隔符，Find_IN_SET 是精确匹配，字段值以英文”,”分隔，Find_IN_SET查询的结果要小于like查询的结果。