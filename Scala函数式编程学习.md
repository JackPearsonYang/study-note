# Scala函数式编程学习



SBT  为Scala准备的 simple build tool，包含依赖管理，整合。



Scala编译级别的语言，但REPL  可以使得Scala交互式的运行，可以更好的用于实验测试



Read Evaluate Print Loop



### 命令行编译Scala程序

​	scalac XXX.scala



### 命令行执行编译后的Scala程序

​	scala XXX.class [传参]





## Scala基础语法

​	

#### 变量

​	三种变量：

1. val 常量
2. var 变量
3. lazy val 惰性求值的常量



数据类型可以不做显示声明，Scala会自动进行类型推导



#### 变量类型



![1546492281058](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546492281058.png)



​	

Numeric types: Byte，Short ，Int，Long，Float ，Double

Boolean:true false

Char:'X'

Unit:在函数式编程中往往表示函数是有副作用的

val p=()

p:Unit=()



#### String

构件于java的String之上

新增了字符串插值的特性



val name="ChenFang"

name:String=ChenFang



调用字符串插值：



s"my name is ${name}"





#### 代码块block



**block用于组织多个表达式,返回最后一个表达式的值**

{

expr1

expr2

}



#### 函数

def functionName(parm: ParamType) : ReturnType={

//function body

}



例：

def Add(x1:int,x2:int)=x1+x2



Add(2,5)

返回值为7



#### if和for

if(logical_expr) valA else valB



if(true) 1 else 2    //>  res0:Int=1



val a=1

if(a==1) a    		//> res2:AnyVal=1

if(a != 1)  "not one"   //>没有else的分支，返回结果为Unit的（）





#### for comprehension



val l=List("alice","bob","cathy")



for(

​	s <- l  //generator迭代器

)println(s)



for{

​	s<-l

​	if(s.length>3)  //filter

}println(s)



val result_for=for{

​	s<-l

​	s1=s.toUpperCase()  

​	if(s1 != "")

}yield(s1)  

**//第三个例子把所有元素都转化为大写**



yield() 导出含义，若满足s1不为空，则把当前s1放在一个新的collection里面。





#### try表达式



var result_try=try{

​	Interger.parseInt("dog")

}	catch{

case _ =>0    //  **_为通配符，可以通配所有对象，=>0代表总是返回0指**

} finally{

​	println("always be printed")

}



**//若result_try为0，代表执行语句中有异常**



#### match表达式（类似switch）



exp match{

case p1=>val1

case p2=>val2

...

case _=>valn    **//  case _ 类似与default**

}



var code=1

var result_match=code match{

case 1=>"one"

case 2=>"two"

case _=>"others"

}



### Scala求值策略



- Call By Value -对函数实参求值，且仅求值一次
- Call By Name - 函数实参每次在函数体内被用到时都会求值



Scala通常使用call by value

**如果函数形参类型以=>开头，那么会使用call by name**



**def foo(x:Int) =x //call by value**

**def foo(x: => Int) =x  //call by name**



#### 求值策略例子



def test1(x:Int ,y :Int ): Int =x *x

def test2(x:=>Int ,y :=>Int ): Int =x *x



test1(3+4,8)  test1(7,8)  7*7   49

test2(3+4,8)   (3+4)*(3+4)  7*(3+4)  7*7  49



test1(7,2*4) test1(7,8)  7**7=49

test2(7,2*4)  7**7=49



![1546755435544](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546755435544.png)



### Scala函数特性



Scala语言支持：

1. 把函数作为实参传递
2. 把函数作为返回值
3. 把函数赋值给变量
4. 把函数存储在数据结构里



在Scala里，函数就像普通变量一样，同样也具有函数的类型



**函数类型的格式为A=>B**，表示接受类型A的参数，并返回类型B的函数



#### 高阶函数

​	用函数作为形参或返回值的函数，称为高阶函数



​	def operate(f:(Int , Int)=>Int)={

​	f(4,4)

​	}



​	def greeting()=(name:String)=>{"hello"+" "+name}



#### 匿名函数（就是函数常量）

​	

​	在Scala里面，匿名函数的定义格式为

（形参列表）=>{函数体}



#### 柯里化



**Curried Function把具有多个参数的函数转换为一条函数链，每个节点都是单一参数**



def add(x:Int ,y:Int)=x+y



def add(x:Int)(y:Int)=x+y



柯里化的例子

​	def curriedAdd(a:Int)(b:Int) =a+b

​	curriedAdd(2)(2)    //4



​	val addOne=curriedAdd(1)_   

​	addOne(2)   //3

​	**通用的整型加一的函数可以用柯里化的加函数与通配符组合构成**





**可以利用柯里化基于通用函数构造一些新的函数**



#### 递归函数

​	递归函数在函数式编程是实现循环的一种技术



​	例子：计算n！

​	

​	def factorial（n:Int): Int=

​	if(n<=0) 1

​	else n*factorial(n-1)



递归常碰到堆栈空间不够的问题，可将递归模式改成尾递归模式



#### 尾递归函数



尾递归函数中所有递归形式的调用都出现在函数的末尾

当编译器检测到一个函数调用是尾递归时，它会覆盖当前活动记录而不是在栈中创建一个新的！！！



//注释为告诉编译器实现尾递归优化

@annotation.tailrc

def factorial(n:Int,m:Int): Int=

​	if (n<=0) m

​	else factorial(n-1,m*n)



factorial(5,1)



**上述代码只需保留一个栈，多开辟一个变量存储空间存放m，保留每次递归结束后的执行结果。**





#### 综合性例子 求a到b区间上f(x)的和



**柯里化函数,a,b表示取值范围区间，在函数里面定义另外一个函数**

def sum(f:Int => Int)(a:Int)(b:Int) :Int={

​	

​	@annotation.tailrec

​	def loop(n:Int,acc:Int):Int={

​	if(n>b){

​	println(s"n=${n},acc=${acc}")

​	acc

​		}

​	else{

​	println(s"n=${n},acc=${acc}")

​	loop(n+1,acc+f(n))

​		}

​	}

​	loop(a,0)

}



//调用sum函数，第一个例子有匿名函数x=>x，即y=x

sum(x = > x)(1)(5)



sum(x=>x*x)(1)(5)







### collection-list基本使用



#### List[T] 

​	T表示泛型，指定数据类型



例子：

val a=List(1,2,3,4)

//输出 List[Int]=List(1,2,3,4)



**val b=0:: a**

**两个冒号的连接操作符，左边是成员，右边是list，将把成员添加到list头部**



var c="x"::"y"::"z"::Nil



**val  c=a:::b**

**三个冒号的连接操作符，左右两边均是list**

#### 



#### 访问list内的元素



a.head  返回第一个元素

a.tail	返回除第一个元素以外的所有元素

a.isEmpty 测试是否为空



利用递归函数遍历list

def walkthru(l:List[Int]): String={

​	if(l.isEmpty)  ""

​	else l.head.toString + " " + walkthru(l.tail)

​	}



walkthru(a)



#### list的高阶函数使用



//保留列表a中的奇数，表达式的值为true，则保留当前x值

//参数为一个匿名函数，接受x，并返回boolean类型变量

a.fiter(x=> x%2 ==1)



//将字符串中的所有数字挑选出来

//先将字符串转变为list

"99 Red Balloons".toList.filter(x=> Character.isDigit(x))



//一直遍历直到取到false值为止

// 99 Red  均不等于B，表达式值返回true，均不终止遍历

"99 Red Balloons".toList.takeWhile(x=> x!='B')

//返回结果为第一次碰到B之前的所有字符所组成的列表



#### list-map,map用来将list内每个元素做一个函数映射



c=List(x,y,z)



c.map(x=> x.toUpperCase)

res: List[String]=List(X,Y,Z)



val q=List(a,List(4,5,6))

//元素是两个list  a=List(1,2,3)

//把双层list里的偶数全部拿出来的方法



q.map(x=> x.filter(_%2==0))

//返回

List(List(2),List(4,6))



q.flatMap(List(List(2),List(4,6)))

//flatMap将两层list摊平，返回

List[Int]=List(2,4,6)





#### Range



**1 to 10 by 2  下闭上闭**

**生成 Range(1,3,5,7,9)**



**(1 to 10).toList**



**1 until 10  下闭上开**



#### Stream



Stream is  a lazy list

1 #:: 2 #:: 3 #:: Stream.empty

![1546935430358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546935430358.png)



![1546935453820](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546935453820.png)



![1546935499365](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546935499365.png)





#### Tuple



Tuple概念与数据库内的记录概念较为类似

![1546935544446](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546935544446.png)



![1546935586115](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546935586115.png)



Tuple可以用于将函数返回的多个值包含在一个Tuple中，



v代表当前元素，t_1,t_2,t_3分别代表累积的元素个数，元素和，元素平方和

```scala
def sumSq(in : List[Int] :(Int,Int,Int))=
in.foldLeft((0,0,0) ((t,v)=>(t._1,t._2+v,t._3+v*v)))
```





#### Map[K,V]  KV键值对



val p=Map(1 -> "David", 9 -> "Elwood")



![1546936061796](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546936061796.png)



p(1)  //根据key值取value

p(9)



p.contains(2)=false



**新增一个键值对 p=p+(8->"Archer")**

**根据key值删除一个键值对 p=p - 1**

**新增list内的若干个键值对 p ++ List(2->"Alice",5->"Bob")**





### Scala实现快速排序



```scala
def qSort(a:List[Int]:List[Int]=
 if(a.length<2) a
 else
   qSort(a.filter(_ <a.head) ++ a.filter( _ == a.head) ++ qSort(a.filter(_>a.head)))
         )
```





## Scala 实现k-means



查看数据格式

```
cat /home/admin/app/spark-2.1.0-bin-2.6.0-cdh5.7.0/data/mllib/sample_kmeans_data.txt


0 1:0.0 2:0.0 3:0.0
1 1:0.1 2:0.1 3:0.1
2 1:0.2 2:0.2 3:0.2
3 1:9.0 2:9.0 3:9.0
4 1:9.1 2:9.1 3:9.1
5 1:9.2 2:9.2 3:9.2

```





```
import org.apache.spark.ml.clustering.KMeans

// Loads data.

val dataset = spark.read.format("libsvm").load("/home/admin/app/spark-2.1.0-bin-2.6.0-cdh5.7.0/data/mllib/sample_kmeans_data.txt")

// Trains a k-means model.
val kmeans = new KMeans().setK(2).setSeed(1L)
val model = kmeans.fit(dataset)

val results = model.transform(dataset)

results.collect()

Array(
[0.0,(3,[],[]                                          ),0], [1.0,(3,[0,1,2],[0.1,0.1,0.1]),0], 
[2.0,(3,[0,1,2],[0                                          .2,0.2,0.2]),0], 
[3.0,(3,[0,1,2],[9.0,9.0,9.0]),1], 
[4.0,(3                                          ,[0,1,2],[9.1,9.1,9.1]),1], 
[5.0,(3,[0,1,2],[9.2,9.2,9.2]),                                          1]
)

results.collect().foreach(
      row => {
        println( row(0) + " is predicted as cluster " + row(2))
      })



0.0 is predicted as cluster 0
1.0 is predicted as cluster 0
2.0 is predicted as cluster 0
3.0 is predicted as cluster 1
4.0 is predicted as cluster 1
5.0 is predicted as cluster 1




insert overwrite local directory "/home/admin/data"
 select v1,v4,v5,v6,v7,v8,v9,v10,v11,v12,v13,v14,v15,v16,v17 from avg_month_table where v3=1;


```

