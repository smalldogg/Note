## Mysql索引

MySQL中的索引是以B+树的形式组织的，为什么使用B+树，B+树相对于B树来说，每层可以存放的节点会更多，那么整体上，树的高度就会低，那么会减少IO的次数。

## 数据查询

MySQL在查询的时候，并不是查询单条数据，而是将整个数据页加载到内存中，这个可以在innodb_buffer_size控制内存缓冲的大小，一个数据页的大小为16Kb，这也是为什么不推荐使用uuid作为主键的原因， 使用uuid作为主键的话，会导致频繁的数据页的分裂和合并

## Explain工具

### explain中的列

#### id 

id列的编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的。
id列越大执行优先级越高，id相同则从上往下执行，id为NULL最后执行  

#### select_type列  

select_type 表示对应行是简单还是复杂的查询  

simple：简单查询。查询不包含子查询和union  

primary：复杂查询中最外层的 select  

subquery：包含在 select 中的子查询（不在 from 子句中）  

derived：包含在 from 子句中的子查询。MySQL会将结果存放在一个临时表中，也称为派生表（derived的英文含义）  

#### table列  

这一列表示 explain 的一行正在访问哪个表 

#### type列 

依次从最优到最差分别为：**system > const > eq_ref > ref > range > index > ALL
**一般来说，得保证查询达到range级别，最好达到ref

**const** 对于单表主键索引的查找

**eq_ref**对于关联表的主键或者唯一索引查询

**ref**关联表的非主键（普通索引）或者唯一性索引的部分索引（联合索引的部分）

**range**范围查询，要用到范围要有区间范围扫描通常出现在 in(), between ,> ,<, >= 等操作中  

**index**开区间的范围查询,扫面全表索引

#### possible_keys列  

这一列显示查询可能使用哪些索引来查找。  

#### key列  

实际使用的索引

#### key_len列  

索引的长度，在使用联合索引的情况下，可以推算出使用了哪些字段

#### Extra列  

这一列展示的是额外信息。常见的重要值如下： 

Using index：使用覆盖索引  

​				explain select film_id from film_actor where film_id = 1;  

Using index condition：查询的列不完全被索引覆盖，where条件中是一个前导列的范围  

​				explain select * from film_actor where film_id > 1;  

Using where：使用 where 语句来处理结果，查询的列未被索引覆盖,字段上没有索引

​				explain select * from actor where name = 'a';  

Using temporary：mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化。  

Using filesort：将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的。  

## 最左前缀法则

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引
中的列  

## 索引的使用建议

1.不在索引列上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描  

2.存储引擎不能使用索引中范围条件右边的列  

3.尽量使用覆盖索引（只访问索引的查询（索引列包含查询列）），减少select *语句  

4.mysql在使用不等于（！=或者<>）的时候无法使用索引会导致全表扫描  

5.is null,is not null 也无法使用索引  

6.like以通配符开头（'$abc...'）mysql索引失效会变成全表扫描操作  

7.字符串不加单引号索引失效  

8.少用or或in，用它查询时，mysql不一定使用索引，mysql内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引，详见范围查询优化  

## 索引优化案例

### order by  和group by

一般order by 或者 group by都要使用所索引，避免文件排序

```mysql
explain select * from employees e where e.name = 'a' order by  age asc,position desc;
```

排序的字段于索引顺序一样，如果一个升序一个降序，那么也不能使用索引排序

```mysql
explain select * from employees e where e.name in( 'a','b') order by e.name;
```

对于排序来说，多个条件也是范围查询，不会使用索引排序	

![image-20200916192006718](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200916192006718.png)

### 优化总结

1.MySQL支持两种方式的排序filesort和index，Using index是指MySQL扫描索引本身完成排序。index效率高，filesort效率低  

2.order by满足两种情况会使用Using index

​	1) order by语句使用索引最左前列。  

​	2) 使用where子句与order by子句条件列组合满足索引最左前列。  

3.尽量在索引列上完成排序，遵循索引建立（索引创建的顺序）时的最左前缀法则。  

4.如果order by的条件不在索引列上，就会产生Using filesort。
5. 能用覆盖索引尽量用覆盖索引
6.	group by与order by很类似，其实质是先排序后分组，遵照索引创建顺序的最左前缀法则。对于group by的优	化如果不需要排序的可以加上**order by null禁止排序**。注意，where高于having，能写在where中
	的限定条件就不要去having限定了  



## mysl 常见优化

## 分页查询

### Join关联查询

mysql的表关联常见有两种算法  

​		Nested-Loop Join 算法	

​		Block Nested-Loop Join 算法

### 嵌套循环连接 Nested-Loop Join(NLJ) 算法  

一次一行循环地从第一张表（称为**驱动表**）中读取行，在这行数据中取到关联字段，根据关联字段在另一张表（**被驱动表**）里取出满足条件的行，然后取出两张表的结果合集。  

这里一般优化器会选择小表驱动大表。

如果EXtra中未出现Using join buffer 则表示使用的算法是NLJ

``` mysql
EXPLAIN select*from t1 inner join t2 on t1.a= t2.a;
```

上面sql的大致流程如下：

1. 从表 t2 中读取一行数据；
2.  从第 1 步的数据中，取出关联字段 a，到表 t1 中查找；
3.  取出表 t1 中满足条件的行，跟 t2 中获取到的结果合并，作为结果返回给客户端；
4.  重复上面 3 步。  

整个过程会读取 t2 表的所有数据(扫描100行)，然后遍历这每行数据中字段 a 的值，根据 t2 表中 a 的值索引扫描 t1 表中的对应行(**扫描100次 t1 表的索引，1次扫描可以认为最终只扫描 t1 表一行完整数据，也就是总共 t1 表也扫描了100行**)。因此整个过程扫描了 **200 行**。如果被驱动表的关联字段没索引，**使用NLJ算法性能会比较低(下面有详细解释)**，mysql会选择Block Nested-Loop Join算法。  

### 基于块的嵌套循环连接 Block Nested-Loop Join(BNL)算法  

把**驱动表**的数据读入到 join_buffer 中，然后扫描**被驱动表**，把**被驱动表**每一行取出来跟 join_buffer 中的数据做对比。  

Extra 中 的Using join buffer (Block Nested Loop)说明该关联查询使用的是 BNL 算法。  

上面sql的大致流程如下：

1. 把 t2 的所有数据放入到 join_buffer 中
2.  把表 t1 中每一行取出来，跟 join_buffer 中的数据做对比
3. 返回满足 join 条件的数据整个过程对表 t1 和 t2 都做了一次全表扫描，因此扫描的总行数为10000(表 t1 的数据总量) + 100(表 t2 的数据总量) =10100。并且 join_buffer 里的数据是无序的，因此对表 t1 中的每一行，都要做 100 次判断，所以内存中的判断次数是100 * 10000= 100 万次。  

### in和exsits优化  

in：当B表的数据集小于A表的数据集时，in优于exists  

```mysql
 select * from A where id in (select id from B)
 #等价于：
 for(select id from B){
	 select * from A where A.id = B.id
 }
```

exists:当A表的数据集小于B表的数据集时，exists优于in将主查询A的数据，放到子查询B中做条件验证，根据验证结果（true或false）来决定主查询的数据是否保留  

```mysql
 select * from A where exists (select 1 from B where B.id = A.id)
 #等价于:
 for(select * from A){
 select * from B where B.id = A.id
 }

```



## mysql中的锁

MyISAM在执行查询语句(SELECT)前,会自动给涉及的所有表加读锁,在执行增删改
操作前,会自动给涉及的表加写锁  	

总结：读锁会阻塞写，但是不会阻塞读。而写锁则会把读和写都阻塞  

InnoDB和Mysam的区别

事务和行锁

### 并发处理带来的问题

丢失更新 : 其他事务覆盖了当前事务的修改

脏读:读到了其他事务未提交的数据

幻读:读到了其他事务新增的数据

不可重复度:读到了其他事务已经提交的数据

![image-20200916201817668](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200916201817668.png)

### 行锁

一个session开启事务更新不提交，另一个session更新同一条记录会阻塞，更新不同记录不会阻塞  

## MVCC机制

select 操作不会更新版本号，是快照读，insert update delete 会更新版本号，是当前读

![image-20200916203356981](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200916203356981.png)

![image-20200916203411913](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200916203411913.png)

## 间隙锁

可以解决幻读

要避免幻读可以用间隙锁在Session_1下面执行update account set name ='zhuge' where id > 10 and id <=20;，则其他Session没法在这个范围所包含的间隙里插入或修改任何数据  

行锁是针对索引的，所以没有索引的话会锁住整张表