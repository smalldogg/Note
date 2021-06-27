```sql
SET @curRank := 0;
```

使用 子查询来声明用户自定义变量：

```sql
(SELECT @curRank :=1) a 
```

```sql
CREATE TABLE `employee` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `salary` int(11) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COMMENT='员工';


INSERT INTO `employee` (`id`, `name`, `salary`,`d_id`) VALUES
(1, 'Samual', 2500, 1),
(2, 'Vino', 2000,1),
(3, 'John', 2000,1),
(4, 'Andy', 2200,1),
(5, 'Brian', 2100,2),
(6, 'Dew', 2400,2),
(7, 'Kris', 2500,1),
(8, 'William', 2600,1),
(9, 'George', 2300,2),
(10, 'Peter', 1900,2),
(11, 'Tom', 2000,2),
(12, 'Andre', 2000,1);

```

**1、在MySQL中实现Rank普通排名函数**
在这里，我们希望获得一个排名字段的列，以及salary的升序排列。所以我们的查询语句将是：

```sql
SELECT id, name, salary, @curRank := @curRank + 1 AS rank
FROM employee e, (SELECT @curRank := 0) q
ORDER BY salary
```

要在mysql中声明一个变量，你必须在变量名之前使用@符号。FROM子句中的(@curRank := 0)部分允许我们进行变量初始化，而不需要单独的SET命令。当然，也可以使用SET，但它会处理两个查询：

**3、在MySQL中实现Rank普通并列排名函数**
现在，如果我们希望为并列数据的行赋予相同的排名，则意味着那些在排名比较列中具有相同值的行应在MySQL中计算排名时保持相同的排名(例如在我们的例子中的salary)。为此，我们使用了一个额外的变量。

```sql
SELECT id, name, salary, 
CASE 
WHEN @prevRank = salary THEN @curRank 
WHEN @prevRank := salary THEN @curRank := @curRank + 1
END AS rank
FROM employee e, 
(SELECT @curRank :=0, @prevRank := NULL) r
ORDER BY salary
```

**4、在MySQL中实现Rank高级并列排名函数**
当使用RANK()函数时，如果两个或以上的行排名并列，则相同的行都会有相同的排名，但是实际排名中存在有关系的差距。

```sql
SELECT id, name, salary, rank FROM
(SELECT id, name, salary,
@curRank := IF(@prevRank = salary, @curRank, @incRank) AS rank, 
@incRank := @incRank + 1, 
@prevRank := salary
FROM employee e, (
SELECT @curRank :=0, @prevRank := NULL, @incRank := 1
) r 
ORDER BY salary) s
```

这是一个查询中的子查询。我们使用三个变量(@incRank，@prevRank，@curRank)来计算关系的情况下，在查询结果中我们已经补全了因为并列而导致的排名空位。我们已经封闭子查询到查询。这个查询相当于MySQL和ORACLE中的RANK()函数。