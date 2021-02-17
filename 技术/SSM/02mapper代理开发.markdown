## mapper代理开发

**一、概要**

**1、原始DAO开发中存在的问题:
**（1）DAO实现方法体中存在很多过程性代码。
（2）调用SqlSession的方法(select/insert/update)需要指定Statement的id，存在硬编码，不利于代码维护。

**2、Mapper动态代理方法**：程序员只需要写dao接口(Mapper)，而不需要写dao实现类，由mybatis根据dao接口和映射文件中statement的定义生成接口实现类代理对象。

**3、目标：**通过一些规则让mybatis根据dao接口和映射文件中statement的定义生成接口实现代理对象。

**二、开发规范**

1、在XXXmapper.xml中namespace等于mapper接口地址（即mapper.xml文件中的namespace与mapper.java接口的类路径相同）。

![](D:\MyWork\MarkDownPicture\mybatis\mybatis_namespace.png)

2、XXXmapper.java接口中的方法和mapper.xml中的statement的Id一致。
3、mapper.java接口中的方法输入参数和mapper.xml中statement的parameterType指定的类型一致。
4、mapper.java接口中的方法的返回值类型和mapper.xml中statement的resultType指定的类型一致。

![](D:\MyWork\MarkDownPicture\mybatis\mapper.png)

 **三、UserMapper.java类代码（接口文件）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
package com.mybatis.mapper;

import java.util.List;

import com.mybatis.entity.User;

/**
 * 用户管理mapper接口
 * @author lxx
 *
 */
public interface UserMapper {
    
    /** 根据ID查询用户信息 */
    public User findUserById(int id);

    /** 根据用户名称模糊查询用户信息 */
    public List<User> findUserByName(String username);

    /** 添加用户 */
    public void insertUser(User user);

    /** 根据ID删除用户 */
    public void deleteUser(Integer id);

    /** 根据ID更新用户 */
    public void updateUser(User user);

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**四、将原来的User.xml拷贝并修改名称为UserMapper.xml，再将UserMapper.xml文件中的namespace改为mapper接口地址**

```xml
 <!-- namespace命名空间,作用就是对sql进行分类化的管理,理解为sql隔离
    注意:使用mapper代理开发时，namespace有特殊作用
 -->
<mapper namespace="com.mybatis.mapper.UserMapper">
```

注：namespace=mapper接口地址
**五、在SqlMapConfig.xml中加载UserMapper.xml**

```xml
<!-- 加载映射文件 -->
    <mappers>
        <mapper resource="com/mybatis/mapping/User.xml"/>
        <mapper resource="com/mybatis/mapping/UserMapper.xml"/>
    </mappers>
```

**六、JUnit测试UserMapperTest.java**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
package com.mybatis.test;

import java.io.InputStream;
import java.util.Date;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import com.mybatis.entity.User;
import com.mybatis.mapper.UserMapper;

public class UserMapperTest {

    private SqlSessionFactory sqlSessionFactory;

    // 此方法是在执行@Test标注的方法之前执行
    @Before
    public void setUp() throws Exception {
        String resource = "SqlMapConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        // 创建SqlSessionFcatory
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }
    
    @Test
    public void testFindUserById() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建Usermapper对象，mybatis自动生成mapper代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.findUserById(1);
        System.out.println(user);
        sqlSession.close();
    }

    @Test
    public void testFindUserByName() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建Usermapper对象，mybatis自动生成mapper代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = mapper.findUserByName("小");
        System.out.println(list);
        sqlSession.close();
    }
    
    @Test
    public void testInsertUser() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建Usermapper对象，mybatis自动生成mapper代理对象
        User user = new User();
        user.setUsername("小东");
        user.setSex("1");
        user.setAddress("天津");
        user.setBirthday(new Date());
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.insertUser(user);
        sqlSession.commit();
        sqlSession.close();
    }
    
    @Test
    public void testUpdateUser() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建Usermapper对象，mybatis自动生成mapper代理对象
        User user = new User();
        user.setId(2);//必须设置Id
        user.setUsername("小刘");
        user.setSex("1");
        user.setAddress("北京海淀");
        user.setBirthday(new Date());
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.updateUser(user);
        sqlSession.commit();
        sqlSession.close();
    }
    
    @Test
    public void testDeleteUser() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 创建Usermapper对象，mybatis自动生成mapper代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        mapper.deleteUser(3);
        sqlSession.commit();
        sqlSession.close();
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   原来这个sqlSession可以自动创建一个mapper接口的代理对象！我们只需要把刚刚写好的mapper接口类的字节码对象传给getMapper方法，即可得到一个该接口对应的代理对象，然后我们就可以使用这个代理对象来操作接口中具体的方法了。
　　到这里，使用mapper代理的方式开发dao就总结完了，但是有个小细节，由于mapper接口中方法的参数要根据映射文件中的parameterType来指定，而parameterType只有一个，所以mapper接口中所有方法的参数都只有一个！那如果我们要传入两个或多个参数该咋整？这没办法，想要传多个参数还是死了这条心了吧，但是可以解决这个问题，就是对传入的对象进行增强，让传进去的对象包含我们需要的参数即可。这算是个小弊端吧，但是不会影响我们开发。

**七、小结**

**1、用mapper代理开发时只要写2个：**

（1）mapper.xml

（2）mapper接口

**2、Mapper接口开发需要遵循以下规范：**

（1）Mapper.xml文件中的namespace与mapper接口的类路径相同。
（2）Mapper接口方法名和Mapper.xml中定义的每个statement的id相同。 
（3）Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql 的parameterType的类型相同。
（4）Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同。

**3、代理对象内部调用selectOne()和selectList():**

如果mapper对象返回单个pojo对象(非集合对象)代理对象内部通过selectOne查询数据库，如果mapper方法返回集合对象，代理对象内部通过selectList查询数据库。

**4、mapper接口中的方法参数只能有一个是否影响系统开发，mapper接口方法参数只能有一个，系统是否不利于维护？**
回答：系统框架中，dao层的代码是被业务层公用的。mapper接口只有一个参数，可以使用包装类型的pojo满足不同的业务方法的需求。