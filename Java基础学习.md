# Java基础学习



## 类与对象

抽象、封装、继承、多态



### 2.1-声明与创建

- 类是对一类对象的描述
- 对象是类的具体实例

![1555401080646](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555401080646.png)



### 2.2-数据成员，方法成员

![1555401408996](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555401408996.png)

transicent修饰符用于密码等敏感变量



![1555473183972](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555473183972.png)

![1555473445239](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555473445239.png)



![1555473449502](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555473449502.png)



类方法不需要将对象实例化，也意味着该方法执行不依赖于对象中的属性值，可直接使用类名调用。Converter.centigradetoFa



注解：类方法与静态变量一样，不需要进行实例化，可直接应用类名调用，静态变量只存一份，没有多个副本。

![1555473527980](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555473527980.png)



![1555473653767](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555473653767.png)







### 2.3-包



![1555473973396](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555473973396.png)



 ![1555483194337](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555483194337.png)



### 2.4-权限访问控制



![1555557332812](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555557332812.png)



局部变量或形参与该类的变量重名时，需使用到this关键字

![1555557593219](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555557593219.png)



**类方法不能调用实例变量？？？？**

![1555557813197](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555557813197.png)



解答：

![1555558011662](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555558011662.png)



### 2.5-初始化与回收

##### 对象初始化：



为同时实现多个构造方法，同时避免代码冗余，可使用this关键字，在参数少的构造方法内调用参数多的构造方法：

![1555558548658](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555558548658.png)



![1555558670500](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555558670500.png)



![1555558856457](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555558856457.png)

##### 内存回收



对象的自动回收：

- 无用对象
  - 离开了作用域的对象
  - 无引用指向对象
- java运行时会通过垃圾收集器周期性地释放无用对象所使用的内存
- Java运行时系统在对对象进行**自动垃圾回收前**，自动调用finalize方法





垃圾收集器：

- 自动扫描对象的**动态内存区**，对不再使用的对象作标记以回收
- 作为**后台线程**运行，通常在系统空闲时异步地执行。



finalize()方法

在类java.lang.Object中声明，用于释放资源。



### 2.6-枚举类



```java
[public] enum 枚举类型名称
    [implements 接口名称列表]
    {
    枚举值;
    变量成员；
    方法成员；
    }
    
enum Score{
    EXCELLENT,
    QUALIFIED,
    FAILED;
}

public class ScoreTester{
    
    
public static void main(String[] args)
{giveScore(Score.EXCELLENT);}

    
}
```



![1555573716950](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555573716950.png)



#### 2.7 小结

若希望对输出的数据进行格式化，可以使用java.text包内的DecimalFormat类

![1555646218465](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555646218465.png)



![1555646357103](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555646357103.png)



#### 2.8 编程练习



待完成



## 类的重用



### 3.1类继承：

- 根据已有类来定义心累，新类拥有已有类的所有功能
- Java支持类的单继承，每个子类只能有一个直接超类



子类对象：

- 从外部看，应包括
  - 与超类相同的接口
  - 可以有更多的方法和属性



类继承中子类可访问的成员：

**子类不能直接访问超类中的私有属性和方法**



### 3.2隐藏和覆盖



#### 属性的隐藏：

```java
class Parent{
    Number aNumber;
}

class Child extends Parent{
    Float aNumber;
}
```



- 子类中声明了与超类中相同的成员变量名

  - 超类继承的变量将隐藏
  - 子类执行来自超类的操作时，处理自超类继承变量，反之处理自己声明的。
  - 使用super. 访问被隐藏的属性。

- 加入超类静态属性static int x=2;超类属性属于整个对象，不会有多个副本

  - 无论通过子类还是超类访问静态属性，访问的是同一个元素，操作也是针对同一值

- 方法覆盖，子类定义新同名方法,覆盖方法权限仅可以比被覆盖的更宽松public》private 

  

  

  1. 必须覆盖的方法：派生类覆盖基类的抽象方法
  2. 不能覆盖的方法：声明为final的终结方法，（很关键，对于系统安全很重要）。

  





### 3.3Object类



- 所有类的直接或间接超类
- 包含所有公共属性



#### 主要方法

- getClass()
- toString()
- equals(Object obj),比较两对象是否指向同一对象
- clone()复制
- hashCode()返回哈希值



#### 相等和同一

- 同一必定相等，Object对象中的==与equals均用于判断同一

  ![1555398841986](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555398841986.png)

![1555398875339](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555398875339.png)



若需实现创建类中的“相等”比较操作，需要覆盖euqals方法

![1555398951233](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555398951233.png)



### 3.4终结类与终结方法



![1555399430887](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555399430887.png)



![1555399542539](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555399542539.png)



![1555399568286](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555399568286.png)



### 3.5抽象类



- 类名前加修饰符abstract
- 可包含常规类能包含的任何成员，包括非抽象方法
- 也可包含抽象方法：用abstract修饰，只规定行为访问接口，但不规定怎么使用
- 只有当子类实现了所有超类中的抽象方法，子类才不是抽象类

```java
//抽象类
abstract class Number{
    ...
        public abstract <return> <Name>(parameter);
    //仅有方法原型的抽象方法
}

new Number(); //报错


```



![1555400626473](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555400626473.png)



#### 泛型



通过类型实参将传递“类型”给泛型类，从而可以创建不同类型的对象。



**泛型方法**接受的参数，方法内部处理的数据类型可定制。

![1555665934287](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555665934287.png)

##### 

##### 通配符？

```java
public class ShowType {

    public void show(GeneralType<?> o)
    {
        System.out.println(
                o.getObject().getClass().getName());
    }
}
```



##### 有限制的泛型



- 在参数type后使用“extends"关键字并加上类名或接口名，表明该参数代表类型必须是该类的子类或者实现了该接口
  - 对于实现了某接口的有限制泛型，也是使用extends关键字，而不是implements关键字

![1555667306010](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555667306010.png)





##### 疑问？？？？

![1555667530894](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555667530894.png)



### 3.6类的组合



选用已有类的对象，组合起来(包含关系)



组合的语法：

- 将已存在类的对象放在新类中即可

  例如：

  ```java
  class Cooker{
     
  }
  
  class Refrigerator{
      
  }
  
  class Kitchen{
      Cooker myCooker;
      Refrigerator myRefrigerator;
  }
  ```

  

- 组合表达——包含关系
- 继承表达——隶属关系



综合使用继承与组合

```java
class Plate{//声明盘子
    public Plate(int i){
        sout("Plate Constructor");
    }}

class DinnerPlate extends Plate{
    //声明餐盘为盘子的子类
    public DinnerPlate(int i){
        super(i);
        sout("DinnerPlate constructor");
    }
}

class PlaceSetting extends Custom{//餐桌的布置,有组合关系
Spoon sp;Fork frk;Knife kn;
DinnerPlate pl;

    public PlaceSetting(int i){
        super(i+1);//超类的构造方法
        sp=new Spoon(i+2);
    }

}
```





## 接口与多态



### 接口：接口是一个纯的抽象类（只有原型设计）



- 继承：既继承了设计，也继承了方法体实现，不支持接口多继承
- 接口：只继承了接口设计，多继承的变通方法





接口的意义：

- 隐藏实现细节
- 使不同的子类具有统一的对外服务接口





### 4.1 接口



接口是一个纯的抽象类

- 接口中可以规定方法原型：方法名、参数列表，返回类型，但不规定方法主体
- 也**可以包含基本数据类型的数据成员**，**但**它们都默认为static和final



接口的作用：

![1555675419079](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555675419079.png)





![1555675497020](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555675497020.png)

**看起来不相关的Food类与WeatherComponent类，可以有实现同一个功能的接口Edible(可食用)**





![1555675868803](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555675868803.png)



##### ？？？？？



![1555676770892](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555676770892.png)

![1555676749905](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555676749905.png)



### 4.2.1-4.2.2 类型转换



- 转换方式
  - 显式
  - 隐式



##### 类型转换规则

- 基本类型间转换
  - 将值从一种转为另一种
- 引用变量的转换
  - 将引用转换为另一类型的引用，不改变对象本身类型
  - 只能被转为
    - 任何一个（直接和间接）超类的类型（向上转）
    - 对象（即实例）所属的类（或超类）实现的一个接口（向上转）
    - 被转为引用指向的对象的类型**（唯一可以向下转型的情况）**



![1555847034330](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555847034330.png)



##### ？？？？



![1555847838786](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555847838786.png)



![1555847857655](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555847857655.png)



#### 4.2.3 方法的查找



![1555847996780](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555847996780.png)





由于类方法与特定的实例对象没有关联，故而与new出的新建对象本身没有关联，只能**通过对象本身属于什么类来执行类方法**

![1555848073054](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555848073054.png)





#### 4.3 多态的概念



##### 多态：不同类型的对象可响应相同的消息，而响应行为不同



**多态的概念：**

- 超类对象和从相同的超类派生出来的多个子类的对象，可被当作同一种类型的对象对待
- 实现同一接口不同类型的对象，可被当作同一种类型的对象对待
- 可向不同类型的对象发送同样的消息，由于多态性，响应行为却可以有所差别
- 例如：
  - 所有的Object类都响应toString
  - 所有的BankAccount类都可以响应deposit方法



**多态的目的**



![1555902299650](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555902299650.png)





![1555902409401](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555902409401.png)





#### 4.4 多态应用实例



例：二次开发

- 有不同种类的交通工具（vehicle），如bus及car，由此声明一个抽象类Vehicle及两个子类Bus及Car；
- 抽象类Driver及两个子类FemaleDriver和MaleDriver
- 在Driver类中声明抽象方法drives，在两子类中对该方法覆盖
- drive接受一个Vehicle类的参数，当不同类型的交通工具被传入时，可以输出具体的交通工具





**所有子类生成的实例都赋给其超类的引用**

![1555902717879](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555902717879.png)



![1555902848527](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555902848527.png)



![1555902902655](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555902902655.png)



当执行测试代码中的a.drives(x),b.drives(y)时，首先根据引用（由于Driver是抽象类，故a,b不是实例对象）a，b进行**一次分发**，判断所实际属于的对象类型，再根据参数引用实际指向的对象类型进行**二次分发**。





##### 4.4 单选题 ？？？？？



#### 4.5 构造方法与多态性





![1556077889794](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556077889794.png)





### 第五章 输入输出



#### 5.1.1-5.1.2 异常处理概念

![1556168288772](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168288772.png)



![1556168305768](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168305768.png)





#### 5.1.3-5.1.5 异常处理



![1556168371542](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168371542.png)





![1556168342957](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168342957.png)



#### 5.2 输入输出流的概念



![1556168472695](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168472695.png)



![1556168497153](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168497153.png)



![1556168555374](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168555374.png)



![1556168447182](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168447182.png)



#### 5.3.1 写文本文件



![1556168984039](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556168984039.png)



- FileWriter添加true参数，表示若文件存在，则向当前文件中追加内容，而不是删除后重建。
- 将所有IO操作放在try块中，捕捉可能发生的IO错误。



![1556169357574](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556169357574.png)



#### 5.3.2 读文本文件



读文件利用FileReader与BufferedReader来进行。



复制文件：为外部提供可调用方法copy（），CopyMaker类中包含四个方法，分别为copy，open，start，copyfile，后三个方法为辅助性方法，用于实现copy（）,定义为private。

![1556177841678](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556177841678.png)



```java
//每个java文件中仅有一个公有类，而CopyMaker类被设为default类
public class FileCopyTest {
    public static void main(String[] args) {
        if(args.length==2) new CopyMaker().copy(args[0],args[1]);
        else System.out.println("please enter file names");
    }
}
```



##### ？？？？ 在执行java FileCopy 时出现找不到主类的错误

![1556183600949](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556183600949.png)







#### 5.3.3 写二进制文件







#### 5.3.3 读二进制文件



#### 5.3.5 File类



#### 5.3.6 处理压缩文件



#### 5.3.7 对象序列化



#### 5.3.8 随机文件读写







### 第六章



![1556260913139](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556260913139.png)



![1556260926216](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556260926216.png)





![1556261008358](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261008358.png)



![1556261137344](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261137344.png)





![1556261314032](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261314032.png)

**set内禁止出现重复元素，相当于数学中的集合概念**



![1556261265736](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261265736.png)





![1556261366879](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261366879.png)





![1556261393316](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261393316.png)





![1556261429886](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261429886.png)







**Map接口，不同于Collection类的另一类的父接口**

![1556261469661](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261469661.png)



![1556261680075](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261680075.png)



![1556261924031](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556261924031.png)





![1556262000043](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556262000043.png)





![1556262062572](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556262062572.png)



![1556262148321](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556262148321.png)



![1556264742292](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556264742292.png)



![1556264853389](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556264853389.png)



![1556264912335](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556264912335.png)



![1556265696781](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556265696781.png)



![1556265787400](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556265787400.png)



![1556265826516](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556265826516.png)



![1556265856632](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556265856632.png)





![1556266274920](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1556266274920.png)



