

# JAVA开发工程师课程——翁凯





![1557134392877](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557134392877.png)



![1557134478900](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557134478900.png)





**swap函数无效，因为java函数只能传值。**

**那如果传递的是数组或者对象呢？**

![1557143406359](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557143406359.png)



![1557197482255](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557197482255.png)



![1557197956393](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557197956393.png)





一个Clock类由两个Display类构成，一个代表分钟，一个代表小时。



![1557198800938](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557198800938.png)





![1557199270586](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557199270586.png)



#### String[] a=new String(notes.size());notes.toarray(a);将ArrayList类型转为数组类型	





#### 对象数组的for-each循环与普通数组不同，普通数组的遍历仅拿到每个元素的复制品，而对象拿到的是真实指向的“被管理元素”





![1557474992358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557474992358.png)



#### 造型：把一种类型的对象赋给另外一个类型的变量称为造型



![1557475536942](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557475536942.png)





 ![1557475985177](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1557475985177.png)

![1558167886005](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558167886005.png)



#### String来做+=操作系统开销会很大，一般会使用StringBuffer来代替，sb.append（String对象是不能修改的对象）





![1558180861903](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558180861903.png)



![1558182184497](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558182184497.png)





#### class Fox extends Animal implements Cell(解决了多继承的问题)



![1558405023699](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558405023699.png)

![1558407636085](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558407636085.png)





![1558408366654](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558408366654.png)





#### 在new对象的时候给出的类的定义形成了匿名类



- 在new对象的时候给出的类的定义形成了匿名类
- 匿名类可以继承某类，也可以实现某接口
- Swing的消息机制广泛使用匿名类







![1558409322968](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558409322968.png)





#### MVC=Model View Control 



![1558410218527](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558410218527.png)





#### java异常处理流程





![1558509705408](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558509705408.png)





#### 如果要读文件





光执行下面五步，很可能会遇到问题：



- 文件名在指定目录中不存在
- 判断文件大小时，文件不一定是磁盘上的文件。socket通信链接也是通过文件访问的。
- 高清电影4G，难以拿到大量内存（分配内存空间出错）
- 文件读入途中，网络流中断

![1558511038269](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558511038269.png)





#### BL-Business Logic  



![1558511567633](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1558511567633.png)

