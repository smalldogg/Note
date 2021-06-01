**1. 代码层防止sql注入攻击的最佳方案就是sql预编译**

```java
public List<Course> orderList(String studentId){
    String sql = "select id,course_id,student_id,status from course where student_id = ?";
    return jdbcTemplate.query(sql,new Object[]{studentId},new BeanPropertyRowMapper(Course.class));
}
```

这样我们传进来的参数 `4 or 1 = 1`就会被当作是一个`student_id`，所以就不会出现sql注入了。

**2. 确认每种数据的类型，比如是数字，数据库则必须使用int类型来存储**

**3. 规定数据长度，能在一定程度上防止sql注入**

**4. 严格限制数据库权限，能最大程度减少sql注入的危害**

**5. 避免直接响应一些sql异常信息，sql发生异常后，自定义异常进行响应**

**6. 过滤参数中含有的一些数据库关键词**

```java
@Component
public class SqlInjectionFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest req=(HttpServletRequest)servletRequest;
        HttpServletRequest res=(HttpServletRequest)servletResponse;
        //获得所有请求参数名
        Enumeration params = req.getParameterNames();
        String sql = "";
        while (params.hasMoreElements()) {
            // 得到参数名
            String name = params.nextElement().toString();
            // 得到参数对应值
            String[] value = req.getParameterValues(name);
            for (int i = 0; i < value.length; i++) {
                sql = sql + value[i];
            }
        }
        if (sqlValidate(sql)) {
            throw new IOException("您发送请求中的参数中含有非法字符");
        } else {
            chain.doFilter(servletRequest,servletResponse);
        }
    }

    /**
     * 关键词校验
     * @param str
     * @return
     */
    protected static boolean sqlValidate(String str) {
        // 统一转为小写
        str = str.toLowerCase();
        // 过滤掉的sql关键字，可以手动添加
        String badStr = "'|and|exec|execute|insert|select|delete|update|count|drop|*|%|chr|mid|master|truncate|" +
                "char|declare|sitename|net user|xp_cmdshell|;|or|-|+|,|like'|and|exec|execute|insert|create|drop|" +
                "table|from|grant|use|group_concat|column_name|" +
                "information_schema.columns|table_schema|union|where|select|delete|update|order|by|count|*|" +
                "chr|mid|master|truncate|char|declare|or|;|-|--|+|,|like|//|/|%|#";
        String[] badStrs = badStr.split("\\|");
        for (int i = 0; i < badStrs.length; i++) {
            if (str.indexOf(badStrs[i]) >= 0) {
                return true;
            }
        }
        return false;
    }
}
```

