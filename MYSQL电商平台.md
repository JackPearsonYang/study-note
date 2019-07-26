# MYSQL电商数据库实战





![1562730982420](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562730982420.png)







# 数据库设计规范



### 数据库命名规范



**1.对象名称必须使用小写字母并用下划线分割**



不同的数据库名：DBName dbname

不同的表名： Table table tabLe



2. **对象名称禁止使用MYSQL保留关键字**



错例：select id，username，from，age from tb_user



**3.对象命名需见名识义**



**4.所有临时表需以tmp为前缀并以日期为后缀**



**5.备份库，备份表必须以bak为前缀并以日期为后缀**



**6.所有存储相同数据的列名和列类型必须一致**（关联列的类型相同对数据库的查询性能十分重要，避免不必要的类型转换）



![1562731515056](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562731515056.png)







数据库基本设计规范



- UTF-8字符集
- 使用InnoDB存储引擎
- 尽量控制单表数据量大小——建议控制在500万条内
  - 历史数据归档（常见于日志表）
  - 分库分表（业务数据）
- 尽量做到冷热数据分离，利用更有效的缓存，避免读入无用的冷数据
- **禁止**在表中建立**预留字段**
  - 对预留字段类型的修改，会对表进行锁定，影响并发性
- **禁止**在数据库中存储图片，文件等**二进制数据**
- **禁止**在线上做数据库压力测试





### 数据库索引设计规范



索引对数据库的查询性能来说是非常重要的



- 建议单张表索引不超过5个，限制每张表上的索引数量

  - 索引的建立降低插入，删除的效率
  - 索引过多会影响执行计划进度

- 每个Innodb表有一个主键

  - 由于每个表根据主键索引来进行排序，故**不推荐**使用UUID,MD5,HASH，字符串等作为主键，因为它们**不保证顺序增长**，若插入新主键值很小，所有大于该值的行都需在索引中往后移。

  

**常见索引列建议**



- SELECT、UPDATE、DELETE语句的WHERE从句中的列
- 包含在ORDER BY、GROUP BY中的字段
- 多表JOIN的关联列



**如何选择索引列的顺序**



- 区分度最高的列放在联合索引的最左侧（主键区分度最高）
- 尽量把字段长度小的列放在联合索引的最左侧
- 频率的列放左侧



**对于频率的查询优先考虑使用覆盖索引（所有查询列均设置索引）（如查询商品信息的业务）**







### 数据库字段设计规范



**优先选择符合存储需要的最小的数据类型**



1. 将字符串转化为数字类型存储

   

   INET_ATON('255.255.255.255')=42.....5   

   - 16字节》》4字节

   

   真正需要用到IP地址时

   INET_NTOA('42.....5')=’255.255.255.255‘

   

2.  非负整数使用无符号数来存储，其存储正数的范围比有符号数大一倍

**上例中用整数存储IP地址就需要用unsigned int**



3.  MYSQL 中，VARCHAR(N)中的N代表的是**字符数**，而不是字节数

   varchar(255)可保存255个中文



![1562813619833](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562813619833.png)



4. **尽可能把所有列定义为NOT NULL**
   1. 索引NULL列需要额外的空间来保存，占用更多的空间
   2. 进行比较和计算时要对NULL值做特别的处理
5. 不要用字符串存储日期型的数据，使用DATETIME或TIMESTAMP
6. 精准的浮点类型decimal，同财务相关的金额类数据，必须使用它
   1. Decimal类型为精准浮点数，在计算时不会丢失精度
   2. 可用于存储比bigint更大的整形数据





### 数据库SQL开发规范



- 建议使用预编译语句进行数据库操作



![1562814124544](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562814124544.png)



- 避免数据类型的隐式转换



eg. 下列中将id参数传递为字符串，应该直接传递整型

```sql
SELECT name,phone from customer where id='111'
```



- 充分利用表上已经存在的索引
  - 避免使用双%号的模糊查询条件，如LIKE “%123%”，**若只有后边的%，查询实际上是可以利用到索引信息的，前缀索引**
  - **使用left join或not exists来优化not in 操作？？？？**



- 程序连接不同数据库使用不同的账号，**禁止**跨库查询



- 下例减少表的变更对已有数据的影响

![1562814744437](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562814744437.png)





![1562814819546](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562814819546.png)



- 需要进行随机排序时，**禁止**使用order by rand()进行随机排序
  - 会把表中符合条件的数据装载到内存中进行排序
  - 推荐在程序中获取一个随机值，然后从数据库中获取数据？？？？



![1562816115556](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562816115556.png)





- 如：合并多种类型的订单数据，由于订单属于不同类型，不会出现重复，故使用UNION ALL

![1562816150492](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562816150492.png)







### 数据库操作行为规范



- 超100万行的批量写操作，要分多批次进行
  - 主从延迟
  - 产生大事务操作



- 大表的结构修改

  

![1562816421393](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562816421393.png)









# 用户模块设计



### 管理和维护用户信息



用户实体：姓名，登录名，密码，手机号，证件类型，邮箱

积分，注册时间，用户状态，用户级别，用户余额



**若将会员级别均放在用户表中，当做修改时，由于数据出现冗余，可能需要修改多行数据**



# 

![1562835355420](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562835355420.png)



![1562835461289](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562835461289.png)





- 设计数据库时最好应满足第三范式（3NF )的要求，而不是所有字段均存储在一张表中



![1562835570766](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562835570766.png)





#### 第三范式



**一个表中的列和其他列之间既不包含部分函数依赖关系，也不包含传递函数依赖关系，那么这个表的设计就符合第三范式**



#### 回顾之前的例子：



![1562835691995](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562835691995.png)



下面的三个字段之间存在一定的函数依赖关系



- {登录名}《《{用户级别}《《{级别积分上限，级别积分下限}





#### 修改：拆分原用户表以符合第三范式



![1562835881060](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562835881060.png)





![1562835976558](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562835976558.png)



**用户登录表：**

```sql
CREATE TABLE customer_login(
customer_id int unsigned AUTO_INCREMENT NOT NULL,

modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    
    #ON UPDATE中MYSQL支持每次更新某行时，modified_time记录更新时间
)
```



**用户信息表：**



```sql
CREATE TABLE customer_inf(
	customer_inf_id int unsigned AUTO_INCREMENT NOT NULL comment '自增主键ID'
    customer_id int unsigned  NOT NULL comment 'customer_login表的自增ID, 为登录表的外键，类名与数据类型与登录表完全一致',

modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    
    #ON UPDATE中MYSQL支持每次更新某行时，modified_time记录更新时间
)
```



**用户级别表：**



```sql
CREATE TABLE customer_level_inf(
	customer_level tinyint NOT NULL AUTO_INCREMENT  comment '会员级别ID'
    level_name varchar(10)  NOT NULL comment '会员级别名称',
    min_point int unsigned,
    max_point int unsigned,

modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    primary key pk_levlid(customer_level)
    #ON UPDATE中MYSQL支持每次更新某行时，modified_time记录更新时间
)
```





**用户地址表**



```sql
CREATE TABLE customer_addr(
	customer_addr_id int unsigned NOT NULL AUTO_INCREMENT  comment '自增主键ID'
    customer_id int unsigned NOT NULL comment 'customer_login表的自增id',
    zip smallint not null,
    province smallint not null comment '地区表中的省份id',
    city smallint not null comment '地区表中的城市id',
    address varchar(200) not null comment '具体的地址门牌号'
    is_default tinyint not null comment '是否默认'
modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    primary key pk_customeraddrid(customer_addr_id)
    #ON UPDATE中MYSQL支持每次更新某行时，modified_time记录更新时间
)
```





**用户积分日志表**



```sql
CREATE TABLE customer_pont_log(
	point_id int unsigned NOT NULL AUTO_INCREMENT  comment '积分日志ID'
    customer_id int unsigned NOT NULL comment 'customer_login表的自增id，记录哪个用户的积分变动日志',
    source smallint unsigned not null comment '积分来源：0 订单 1 登录 2 活动',
    change_point smallint not null default 0 comment '变更积分数',
	create_time timestamp NOT NULL COMMENT '日志生成时间',
    primary key pk_pointid(point_id)
    
)
```



**用户余额变动表**



```sql
CREATE TABLE customer_balance_log(
	balance_id int unsigned NOT NULL AUTO_INCREMENT  comment '余额日志ID'
    customer_id int unsigned NOT NULL comment 'customer_login表的自增id，记录哪个用户的余额变动日志',
    source smallint unsigned not null comment '记录来源：1 订单 2 退货单',
    source_sn int unsigned not null comment '相关单据id',
	create_time timestamp NOT NULL default current_timestamp COMMENT '日志生成时间',
    
    #涉及到财务的数据，需要用decimal类型表示
    amount decimal(8,2) not null default 0.00 comment '变动金额'
    
    primary key pk_balanceid(balance_id)
    
)
```





**用户登录日志表**



```sql
CREATE TABLE customer_login_log(
	login_id int unsigned NOT NULL AUTO_INCREMENT  comment '登录日志ID'
    customer_id int unsigned NOT NULL comment 'customer_login表的自增id，记录哪个用户的登录日志',
    login_time timestamp NOT NULL COMMENT '登录时间',
    login_ip int unsigned NOT NULL COMMENT '登录IP',
    login_type tinyint NOT NULL COMMENT '登录类型：0未成功 1 成功',
    
    primary key pk_loginid(login_id)
    
)
```





### MYSQL分区表的介绍



#### **MySQL分区表很适合用来存储用户日志表**



**确认MySQL服务器是否支持分区表**



```sql
SHOW PLUGINS;
```



![1562908143678](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562908143678.png)





#### MYSQL分区表的特点



**在逻辑上为一个表，在物理上存储在多个文件中**



```sql
CREATE TABLE customer_login_log(
	login_id int unsigned NOT NULL AUTO_INCREMENT  comment '登录日志ID'
    customer_id int unsigned NOT NULL comment 'customer_login表的自增id，记录哪个用户的登录日志',
    login_time timestamp NOT NULL COMMENT '登录时间',
    login_ip int unsigned NOT NULL COMMENT '登录IP',
    login_type tinyint NOT NULL COMMENT '登录类型：0未成功 1 成功',
    
    primary key pk_loginid(login_id)   
)

#多加两行语句

PARTITION BY HASH(customer_id)
	PARTITIONS 4;

```





```shell
#非分区表物理文件：
customer_login_log.frm
customer_login_log.ibd 

#分区表物理文件：
customer_login_log.frm
customer_login_log#P#p0.ibd
customer_login_log#P#p1.ibd 
customer_login_log#P#p2.ibd 
customer_login_log#P#p3.ibd 
```



#### 按HASH分区



若选择非整形字段作为HASH分区依据，可先转为整型再用HASH函数处理



如：

```sql
PARTITION BY HASH(UNIX_TIMESTAMP(login_time))
```





#### 按范围分区（RANGE）



**RANGE分区特点**



- 根据分区**键值**的范围把数据行存储到表中的不同分区中
- 多个分区的范围要连续，但不能重叠
- 默认情况下使用 VALUES LESS THAN 属性（<），即每个分区不包括指点的那个值



```sql
PARTITION BY RANGE(customer_id)(
	PARTITION p0 VALUES LESS THAN(10000),
    PARTITION p1 VALUES LESS THAN(20000),
    PARTITION p2 VALUES LESS THAN(30000),
    PARTITION p3 VALUES LESS THAN MAXVALUE,
);
```



**适应场景：**



- 分区键为日志或是时间类型
- 所有查询中都包括分区键，能很大程度上避免跨分区扫描
- 定期按分区范围清理历史数据





#### LIST分区



LIST分区的特点

​	按分区键取值的列表进行分区

​	通范围分区一样，各分区列表值不能重复



```sql
PARTITION BY LIST(login_type)(
	PARTITION p0 VALUES in(1,3,5,7,9),
    PARTITION p1 VALUES in(2,4,6,8)
);
```





#### 如何为Customer_login_log表分区



**业务场景：记录每次用户登录的日志，用户登录日志保存一年，一年后可删除**



**适合使用RANGE分区**



**以login_time作为分区键**，以年为单位

```sql
PARTITION BY RANGE(YEAR(login_time))(
	PARTITION p0 VALUES LESS THAN(2015),
    PARTITION p1 VALUES LESS THAN(2016),
    PARTITION p2 VALUES LESS THAN(2017),
);
```



**每年的固定任务，增加分区：**

```sql
ALTER TABLE customer_login_log ADD PARTITION(PARTITION p3 VALUES LESS THAN(2018)); 
```



**删除老的分区**

```sql
ALTER TABLE customer_login_log DROP PARTITION p0;
```



这种分区表删除方法比一般的DELETE WHERE方法更简易，快速。



**归档**

创建归档表——表结构与日志表相同：

```sql
CREATE TABLE arch_customer_login_log(
	login_id int unsigned NOT NULL AUTO_INCREMENT  comment '登录日志ID'
    customer_id int unsigned NOT NULL comment 'customer_login表的自增id，记录哪个用户的登录日志',
    login_time timestamp NOT NULL COMMENT '登录时间',
    login_ip int unsigned NOT NULL COMMENT '登录IP',
    login_type tinyint NOT NULL COMMENT '登录类型：0未成功 1 成功',
    
    primary key pk_loginid(login_id)   
)
```



**MYSQL > 5.7**

分区迁移：**EXCHANGE  WITH** 

```sql
ALTER TABLE customer_login_log exchange partition p2 WITH TABLE
arch_customer_login_log;
```



**将归档表改为归档引擎：**

- 只能进行查询操作，但占存储空间更小

```sql
ALTER TABLE arch_customer_login_log ENGINE=ARCHIVE
```







### 商品实体设计





![1562922098627](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1562922098627.png)



#### 品牌信息表（brand_info）



```sql

CREATE TABLE brand_info(
	brand_id SMALLINT NOT NULL AUTO_INCREMENT  comment '品牌ID'
    brand_name varchar(50) NOT NULL comment '品牌名称',
    telephone varchar(50) NOT NULL comment '联系电话',
    brand_web varchar(100) NOT NULL comment '品牌网站',
    brand_logo varchar(100) NOT NULL comment '品牌logoURL',
    brand_desc varchar(50) NOT NULL comment '品牌描述',
    brand_staus tinyint not null default 1,
    modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    primary key pk_brandid(brand_id)   
)
```



 

#### 分类信息表（product_category）



```sql
CREATE TABLE brand_info(
	brand_id SMALLINT NOT NULL AUTO_INCREMENT  comment '品牌ID'
    brand_name varchar(50) NOT NULL comment '品牌名称',
    telephone varchar(50) NOT NULL comment '联系电话',
    brand_web varchar(100) NOT NULL comment '品牌网站',
    brand_logo varchar(100) NOT NULL comment '品牌logoURL',
    brand_desc varchar(50) NOT NULL comment '品牌描述',
    brand_staus tinyint not null default 1,
    modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    primary key pk_brandid(brand_id)   
)

```





#### 供应商信息表（supplier_info）



```sql
CREATE TABLE supplier_info(
	supplier_id int unsigned NOT NULL AUTO_INCREMENT  comment '供应商ID'
    supplier_code char(8) NOT NULL comment '供应商品牌',
    supplier_name char(50) NOT NULL comment '供应商名称',
    supplier_type tinyint NOT NULL comment '供应商类型：1.自营，2.平台',
    link_man varchar(10) NOT NULL comment '联系电话',
    bank_name varchar(50) NOT NULL comment '开户银行名称',
    address varchar(200) not null comment '供应商地址',
    modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 
    ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    primary key pk_supplierid(supplier_id)   
)
```





#### 商品信息表（product_info)



```sql
CREATE TABLE product_info(
	product_id int unsigned AUTO_INCREMENT not null comment '商品ID'，
    product_code char(16) not null comment '商品编码',d
)
```





### 如何对评论进行分页展示



```sql

EXPLAIN
SELECT customer_id,title,content
	FROM 'product_comment'
	WHERE audit_status=1
	AND product_id=199726
	LIMIT 0,5;
```





#### 分析SQL执行计划



![1563005175627](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563005175627.png)





![1563007367585](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563007367585.png)



![1563007513445](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563007513445.png)





![1563007603672](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563007603672.png)







#### TYPE列：执行计划所使用的类型性能



![1563008074952](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563008074952.png)





若在执行计划中TPYE列出现了**ALL**字段，说明出现了效率差的全表扫描，需要看是否能建立合适的索引。





#### Extra列：额外信息列





![1563008309584](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563008309584.png)



![1563008342300](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563008342300.png)





#### POSSIBLE_KEYS列



指出MySQL能使用哪些索引来优化查询，涉及到的查询列上的索引都会被列出，但不一定使用



#### KEY列



- 查询优化器优化查询时实际所使用的索引
- 如果没有可用的索引，则显示为NULL
- 如查询使用了覆盖索引，则该索引仅出现在Key列中





#### Ref列



表示那些列或常量被用于查找索引列上的值



#### rows列



表示MySQL通过索引统计信息，**估算的所需读取的行数**





#### Filtered列：可理解为行命中率



![1563008918090](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563008918090.png)





#### 当前版本执行计划的一些缺憾



无法展示存储过程，触发器，UDF对查询的影响





### 优化评论分页查询





```sql
EXPLAIN
SELECT customer_id,title,content
	FROM 'product_comment'
	WHERE audit_status=1
	AND product_id=199726
	LIMIT 0,5;
```



在上述查询的执行计划中显示，查询没有利用到索引，故考虑建立

**audit_status与product_id的联合索引**



建立联合索引前，需考虑将**区分度更高的字段放在联合索引的前面**，根据以下SQL语句估算区分度，结果表明值接近于1的product_rate区分度更高。



```sql
SELECT COUNT(DISTINCT audit_status)/COUNT(*) AS audit_rate,
	COUNT(DISTINCT product_id)/COUNT(*) AS product_rate
	FROM product_comment;
```



```sql
create index idx_productID_auditStatus on product_comment(product_id,audit_status)
```





#### 进一步的优化



**并不是很能理解下面这种SQL执行方式为何更优？？？？**

查询不会随着页数增大而开销增大？？？？



![1563009912567](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563009912567.png)









### 如何删除重复数据



删除评论表中对同一订单同一商品的重复评论，**只保留最早的一条**



1. 查看是否存在重复评论
2. 备份product_comment表
3. 删除同一订单的重复评论





#### 查看是否有重复评论



```sql
	SELECT order_id,product_id,COUNT(*)
    FROM product_comment
    GROUP BY order_id,prodct_id
    HAVING COUNT(*)>=2
```







#### 备份评论表



```sql
CREATE TABLE bak_product_comment_190716
LIKE
product_comment

INSERT INTO bak_product_comment_190716
SELECT * FROM product_comment;
```





#### 删除重复评论



```sql
DELETE a
FROM product_comment a
JOIN(
	SELECT order_id,product_id,MIN(comment_id) AS comment_id
    FROM product_comment
    GROUP BY order_id,prodct_id
    HAVING COUNT(*)>=2
) b ON a.order_id=b.order_id AND a.product_id= b.product_id
AND a.comment_id>b.comment_id
```





### 如何进行分区间统计



```sql
SELECT
COUNT(CASE WHEN IFNULL(total_money,0)>=1000) THEN a.customer_id END) 
AS '大于1000',
COUNT(CASE WHEN IFNULL(total_money,0)<1000) THEN a.customer_id END) 
AS '小于1000'
FROM mc_userdb.'customer_login' a
LEFT JOIN
(
	SELECT customer_id,SUM(order_money) AS total_money
    FROM mc_orderdb.'order_master'
    GROUP BY customer_id
) b
ON a.customer_id=b.customer_id
```





###  捕获有问题的SQL



如何找到需要优化的SQL呢？



**答案：慢查询 日志**





```shell
set global slow_query_log_file=/sql_log/slow_log.log;

set global long_query_time=0.001;
#抓取执行时间超过0.001秒的SQL

set global low_query_log=on;
#启动慢查询日志
```





### 慢查询日志记录的内容





![1563261833878](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563261833878.png)





- query_time 查询时间
- lock_time 锁定时间
- rows_sent 返回行数，返回0
- rows_examined 扫描行数，扫描10000行，返回0说明没有正确使用索引







#### MySQL提供的分析慢查询日志的命令行工具



**会将重复的慢查询合并：**



```shell
mysqldumpslow slow-mysql.log
```







## 数据库备份



 

#### 逻辑备份与物理备份



逻辑备份的结果为SQL语句，适合于所有存储引擎。



物理备份是对数据库目录的拷贝，对于内存表（Memory存储引擎）只备份结构。





#### 全量备份和增量备份



全量备份是对整个数据库的完整备份



增量备份是在上次全量或增量备份基础上，对于更改数据进行的备份

（**XtraBackUp支持增量备份，较mysqldump更安全高效**）

  

#### 热备工具：XtraBackUp





#### 使用Mysqldump备份



常用语法



mysqldump [OPTIONS] database [tables]







#### 备份保证数据一致性



--single-transaction  对支持事务的InnoDB存储引擎有效

--lock-tables        不支持事务的MyIsam使用时，每次仅加锁一个表，不能保证整个数据库的数据一致性。

--lock-all-tables  加锁所有表

--master-data=[1/2]



![1563518079419](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563518079419.png)





--R,--routines   备份数据库中存在的存储过程

--triggers    备份数据库中存在的触发器

-E， --events   备份数据库中存在的调度事件





--hex-blob   

--tab=path     

-w,  --where='过滤条件'  where支持单表导出





####  创建用户



```sql
create user 'backup'@'localhost' identified by '123456';

#创建仅在本地可登录的backup账号，密码是123456

#接着授权

grant select,reload,lock tables,replication client,show view,event,process on *.* to 'backup'@'localhost'
```



```shell
cd /data
cd db_backup
mysqldump -ubackup -p --master-data=2 --single-transaction
--routines --triggers --events mc_orderdb > mc_orderdb.sql


```



#### 验证以上命令完成了全备



```shell
grep "CREATE TABLE" mc_orderdb.sql
#查找mc_orderdb.sql中所有包含CREATE TABLE的字符串
```



![1563518961312](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563518961312.png)







#### 备份order_master单表



```shell
mysqldump -ubackup -p --master-data=2 --single-transaction
--routines --triggers --events mc_orderdb order_master > order_master.sql

```





#### 备份mysql实例下的所有数据库





```shell
mysqldump -ubackup -p --master-data=2 --single-transaction
--routines --triggers --events --all-databases > mc.sql

```





#### 赋file权限给backup用户并用-tab指定备份目录





**使用tab参数备份就在一个文件夹内生成多个SQL文件与TXT文件，SQL文件仅存储表结构，而TXT文件存储表数据。**





```sql
grant file on *.* to 'backup'@'localhost'

#将/tmp/mc_orderdb目录修改写权限
chown mysql:mysql mc_orderdb/

mysqldump -ubackup -p --master-data=2 --single-transaction
--routines --triggers --events --tab="/tmp/mc_orderdb" mc_orderdb > mc_orderdb.sql
```







### 从SQL文件恢复数据库



![1563519964821](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1563519964821.png)



```shell
mysql -uroot -p -e"create database bak_orderdb"

mysql -uroot -p bak_orderdb < mc_orderdb.sql
```



**建立订单表备份数据库后，可将备份数据库与主数据库进行左连接，查找在主数据中不存在而在备份数据库中存在的误删除行，并插入主数据库进行恢复**







### 从-tab参数生成的结果集恢复





可仅恢复单表



```sql
mysql>  source /tmp/mc_orderd/region_info.sql;

mysql>  load data infile '/tmp/mc_orderdb/region_info.txt' into table region_info;
```





### 如何指定时间点的恢复点



进行某一时间点的恢复

​	

**先决条件**

​	**具有指定时间点前的一个全备**



