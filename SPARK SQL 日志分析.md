# SPARK SQL 日志分析



## 环境配置



- Linux  CentOS(6.4)
- Hadoop CDH(hadoop-2.6.0-cdh5.7.0)
- Hive CDH(hive-1.1.0-cdh5.7.0)
- Scala : 2.11.8
- Spark : spark-2.1.0



生产环境中用CDH版本时，cdh版本应该一致





开发工具：

​	IDEA（本课程的选择）



![hadoop4](C:\Users\Administrator\Desktop\项目知识\hadoop4.PNG)



![hadoop1](C:\Users\Administrator\Desktop\项目知识\hadoop1.PNG)![hadoop3](C:\Users\Administrator\Desktop\项目知识\hadoop3.PNG)





## 分布式集群搭建



hadoop000:  IP地址1

hadoop001:  IP地址2

hadoop002:  IP地址3



hostname设置： sudo vi /etc/sysconfig/network 

NETWORKING=YES

HOSTNAME=hadoop001



hostname和IP地址的设置 : sudo vi/etc/hosts

IP地址1  hadoop000

IP地址2  hadoop001

IP地址3  hadoop002



各节点角色分配：



hadoop000: NameNode/DataNode  ResourceManager/NodeMangeer

hadoop001: DataNode NodeManager

hadoop002: DataNode NodeManager





## 前置配置



- SSH免密码登录

在每台机器上运行: ssh-keygen -t rsa

以hadoop000机器为主



ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop000

ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop001

ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop002



测试ssh：

1. 

.ssh文件夹中将生成authorized_keys文件



打开三台机器分别看看cat authorized_keys，结尾都应该是

****************=admin@hadoop000



2. 在Hadoop000上测试 ssh hadoop001  ssh hadoop002



- JDK下载安装



修改环境变量:

vi ~/.bash_profile,加入JAVA_HOME 以及 PATH

环境变量修改生效：

source ~/.bash_profile



- Hadoop集群环境配置分发



在hadoop000机器上解压Hadoop安装包，并设置HADOOP_HOME



在Hadoop根目录下的etc/hadoop文件夹中包含所有配置相关文件



1. 修改  vi  hadoop-env.sh   

   将java环境放入其中

   export JAVA_HOME=/home/admin/app/jdk1.8.0_191

2. 修改 core-site.xml

   ```
   <property>
   <name>fs.defaultFS</name>
   <value>hdfs://hadoop000:8020</value>
   </property>
   
   <property>
   <name>hadoop.tmp.dir</name>
   <value>/home/admin/app/tmp</value>
   </property>
   
   
   <!--指定临时文件存放地址 -->
   ```

3. 修改hdfs-site.xml,手工创建创房DataNode，namenode数据的tmp目录

```

<configuration>
                <property>
                <name>dfs.replication</name>
                <value>1</value>
            </property>

               
</configuration>


```

若是集群模式，hdfs-site中还需配置两组属性

```
<property>
<name>dfs.namenode.name.dir</name>
<value>/home/admin/app/tmp/dfs/name</value>
</property>

<property>
<name>dfs.datanode.data.dir</name>
<value>/home/admin/app/tmp/dfs/data</value>

</property>
```



4.修改YARN-site.xml ,配两组属性



纠错！！！！！mapreduce-shuffle 改为 MapReduce_shuffle !!!!!!

 = =.     =  改为 _



可启动./start-yarn.sh

验证：8088端口来看resourcemanager



```
        
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.resourcemanager.hostname</name>
<value>hadoop000</value>
</property>


```



5.拷贝一份mapred-site.xml.template到mapred-site.xml

cp mapred-site.xml.template mapred-site.xml

添加一组参数，与伪分布式一致



```
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>

```



6.若为集群模式，应修改 slaves文件，将默认的localhost修改成从节点的主机名

vi slaves 

hadoop000

hadoop001

hadoop002



## YARN架构

1） ResourceManger ：RM

​	整个集群同一时间提供服务的RM只有一个，负责集群资源的统一管理和调度

​	处理客户端的请求：提交一个作业、杀死一个作业

​	监控我们的MM，一旦某个MM挂了，那么该NM上运行的任务需要告诉我们的AM如何进行处理



2） NodeManger ：NM

​	集群中有多个，负责自己本身节点资源管理和使用

​	定时向RM汇报本节点的资源使用情况

​	接受并处理来自RM的各种命令：启动Container

​	处理来自AM的命令



3） ApplicationMaster : AM

​	每个应用程序对应一个：MR、Spark，负责应用程序的管理

​	为应用程序向RM申请资源（core、memory），分配给内部task

​	需要与NM通信：启动/停止task,task运行在container里面



4） Container

​	封装了CPU、Memory等资源的一个容器

​	是一个任务运行环境的抽象



5） Client(客户端)

​	提交作业，杀死作业

​	查询作业运行进度



## 分发安装包到Hadoop001 Hadoop002节点



scp -r  ~/app  admin@hadoop001:~/

scp -r  ~/app  admin@hadoop002:~/

分发APP下面的java，Hadoop，tmp文件夹



scp  ~/.bash_profile  admin@hadoop001:~/

scp  ~/.bash_profile  admin@hadoop002:~/

分发环境变量文件



-r目录下面的所有文件传输



在从节点上让环境变量文件生效 source





## 对NAMENode 做格式化，只要在hadoop000上执行即可



cd bin/

./hdfs namenode -format



格式化后，会有信息提示顺利存储在/tmp文件夹内



## 启动集群

格式化成功后在sbin目录内，启动集群

./start-all.sh



## 验证

jps命令

主节点上有6个进程，从节点上有3个进程

7586 DataNode
7923 ResourceManager
8603 Jps
7774 SecondaryNameNode
7439 NameNode



**本次实验缺少NodeManager进程，后续寻找问题**



webUI  http://hadoop000:50070

http://hadoop000:8088

​	



## 集群停止

./stop-all.sh





## BUG

java.net.ConnectException: to 0.0.0.0:10020 failed on connection

这个问题是由于没有启动historyserver引起的，解决办法：
在mapred-site.xml配置文件中添加：
<property>  
​        <name>mapreduce.jobhistory.address</name>  
​        <value>sjfx:10020</value>  
</property>

在namenode上执行命令：mr-jobhistory-daemon.sh start historyserver 

这个问题是由于没有启动historyserver引起的，解决办法：
在mapred-site.xml配置文件中添加：

<property>  
​        <name>mapreduce.jobhistory.address</name>  
​        <value>sjfx:10020</value>  
</property>



在namenode上执行命令：mr-jobhistory-daemon.sh start historyserver 

这样在，namenode上会启动JobHistoryServer服务，可以在historyserver的日志中查看运行情况

## 



## Hive环境搭建



1. Hive下载 hive-1.1.0-cdh5.7.0

2. 解压 tar -zxvf hive..........     -C ~/app

3. 配置Hive

   export HIVE_HOME=/home/admin/app/hive-1.1.0-cdh5.7.0
   export PATH=$HIVE_HOME/bin:$PATH



   conf下的env文件

   HADOOP_HOME=/home/admin/app/hadoop-2.6.0-cdh5.7.0



   4.安装一个mysql，无论是在本地还是在虚拟机上都可以

   5.在conf下创建hive-site.xml，配置四组参数





   ```xml
   <configuration>
   <property>
   <name>javax.jdo.option.ConnectionURL</name>
   <value>jdbc:mysql://server110:3306/hive?createDatabaseIfNotExist=true</value>
   </property>
   <property>
   <name>javax.jdo.option.ConnectionDriverName</name>
   <value>com.mysql.jdbc.Driver</value>
   </property>
   <property>
   <name>javax.jdo.option.ConnectionUserName</name>
   <value>root</value>
   </property>
   <property>
   <name>javax.jdo.option.ConnectionPassword</name>
   <value>root</value>
   </property>
   </configuration>
   ```

4. 拷贝mysql驱动包到hive库文件夹里面：$HIVE_HOME/lib里面

```
cp ~/software/mysql-connector-java-5.6.42-bin.jar .
```

5.启动hive,bin下面的 ./hive



### 用hive完成word-count任务



1.创建表

```
create table hive_wordcount (context string)
```

创建表后，在mysql内的hive（通过hive-site.xml配置文件命名的）数据库内的TBLS代表在hive中创建的所有表的元数据。生成一行元组。





2.加载文件内的数据



```
LOAD DATA LOCAL INPATH 'filepath' INTO TABLE tablename
```

```
LOAD DATA LOCAL INPATH '/home/hadoop/data/hello.txt' INTO TABLE hive_wordcount
```



3.workcount程序用SQL语句执行



```
select word,count(1) from hive_wordcount lateral view explode(split(context,'\t')) wc as word group by word;
```



## MapReduce局限性

​	1）代码繁琐

​	2）只能够支持map和reduce方法

​	3）执行效率低下（中间数据在磁盘上落地）

​	4）不适合迭代多次、交互式、流式的处理





## Hadoop与Spark协作性



Hadoop优势：

- 不受限的规模（多种数据来源，横向机器扩展）
- 企业级的平台（可靠性，安全性）
- 广泛的应用（文件，数据库，半结构化）



Spark优势

- 开发易用性（简单的API，支持Python，java等）
- 内存式的计算框架，计算性能高（RDDs）
- 结合其他框架





## Spark环境搭建



- Spark源码编译



​	需要根据Hadoop版本来对spark进行编译。



```
./dev/make-distribution.sh \
--name 2.6.0-cdh5.7.0 \
--tgz \
-Phadoop-2.6 -Phive \
-Phive-thriftserver -Pyarn \
-Dhadoop.version=2.6.0-cdh5.7.0
```

```
<repository>
    <id>cloudera</id>
    <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
</repository>
```

```
./dev/change scala version
```



成功后：解压出spark-2.1.0-bin-2.6.0-cdh5.7.0.tgz

将其解压



- Spark的简单使用



​	单词统计操作







## 从Hive平滑过渡到Spark SQL



- SQLContext、HIveContext 、SparkSessoion



**Spark1.x中Spark SQL的入口点：SQLcontext**



创建SQLContext上下文



### **IDEA开发小应用程序：Scala**



- 修改Scala版本为2.11.8

- 修改spark-SQL为2.11版本（与Scala对应）

- 可以将版本号当做变量写在properties里面：

  例如：<spark.version>2.1.0<spark.version>



依赖dependencies中引用时写成${spark.version}的形式



#### 新建一个object  SQLContextApp

- 添加maven依赖 hive-jdbc
- 建立Scala class  通过JDBC的方式访问







## DataFrame



A Dataset is a distributed collection of data:分布式的数据集

A DataFrame is a Dataset organized into name columns

以列（列名、列的类型、列值）的形式构成的分布式数据库，按照列赋予不同的名称







## Hive学习



### 创建分区表

```
create external table student_ptn(id int, name string, sex string, age int,department string)
. . . . . . . . . . . . . . .> partitioned by (city string)
. . . . . . . . . . . . . . .> row format delimited fields terminated by ","
. . . . . . . . . . . . . . .> location "/hive/student_ptn";
```

添加分区

```
alter table student_ptn add partition(city="shenzhen");
```



如果某张表是分区表。那么每个分区的定义，其实就表现为了这张表的数据存储目录下的一个子目录
如果是分区表。那么数据文件一定要存储在某个分区中，而不能直接存储在表中。



查看分区信息：

```
show partitions student_ptn;
```



向分区表中插入数据只能存放在特定分区中：下面代码将student文件所有数据放入Beijing子分区中。

```
 load data local inpath "/home/hadoop/student.txt" into table student_ptn partition(city="beijing");
```



### 创建分桶表



```
create external table student_bck(id int, name string, sex string, age int,department string)
. . . . . . . . . . . . . . .> clustered by (id) sorted by (id asc, name desc) into 4 buckets
. . . . . . . . . . . . . . .> row format delimited fields terminated by ","
. . . . . . . . . . . . . . .> location "/hive/student_bck";
```





### 子查询结果创建表



从一个查询SQL的结果来创建一个表进行存储



现有**表student**结构如下所示：

```
+-------------+---------------+--------------+--------------+---------------------+
| student.id  | student.name  | student.sex  | student.age  | student.department  |
+-------------+---------------+--------------+--------------+---------------------+
| 95002       | 刘晨            | 女            | 19           | IS                  |
| 95017       | 王风娟           | 女            | 18           | IS                  |
| 95018       | 王一            | 女            | 19           | IS                  |
| 95013       | 冯伟            | 男            | 21           | CS                  |
```



```
 create table student_ctas as select * from student where id < 95012;
```





### 修改表结构

增加字段：

```sql
alter table new_student add columns (score int);
```

修改字段定义：

```sql

```



### 子查询插入数据



定义一张根据年龄来分区的分区表

```
create table student_ptn_age(id int,name string,sex string,department string) partitioned by (age int);
```



通过子查询插入的操作，并指定partition(age)，实现**动态分区**

```
insert overwrite table student_ptn_age partition(age)
. . . . . . . . . . . . . . .> select id,name,sex,department，age from student_ptn;
```





### 删除，清空表

删除：

```
drop table new_student;
```

清空：

```
truncate table student_ptn;
```



### 创建表时字段的说明

```sql
create external table movie(
userID int comment '用户ID')
```



#### **字段说明通过desc formatted movie会显示出来**



若字段说明为中文，并显示为乱码，则是MYSQL元数据内的默认编码方式不支持中文，应进入元数据库更改。



#### 1、进入数据库 Metastore 中执行以下 5 条 SQL 语句

#### （1）修改表字段注解和表注解

```
alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8；
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8；
```

#### （2）修改分区字段注解

```
alter table PARTITION_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8 ;
alter table PARTITION_KEYS modify column PKEY_COMMENT varchar(4000) character set utf8;
```

#### （3）修改索引注解

```
alter table INDEX_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;
```



#### 2、修改 metastore 的连接 URL



```xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://IP:3306/db_name?createDatabaseIfNotExist=true&amp;useUnicode=true&characterEncoding=UTF-8</value>
    <description>JDBC connect string for a JDBC metastore</description>
</property>
```



### Hive的高级操作



 现有数据如下：

```
1 huangbo guangzhou,xianggang,shenzhen a1:30,a2:20,a3:100 beijing,112233,13522334455,500
2	xuzheng	xianggang	b2:50,b3:40	tianjin,223344,13644556677,600
3	wangbaoqiang	beijing,zhejinag	c1:200	chongqinjg,334455,15622334455,20
```



建表语句

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
use class;
create table cdt(
id int, 
name string, 
work_location array<string>, 
piaofang map<string,bigint>, 
address struct<location:string,zipcode:int,phone:string,value:int>) 
row format delimited 
fields terminated by "\t" 
collection items terminated by "," 
map keys terminated by ":" 
lines terminated by "\n";
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
load data local inpath "/home/admin/data/cdt.txt" into table cdt;
```





###        自定义函数UDF



场景：解析json字符串转换为数据库需要用到解析json的UDF



- **UDF**（user-defined function）作用于单个数据行，产生一个数据行作为输出。（数学函数，字 符串函数）
- UDAF**（用户定义聚集函数 User- Defined Aggregation Funcation）：接收多个输入数据行，并产 生一个输出数据行。（count，max）**
- UDTF（表格生成函数 User-Defined Table Functions）：接收一行输入，输出多行（explode）





#### A.　导入hive需要的jar包，自定义一个java类继承UDF，重载 evaluate 方法

#### B.　打成 jar 包上传到服务器

#### C.　将 jar 包添加到 hive 的 classpath

#### D.　创建临时函数与开发好的 class 关联起来

#### E.　至此，便可以在 hql 在使用自定义的函数





### 例子：

求：所有数学课程成绩 大于 语文课程成绩的学生的学号

```
use myhive;
CREATE TABLE `course` (
  `id` int,
  `sid` int ,
  `course` string,
  `score` int 
) ;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);

```
// 插入数据
// 字段解释：id, 学号， 课程， 成绩
INSERT INTO `course` VALUES (1, 1, 'yuwen', 43);
INSERT INTO `course` VALUES (2, 1, 'shuxue', 55);
INSERT INTO `course` VALUES (3, 2, 'yuwen', 77);
INSERT INTO `course` VALUES (4, 2, 'shuxue', 88);
INSERT INTO `course` VALUES (5, 3, 'yuwen', 98);
INSERT INTO `course` VALUES (6, 3, 'shuxue', 65);
```



#### 1、使用case...when...将不同的课程名称转换成不同的列

```
create view tmp_course_view as
select sid, case course when "shuxue" then score else 0 end  as shuxue,  
case course when "yuwen" then score else 0 end  as yuwen from course;  

select * from tmp_course_view;
```



#### 2、以sid分组合并取各成绩最大值

```
create view tmp_course_view1 as
select aa.sid, max(aa.shuxue) as shuxue, max(aa.yuwen) as yuwen from tmp_course_view aa group by sid;  

select * from tmp_course_view1;
```



#### 3、比较结果

```
select * from tmp_course_view1 where shuxue > yuwen;
```





```
CREATE EXTERNAL TABLE avg_month_table(v1 varchar(9),v2 varchar(9),v3 int ,v4 float,v5 float,v6 float,v7 float,v8 float,v9 float ,v10 float,v11 float,v12 float,v13 float,v14 float ,v15 float ,v16 float,v17 float,v18 float,v19  float,v20 float,v21 float,v22 float,v23 float,v24 float,v25 float,v26 float,v27 float,v28 float,v29 float,v30 float,v31 float,v32 float,v33 float,v34 float,v35 float,v36 float,v37 float) 
ROW FORMAT
DELIMITED FIELDS TERMINATED BY ' '
LINES TERMINATED BY '\n'
tblproperties ("skip.header.line.count"='1');
```





```
LOAD DATA LOCAL INPATH '/home/admin/data/avg_month_weather.txt' INTO TABLE avg_month_table;

select v1,v4,v5,v6,v7,v8,v9,v10,v11,v12,v13,v14,v15,v16,v17 from avg_month_table where v3=1;

insert overwrite local directory "/home/admin/cloud/"
select v1,v4,v5,v6,v7,v8,v9,v10,v11,v12,v13,v14,v15,v16,v17 from avg_month_table where v3=1;

```

```
import org.apache.spark.mllib.clustering.{KMeans, KMeansModel}
import org.apache.spark.mllib.linalg.Vectors

// Load and parse the data
val data = sc.textFile("/home/admin/cloud/000000_0")
val parsedData = data.map(s => Vectors.dense(s.split(' ').map(_.toDouble))).cache()

// Cluster the data into two classes using KMeans
val numClusters = 2
val numIterations = 20
val clusters = KMeans.train(parsedData, numClusters, numIterations)

// Evaluate clustering by computing Within Set Sum of Squared Errors
val WSSSE = clusters.computeCost(parsedData)
println("Within Set Sum of Squared Errors = " + WSSSE)

// Save and load model
clusters.save(sc, "target/org/apache/spark/KMeansExample/test01")
val sameModel = KMeansModel.load(sc, "target/org/apache/spark/KMeansExample/test01")


var predict=clusters.predict(parsedData)
 
predict.collect()
```





```
create table test02(v1 varchar(9),v2 varchar(9),v3 float) ;


insert overwrite table test02 select v1,v2,v4 from avg_month_table where v3=1;
```







```
54398   119812010           0  顺义
54399   119812010       	0	海淀
54406   120002010       	1	延庆
54416   119812010       	1	密云
54419   119962010       	1	怀柔
54419   219811996       	0	怀柔
54421   119812010       	1	密云上甸子
54424   119812006       	1	平谷
54431   119942010       	0	通州
54431   219811993       	0	通州
54433   119812010       	0	朝阳
54499   119812010       	0	昌平
54501   119812010       	1	斋堂
54505   119812010       	0	门头沟
54511   119972010       	0	北京
54511   219811997       	0	北京
54513   119812010       	0	石景山
54514   119812010       	0	丰台
54594   119812010       	0	大兴
54596   119812010       	0	房山
54597   119872010       	0	霞云岭



```

