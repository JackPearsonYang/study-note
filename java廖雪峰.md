



![1561015493085](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561015493085.png)



![1561015710911](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561015710911.png)





![1561015721181](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561015721181.png)



![1561015802949](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561015802949.png)







try-with-resources语法糖，自动关闭InputStream



![1561015976188](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561015976188.png)



![1561016087329](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561016087329.png)



![1561016222342](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561016222342.png)



![1561016847739](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561016847739.png)





利用try-with-resources 一次写入多个字节



![1561016950263](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561016950263.png)



各种各样的InputStream



**![1561017088592](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561017088592.png)**

若将Stream流的附加功能（如加解密，缓冲，签名生成等）对应一个InputStream的子类，则会产生子类爆炸的情况，java选择装饰器模式对提供数据的InputStream进行包装。



![1561017207415](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561017207415.png)

![1561017231592](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561017231592.png)





![1561018710386](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561018710386.png)





![1561019056041](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561019056041.png)



![1561019193361](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561019193361.png)



![1561019641593](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561019641593.png)





Writer可通过OutPutStream构造，Reader可通过InputPutStream构造

![1561020779720](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561020779720.png)







日期和时间





![1561021688881](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561021688881.png)



java有两套处理日期，时间的API。JDK8引入新API java.time



![1561021910706](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561021910706.png)



自己想要的方式格式化Date



![1561022002573](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561022002573.png)

![1561031430854](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561031430854.png)

![1561031522226](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561031522226.png)







![1561087042082](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561087042082.png)

localdatatime可以转换为zonedatetime

![1561087349171](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561087349171.png)

数据库日期与java类的对应



![1561089161237](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561089161237.png)



![1561089678368](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561089678368.png)



**epochToString  复杂处理**

![1561089679453](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561089679453.png)







![1561090182930](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561090182930.png)

Junit断言





![1561090593198](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561090593198.png)





![1561105921608](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561105921608.png)



参数化测试

![1561106748939](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561106748939.png)





正则化



![1561107686897](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561107686897.png)



![1561107912490](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561107912490.png)





![1561107923638](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561107923638.png)



![1561107970315](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561107970315.png)





*号修饰之前的\d，表示匹配任意个数字

+号修饰之前的\d，表示匹配至少一个数字

![1561108025202](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561108025202.png)



（）在正则表达式中很重要，



![1561108712661](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561108712661.png)



![1561108718644](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561108718644.png)







![1561185944054](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561185944054.png)

找一串数字末尾有几个零

预想的方法：



![1561186377711](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561186377711.png)



![1561186385901](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561186385901.png)

![1561186417225](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561186417225.png)

![1561190475924](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561190475924.png)







![1561190422982](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561190422982.png)



![1561190506667](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561190506667.png)







线程直接调用run方法是无效的，但必须复写run方法（调用start方法会自动调用run方法）



![1561190657764](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561190657764.png)





设定线程优先级

![1561190830421](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561190830421.png)





![1561191016336](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561191016336.png)



![1561191083982](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561191083982.png)



主线程中使用interrupt方法，中断自建线程



![1561191260563](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561191260563.png)







守护线程





![1561191414827](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561191414827.png)





![1561192696898](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561192696898.png)





![1561192701639](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561192701639.png)







![1561193420048](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561193420048.png)



​      



![1561346888302](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561346888302.png)



若读取非基本数据类型，则读取方法同样为非原子操作，同样需要同步

下图中读取result0时，其他线程可能改变result1的值



![1561347101643](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561347101643.png)







![1561347164909](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561347164909.png)







![1561347280035](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561347280035.png)



不同线程获取多个不同对象的锁可能会导致死锁



![1561347301509](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561347301509.png)







![1561348035051](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561348035051.png)





只要在synchronize代码块中才能调用wait方法，它是native方法，通过JVM实现

wait方法调用时，将释放获取的对象锁，等待完毕后会重新获得锁。



wait与notify的配合使得不同线程操作同一对象变得更为灵活，在队列为空时，可先让取元素的线程进入wait状态，释放队列的锁，使得添加元素的线程可以获取到锁。





![1561348205754](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561348205754.png)





concurrent包可以简化多线程程序的编写



![1561348723204](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561348723204.png)





![1561348791813](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561348791813.png)







![1561356387317](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561356387317.png)





![1561356445648](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561356445648.png)



condition对象中的方法可用于代替原生的wait notifyAll方法







![1561356612330](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561356612330.png)



Concurrent包提供的与各大容器类对应的线程安全容器



![1561357166038](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561357166038.png)





Atomic类基本原理是CAS，Compare And Set



CAS：var.compareAndSet(prev,next) 若当前变量值为prev，则将其更新为next，并

返回true，否则返回false



![1561357297260](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561357297260.png)



在反复实现的add，Dec方法中，将++，--改为addAndget方法，则它们不再需要被加锁，因为Atomic类中的方法为原子操作。



![1561357668598](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561357668598.png)







![1561357757351](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561357757351.png)



![1561357772926](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561357772926.png)



ExecutorService包提供的线程池类：

- FixedThreadPool  线程数固定

- CachedThreadPool  线程数根据任务动态调整

  

  

  

  ![1561359180600](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561359180600.png)

  

  Future接口表示一个未来可能会返回的接口，需要实现泛型方法call，在主线程中可以使用get（）方法来接收返回异步返回的结果，但该方法可能会导致阻塞。

  

  

  ![1561359291294](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561359291294.png)





Runable与Callable接口的区别



![1561360037284](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561360037284.png)



CompletableFuture会在异步任务结束或出错时，自动调用某个对象的方法



![1561360241223](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561360241223.png)



![1561360390515](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561360390515.png)







![1561360536402](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561360536402.png)



![1561360730072](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561360730072.png)



maven常用插件：生成可执行jar，生成覆盖率报告。



![1561431539358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561431539358.png)



![1561444006464](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561444006464.png)



多线程处理多个客户端链接



![1561444585360](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561444585360.png)





![1561444564539](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561444564539.png)





serversocket可以指定IP地址与端口来进行监听，意味着可以只允许特定的机器进行访问，提高安全性。





![1561445533916](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561445533916.png)







![1561445650559](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561445650559.png)







邮件发送



![1561445907255](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561445907255.png)



![1561448036364](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561448036364.png)





函数式编程



![1561953258632](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561953258632.png)



Lambda表达式：

- 简化语法
- JDK>=1.8



**匿名类与lambda表达式实现单方法接口的比较**



![1561953344383](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561953344383.png)

方法引用（实现接口的多种形式）：



![1561965402623](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561965402623.png)

InputStream与Stream

![1561966072559](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561966072559.png)

Stream与List区别

![1561966113307](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561966113307.png)

- Stream可以“存储”有限个或无限个元素



创建Stream的方法：

- Stream.of(1,2,3,4,5)
- Arrays.stream(theArray)
- aList.stream();



仅提供算法而不是线程序列，可用来生成无限元素的stream

- Stream.generate(Supplier<T> S);



![1561966441484](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561966441484.png)



对于无限序列，若调用遍历或求值方法，将陷入死循环，需要先将无限变为有限。



![1561966532886](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561966532886.png)

![1561966666644](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561966666644.png)

重点：！！！

**Stream.map()接受一个Function参数，即接受一个函数作为参数，达到映射Stream的目的函数式编程**







一个Stream经过滤后得到另一个Stream

**示例1：用于生成偶数序列，被5整除的自然数序列等**

**示例2：用于排除字符串数组中存在的空字符串，null值**



![1561967171032](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561967171032.png)

**reduce()方法：聚合方法**（累加，累乘，计数等）



![1561970673785](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561970673785.png)









