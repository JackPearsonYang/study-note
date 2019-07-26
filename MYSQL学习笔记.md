# MYSQL学习笔记



## 一：操作数据库

​	**{} ：必备项**

​	**[] ：可选项**



### 1.创建数据库



CREATE {DATABASE | SCHEMA } [ IF NOT EXISTS ] db_name

[DEFAULT]  CHARACTER SET [=] charset_name(编码方式)



#### 查看警告信息: SHOW WARNINGS

#### 查看数据库创建指令 SHOW CREATE DATABASE table-name

#### 修改已有数据库编码方式：

​	ALTER {DATABASE | SCHEMA } [db_name] 

[DEFAULT] CHRACTER SET [=] charset_name





### 2.创建数据表

 	数据表是数据库最重要的内容之一

​	命令行打开数据库 USE table_name



CREATE TABLE  [ IF NOT EXISTS ] db_name(

column_name  data_type,

...

)

### 



### 3.查看数据表列表,结构



  SHOW TABLES [FROM db_name]  

[LIKE 'pattern'| WHERE expr]



可以利用FROM db_name查看非当前数据库的表



SHOW COLUMNS FROM tbl_name

查看数据表结构



### 4.数据表行操作



#### 插入记录

INSERT [INTO] tbl_name [(col_name,..)] VALUES(val,..)

[INTO]和[(col_name,..)]均可省略，但若列名省略，需要给所有字段赋值



#### 记录查找命令

SELECT expr, ... FROM tbl_name



#### 设置数据属性，跟在数据类型后面(例：TINYINT NOT NULL)

1. 非空  NOT NULL
2. 自动编号,必须与主键组合使用 id SMALLINT AUTO_INCREMENT PRIMARY KEY
3. 自动编号注：由于自动编号不用赋值，故而插入记录时列名不能省略





### 5.MYSQL数据库约束

- 约束分为表级约束和列级约束
- 约束保证数据的完整性和一致性
- 约束类型包括（非空 主键 唯一 默认 外键）





#### 非主键的唯一约束

UNIQUE KEY  这种唯一约束数据表可以存在多个



#### 默认约束 DEFAULT

当插入数据时，没有明确赋值则自动赋值

语法：sex ENUM('1','2','3') DEFAULT '3'



#### 外键约束  FOREIGN KEY

1. 父表和子表必须使用相同存储引擎
2. 存储引擎只能为InnoDB
3. 外键列和参照列必须具有相似的数据类型。若为数字，则长度和符号为均需相同，字符长度可以不同
4. 外键列和参照列必须创建索引。若不存在，MYSQL自动创建索引



##### 外键约束实践：

​	创建省份表province 包含 主键ID和省份名

​	创建用户字段，包含用户ID，用户名，和所在省份

 CREATE TABLE users(

id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT

username VARCHAR(20) NOT NULL

pid BIGINT,

FOREIGN KEY (pid) REFERENCE province (id)

)



**FOREIGN KEY (pid) REFERENCE province (id) 表示给当前表的pid字段设置外键约束，参照province表中的ID字段。但若pid与父表中的ID字段数据类型不同，比如BIGINT与INITINT就会导致错误。**



**参照列为province中的ID字段，它为省份表中的主键值，主键值创立时会自动创建索引，满足外键约束的第四个要求 。**

**可根据命令SHOW INDEXS FROM province来验证**



##### 外键约束的参照操作

若将上述外键约束代码改为

**FOREIGN KEY (pid) REFERENCE province (id) ON DELETE CASCADE**

当父表删除操作执行时，子表中记录同时被删除DELETE FROM province WHERE id=3





1. **CASCADE:从父表删除或更新的同时自动删除或更新子表中匹配行**
2. **SET NULL:从父表删除或更新行，并设置子表中的外键列为NULL。但若使用该选项，子表中的被约束列必须没有指定 NOT NULL**
3. RESTRICT:拒绝对父表的删除或更新
4. NO ACTION:标准SQL关键字，与3效果相同





### 6.修改数据表



#### 单列的添加

ALTER TABLE tbl_name ADD [COLUMN] col_name

column_definition [FIRST | AFTER col_name]\

1. FIRST 表示添加列放在第一列
2. AFTER col_name 表示新增列放在col_name后面
3. 默认为新增列为最后一列



#### 单列的删除

ALTER TABLE tbl_name DROP col_name,DROP col_name2



#### 约束的添加

1. 主键约束 	

   ALTER TABLE tbl_name ADD [CONSTRAINT [symbol (约束名字)] ]  PRIMARY KEY   [index_type(创建的索引类型)] (index_col_name,..)

2. 唯一约束

   ALTER TABLE tbl_name ADD [CONSTRAINT [symbol (约束名字)] ]  UNIQUE [INDEX |KEY ]

   [index_name]

   [index_type]  (index_col_name,...)

3. 外键约束

   ALTER TABLE tbl_name ADD [CONSTRAINT [symbol (约束名字)] ]  FOREIGN KEY (pid) REFERENCE province(id)



#### 约束的删除

1. 主键约束删除

   ALTER TABLE tbl_name  DROP PRIMARY KEY

2. 唯一约束删除

   ALTER TABLE tbl_name  DROP [INDEX | KEY ] index_name

   需要首先知道对应的唯一约束的索引名，故而有命令

   SHOW INDEXES FROM tbl_name,找到KEY_name所对应的值，放进ALTER语句的index_name中

3. 删除外键约束

   ALTER TABLE tbl_name  DROP  FOREIGN KEY constraint_name

   外键约束名可通过SHOW CREATE tbl_name命令来查看



#### 单列的修改

**列定义的修改MODIFY**

ALTER TABLE tbl_name MODIFY id SMALLINT UNSIGNED NOT NULL FIRST

数据类型由大类型改为小类型可能会造成数据的丢失



**列名称的修改CHANGE**

ALTER TABLE tbl_name  CHANGE old_name  new_col_name  col_defintion



### 6.单表更新记录

UPDATE [LOW_PRIORITY]  

[IGNORE] table_reference SET	col_name={expr|DEFAULT},col_name=...  [WHERE ]



E.P： UPDATE users set age=age +5;  //所有年龄加5



同时更新多列所有行 ：  UPDATE users SET age=age-id, sex=0;



所有ID号为偶数的用户年龄加十岁：UPDATE users SET age=age+10  WHERE id%2==0





### 7.单表删除操作



DELETE FROM tbl_name  [WHERE where_condition]



DELETE FROM users WHERE id=6；



### 8.单表查找操作



SELECT select_expr [,select_expr]

[

​	FROM table_reference 

​	[WHERE where_condition]

​	[GROUP BY]

​	[HAVING]

​	[ORDER BY]

​	[LIMIT]

]



**SELECT id,username FROM users**    

**以上语句仅选取表中的两列**



在使用多表连接中的查询操作时，可能需要用到users.id,users.username或者users.*



#### 可以用AS为长字段名赋别名alias

SELECT id AS userId,username AS uname



#### 分组GROUP BY

[GROUP BY {col_name|position} ]  



SELECT sex FROM users GROUP BY  sex  用性别来分组



#### HAVING  分组条件  

[HAVING where having_condition]



1. 用HAVING组件需满足下列两个条件之一
2. where_condtion条件为聚合函数count,max,average等
3. 或者where_condition中的字段出现在SELECT后，如

SELECT sex,age FROM users GROUP BY sex HAVING age>30

对年龄大于30的用户进行分组，分为”男“ ”女“性别两组。



#### ORDER BY 查询结果排序

[ORDER BY {col_name|expr|position}]

[DESC | ASC]



多字段排序，位于前面的字段排序优先级更高 

SELECT * FROM users ORDER BY age,id DESC



#### LIMIT 限制查询的显示条数

[LIMIT {[offset,] row_count |rowcount OFFSET offset}]

常用第一种语法结构



LIMIT 2  仅返回两条记录

LIMIT 2，2  返回第三**第四条记录**

**offset 为起始偏移量。**



以上顺序为：HAVING		ORDER		LIMIT



### 9.利用有子查询的插入语句迁移数据表中的数据

INSERT test（新表） SELECT id,username FROM users WHRE age>=30；



利用SELECT子查询将users表中年龄大于30的用户写入到新表test中





### 10.子查询与连接



#### 子查询

子查询是指出现在其他SQL语句内的SELECT子句



**子查询指嵌套在查询内部，且必须始终出现在圆括号内******



**子查询外层查询可以是：SELECT,INSERT,UPDATE,SET或DO**



**子查询可以返回标量、一行、一列或子查询**



#### 一：使用比较运算符的子查询

SELECT AVG(goods_price) FROM tdb_goods

聚合函数AVG,MAX,COUNT等仅返回一个值，上例求所有商品的平均值

ROUND（AVG（goods_price),2)，对平均值四舍五人并保存两位小数值



查询商品价格**>**平均价格的商品

SELECT goods_id,goods_name，goods_price FROM tdb_goods WHERE goods_price>

（SELECT AVG(goods_price) FROM tdb_goods)



**当子查询返回了多行时，比如返回了多个笔记本的价格，可以使用ANY、SOME或ALL修饰的比较运算符来使SQL语句正常运行。**



》ANY（SELECT 子查询）   大于子查询返回值中的任意一个即可，即大于最小值

》ALL（SELECT 子查询） 大于子查询返回值中的所有值，即大于最大值





#### 二：使用[NOT] IN的子查询



=ANY 运算符 与IN等效

！=ALL或<>ALL运算符与NOT IN 等效

####  



#### 三：使用[NOT] EXISTS的子查询，使用较少





**应用场景：随着数据量的不断加大，前期数据库设立的商品品牌，商品种类如戴尔|笔记本等均为汉字，它们的数据量比小整数大，导致查询效率低。这是可以创建一张新的品牌名表，建立外键约束。但要在品牌名表中一条一条插入品牌数据是极耗时且易错的，这时可利用子查询放在INSERT语句中。**

SELECT goods_cate FROM tdb_goods GROUP BY goods_cate; 









## 二:MySQL事务部分





set global slow_query_log=ON 开启慢查询日志，set long_query_time=0 将0秒以上的查询均定义为长查询，



```sql
mysql> show variables like 'slow%';
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_launch_time    | 2                                    |
| slow_query_log      | ON                                   |
| slow_query_log_file | /var/lib/mysql/3247d14dbbc9-slow.log |
+---------------------+--------------------------------------+

--slow_qery_log_file 指向慢查询日志的位置
```



```sql
set global slow_query_log=ON;
set long_query_time=0;
select * from t where a between 10000 and 20000; /*Q1*/
select * from t force index(a) where a between 10000 and 20000;/*Q2*/

```



在没有长事务的干扰的情况下，查询语句执行日志如下图所示，扫描行数10000行

```sql
# Time: 2019-03-08T02:45:30.188319Z
# User@Host: root[root] @  [172.16.131.60]  Id:   882
# Query_time: 0.017316  Lock_time: 0.000058 Rows_sent: 10001  Rows_examined: 10001
SET timestamp=1552013130;

```



**对于由于索引统计信息不准确导致的问题，你可以用analyze table来解决。**

**而对于其他优化器误判的情况，你可以在应用端用force index来强行指定索引，也可以通过修改语句来引导优化器，还可以通过增加或者删除索引来绕过这个问题。**