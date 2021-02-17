## 动态sql

MyBatis 的强大特性之一便是它的动态 SQL。如果你有使用 JDBC 或其他类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句有多么痛苦。拼接的时候要确保不能忘了必要的空格，还要注意省掉列名列表最后的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

**名词解析：**OGNL表达式

OGNL，全称为Object-Graph Navigation Language，它是一个功能强大的表达式语言，用来获取和设置Java对象的属性，它旨在提供一个更高的更抽象的层次来对Java对象图进行导航。

OGNL表达式的基本单位是"导航链"，一般导航链由如下几个部分组成：

（1）属性名称（property）

（2）方法调用（method invoke）

（3）数组元素

所有的OGNL表达式都基于当前对象的上下文来完成求值运算，链的前面部分的结果将作为后面求值的上下文。例如：names[0].length()。

------

 mybatis 的**动态sql语句**是**基于OGNL表达式**的。可以方便的在 sql 语句中实现某些逻辑. 总体说来mybatis 动态SQL 语句主要有以下几类:
　　1. **if** 语句 (简单的条件判断)
　　2. **choose** (when,otherwize) ,相当于java 语言中的 switch ,与 jstl 中的choose 很类似.
　　3. **trim** (对包含的内容加上 prefix,或者 suffix 等，前缀，后缀)
　　4. **where** (主要是用来简化sql语句中where条件判断的，能智能的处理 and or ,不必担心多余导致语法错误)
　　5. **set** (主要用于更新时)
　　6. **foreach** (在实现 mybatis in 语句查询时特别有用)
下面分别介绍这几种处理方式

1、mybatis if语句处理

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
<select id="dynamicIfTest" parameterType="Blog" resultType="Blog">
    select * from t_blog where 1 = 1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="content != null">
        and content = #{content}
    </if>
    <if test="owner != null">
        and owner = #{owner}
    </if>
</select>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**解析**：

　　如果你提供了title参数，那么就要满足title=#{title}，同样如果你提供了Content和Owner的时候，它们也需要满足相应的条件，之后就是返回满足这些条件的所有Blog，这是非常有用的一个功能。

　　以往我们使用其他类型框架或者直接**使用JDBC的时候**， 如果我们要达到同样的选择效果的时候，我们就**需要拼SQL语句**，这是极其麻烦的，比起来，上述的动态SQL就要简单多了。

2、**choose** (when,otherwize) ,相当于java 语言中的 switch ,与 jstl 中的choose 很类似

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
<select id="dynamicChooseTest" parameterType="Blog" resultType="Blog">
    select * from t_blog where 1 = 1 
    <choose>
        <when test="title != null">
            and title = #{title}
        </when>
        <when test="content != null">
            and content = #{content}
        </when>
        <otherwise>
            and owner = "owner1"
        </otherwise>
    </choose>
</select>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   when元素表示当when中的条件满足的时候就输出其中的内容，**跟JAVA中的switch效果差不多**的是按照条件的顺序，**当when中有条件满足的时候，就会跳出choose**，即所有的when和otherwise条件中，只有一个会输出，当所有的我很条件都不满足的时候就输出otherwise中的内容。所以上述语句的意思非常简单，当title!=null的时候就输出and titlte = #{title}，不再往下判断条件，当title为空且content!=null的时候就输出and content = #{content}，当所有条件都不满足的时候就输出otherwise中的内容。

3、**trim** (对包含的内容加上 prefix,或者 suffix 等，前缀，后缀)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
<select id="dynamicTrimTest" parameterType="Blog" resultType="Blog">
    select * from t_blog 
    <trim prefix="where" prefixOverrides="and |or">
        <if test="title != null">
            title = #{title}
        </if>
        <if test="content != null">
            and content = #{content}
        </if>
        <if test="owner != null">
            or owner = #{owner}
        </if>
    </trim>
</select>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

  trim元素的主要**功能**是可以**在自己包含的内容前加上某些前缀**，**也可以在其后加上某些后缀**，与之对应的属性是prefix和suffix；可以把包含内容的首部某些内容覆盖，即忽略，也可以把尾部的某些内容覆盖，对应的属性是prefixOverrides和suffixOverrides；正因为trim有这样的功能，所以我们也可以非常简单的利用trim来代替where元素的功能。

trim标记是一个格式化的标记，可以完成set或者是where标记的功能，如下代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
select * from user 
<trim prefix="WHERE" prefixoverride="AND |OR">
    <if test="name != null and name.length()>0"> 
        AND name=#{name}
    </if>
    <if test="gender != null and gender.length()>0"> 
        AND gender=#{gender}
    </if>
</trim>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

假如说name和gender的值都不为null的话打印的SQL为：select * from user where  name = 'xx' and gender = 'xx'

在红色标记的地方是不存在第一个and的，上面两个属性的意思如下：

prefix：前缀　　　　　　

prefixoverride：去掉第一个and或者是or

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
update user
<trim prefix="set" suffixoverride="," suffix=" where id = #{id} ">
    <if test="name != null and name.length()>0">
        name=#{name} ,
    </if>
    <if test="gender != null and gender.length()>0">
        gender=#{gender} ,  
    </if>
</trim>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

假如说name和gender的值都不为null的话打印的SQL为：update user set name='xx' , gender='xx'   where id='x'

在红色标记的地方不存在逗号，而且自动加了一个set前缀和where后缀，上面三个属性的意义如下，其中prefix意义如上：

suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）

suffix：后缀

4、**where** (主要是用来简化sql语句中where条件判断的，能智能的处理 and or 条件)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
<select id="dynamicWhereTest" parameterType="Blog" resultType="Blog">
    select * from t_blog 
    <where>
        <if test="title != null">
            title = #{title}
        </if>
        <if test="content != null">
            and content = #{content}
        </if>
        <if test="owner != null">
            and owner = #{owner}
        </if>
    </where>
</select>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   where元素的**作用**是会**在写入where元素的地方输出一个where**，另外一个**好处**是你**不需要考虑where元素里面的条件输出是什么样子的**，MyBatis会智能的帮你处理，如果所有的条件都不满足那么MyBatis就会查出所有的记录，如果输出后是and 开头的，MyBatis会把第一个and忽略，当然如果是or开头的，MyBatis也会把它忽略；此外，在where元素中你不需要考虑空格的问题，MyBatis会智能的帮你加上。像上述例子中，如果title=null， 而content != null，那么输出的整个语句会是select * from t_blog where content = #{content}，而不是select * from t_blog where and content = #{content}，因为MyBatis会智能的把首个and 或 or 给忽略。

5、**set** (主要**用于更新**时) 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
<update id="dynamicSetTest" parameterType="Blog">
    update t_blog
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="content != null">
            content = #{content},
        </if>
        <if test="owner != null">
            owner = #{owner}
        </if>
    </set>
    where id = #{id}
</update>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   set元素主要是用在更新操作的时候，它的主要**功能和where元素其实是差不多的**，主要是在包含的语句前输出一个set，然后如果包含的语句是以逗号结束的话将会把该逗号忽略，如果set包含的内容为空的话则会出错。有了set元素我们就可以动态的更新那些修改了的字段。

6、**foreach** (在实现 mybatis **in 语句查询时特别有用**)

foreach的主要用在构建in条件中，它可以在SQL语句中进行迭代一个集合。foreach元素的属性主要有item，index，collection，open，separator，close。

**（1）item**表示集合中每一个**元素**进行迭代时的**别名。**

**（2）index**指定一个名字，用于表示在迭代过程中，每次**迭代到**的**位置。**

**（3）open**表示该语句**以什么开始。**

**（4）separator**表示在每次进行迭代之间以什么符号作为**分隔符。**

**（5）close**表示**以什么结束。**

在使用foreach的时候最关键的也是最容易出错的就是**collection属性**，该属性是**必须指定的**，但是在不同情况下，该属性的值是不一样的，主要有一下3种情况：

（1）如果传入的是**单参数**且**参数类型是一个List**的时候，collection属性值为**list**

（2）如果传入的是**单参数**且参数**类型是一个array数组**的时候，collection的属性值为**array**

（3）如果传入的**参数是多个**的时候，我们就需要把它们封装成一个Map了，当然单参数也可以封装成map，实际上如果你在传入参数的时候，在MyBatis里面也是会把它封装成一个Map的，map的key就是参数名，所以这个时候collection属性值就是传入的List或array对象在自己封装的map里面的key。

**6.1、单参数List的类型**

```xml
<select id="dynamicForeachTest" resultType="com.mybatis.entity.User">
    select * from t_user where id in
    <foreach collection="list" index="index" item="item" open="(" separator="," close=")">
        #{item}
    </foreach>
</select>
```

上述collection的值为list，对应的**Mapper**是这样的：

```java
/**mybatis Foreach测试 */
public List<User> dynamicForeachTest(List<Integer> ids); 
```

测试代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Test
public void dynamicForeachTest() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<Integer> ids = new ArrayList<Integer>();
    ids.add(1);
    ids.add(2);
    ids.add(6);
    List<User> userList = mapper.dynamicForeachTest(ids);
    for (User user : userList){
        System.out.println(user);
    }
    sqlSession.close();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**6.2、数组类型的参数**

```javascript
<select id="dynamicForeach2Test" resultType="com.mybatis.entity.User">
    select * from t_user where id in
    <foreach collection="array" index="index" item="item" open="(" separator="," close=")">
        #{item}
    </foreach>
</select>
```

对应mapper：

```java
public List<User> dynamicForeach2Test(int[] ids);  
```

测试代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Test
public void dynamicForeach2Test() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int[] ids = {1,2,6};
    List<User> userList = mapper.dynamicForeach2Test(ids);
    for (User user : userList){
        System.out.println(user);
    }
    sqlSession.close();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**6.3、Map类型的参数**

```javascript
<select id="dynamicForeach3Test" resultType="com.mybatis.entity.User">
    select * from t_user where username like '%${username}%' and id in
    <foreach collection="ids" index="index" item="item" open="(" separator="," close=")">
        #{item}
    </foreach>
</select>
```

mapper 应该是这样的接口:

```java
/**mybatis Foreach测试 */
public List<User> dynamicForeach3Test(Map<String, Object> params); 
```

测试方法：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
@Test
public void dynamicForeach3Test() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<Integer> ids = new ArrayList<Integer>();
    ids.add(1);
    ids.add(2);
    ids.add(6);
    Map map =new HashMap();
    map.put("username", "小");
    map.put("ids", ids);
    List<User> userList = mapper.dynamicForeach3Test(map);
    System.out.println("------------------------");
    for (User user : userList){
        System.out.println(user);
    }
    sqlSession.close();
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

通过以上方法，就能完成一般的mybatis 的 动态SQL 语句.**最常用**的就是 if where foreach这几个，一定要重点掌握.





https://www.cnblogs.com/xiaoxi/p/6406504.html