

# JAVA项目学习

## 第一节

#### 基本JAVA语法

```java
public static void demoSet(){
    Set<String> strSet=new HashSet<String>();
    for(int i=0;i<3;i++){
        strSet.add(String.valueof(i));
    }
}

public static void demoKeyValue(){
    Map<String,String> map=new HashMap<>();
    int i=5
    map.put(String.valueof(i),String.valueof(i*i));
}

public static void demoException(){
    try{
        print();
        String a=null;
        a.indexOf('a');
    }
    //try内为可能出错代码
    catch(NullPointerException npe){
        
    }//catch内可捕捉置顶的异常，这里设为没有指向异常
    
    catch(Exception e){}
    //catch来捕捉异常
    finally{print()};
    //总会走过finally，清理工作
}

public static void demoCommon(){
    Random random =new Random();
    	//设置种子
        random.setSeed(1);
        random.nextInt(100);
    
    List<Interger> array=Arrays.asList(new Interger[]{1,2,4,6});
    //打乱数组
    Collections.shuffle(array);
    
    //时间格式打印
    Date date =new Date();
    date.getTime();
    
    //时间格式化
    //重点
    DateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    
    //UUID
    UUID.randomUUid()
}
```



## 第二节课



### Request Mapping



​		新建一个管理所有controller的文件夹，通过注释@Controller来管理Mapping，基本由RequestMapping,ResponseBody,PathVariable等主要部分所组成。

```java
@Controller
public class IndexController {

    private  static Logger logger= LoggerFactory.getLogger(IndexController.class);

    @Autowired
    private ToutiaoService toutiaoService;

    @RequestMapping(path ={ "/","/index"})
    @ResponseBody
    public String index(HttpSession session){
        logger.info("Visit Index");
        return "HELLO NOW"+session.getAttribute("msg")+"<br>"+toutiaoService.say();
    }

    @RequestMapping(value ={"/profile/{groupId}/{userId}"})
    @ResponseBody
    public String profile(@PathVariable("groupId") String groupId,
                          @PathVariable("userId") int userId,
                          @RequestParam(value = "type",defaultValue = "1")int type,
                          @RequestParam(value = "key",defaultValue = "nowcoder" )String key){
        return String.format("{%s},{%d},{%d},{%s}",groupId,userId,type,key);
        
    }
```



### Request 与 Response

​		request和response头文件里的所有信息可以包装成http 变量，这意味着其中所有的信息都可以被访问到（比如IP地址，访问浏览器，请求方式等），这些信息也被统称为用户行为日志。分别被包装成为

```
HttpServletRequest request和HttpServletResponse response,
```

```java
    @RequestMapping(value = {"/request"})
    @ResponseBody
    public String request(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpSession seesion){

        StringBuilder sb=new StringBuilder();
        Enumeration<String> headerNames=request.getHeaderNames();
        while(headerNames.hasMoreElements()){
            String name=headerNames.nextElement();
            sb.append(name +":"+request.getHeader(name)+"<br>");
        }//打印request头文件内包含的name和对应value

        for(Cookie cookie :request.getCookies()){
            sb.append("Cookies:");
            sb.append(cookie.getName());
            sb.append(":");
            sb.append(cookie.getValue());
            sb.append("<br>");
        }//打印Cookies信息
        sb.append("getMethod:"+request.getMethod()+"<br>");
        sb.append("getPathInfo:"+request.getPathInfo()+"<br>");
        sb.append("getQueryString:"+request.getQueryString()+"<br>");
        sb.append("getRequestURI:"+request.getRequestURI()+"<br>");
        return sb.toString();
    }
```





### 重定向与ERROR

​	重定向在JAVA SPRING中同样使用REQUESTMAPPING来实现，但是因为没有返回内容，只有跳转网页，故而没有RESPONSE BODY的实现。

​	301为强制性跳转，302状态为临时性跳转

```java
    @RequestMapping("/redirect/{code}")//code为传递301,302的参数
    public String redirect(@PathVariable("code") int code,
                           HttpSession session){

        session.setAttribute("msg","Jump from redirect ");
        return "redirect:/";
        
        //返回“redirect：/”的重定向总是301的强制性跳转
    }
```

​	

​	统一处理ERROR的页面

```java
    @ExceptionHandler
    @ResponseBody
    public  String error(Exception e){
        return "error:"+e.getMessage();
    }
}
```





### IOC控制反转（在Controller层调用服务模块）

​	把服务的实现放在一个单独的地方，包装成一个对象，在Controller实现的地方注入服务，用@Autowired这个注释来引用这个对象，作为一个变量来使用。



```java
//调用Service的地方
@Autowired
private ToutiaoService toutiaoService;
```

```java
//实现Service的地方
@Service
public class ToutiaoService {

    public  String say(){
        return "this is from toutiao";
    }
}
```





### 面向接口的编程  AOP

​	面向接口的编程核心思想是“可复用性”，一个解决方案可以提供给所有服务使用，比如一个日志分析工具可以提供给所有的服务来运行。

​	创建aspect的文件夹存放AOP文件，实现LogAspect(日志记录）。

```java
@Aspect
@Component
public class LoginAspect {
    private  static final Logger logger= LoggerFactory.getLogger(LoginAspect.class);

    //before注释后面指定~~切面~~的目标（项目类的所有方法 .*）（在执行这些方法前，都先执行beforeMethod
    @Before("execution(* com.example.demo.controller.IndexController.*(..))")
    public  void  beforeMethod(JoinPoint joinPoint){
        StringBuilder sb=new StringBuilder();
        for(Object arg:joinPoint.getArgs()){
            sb.append("arg"+arg.toString()+"|");
		//joinPoint会把调用方法所包含的所有参数都通过日志记录下来，比如运行
        //@RequestMapping(value ={"/profile/{groupId}/{userId}"})
        }
        logger.info("before method"+sb.toString());
    }
    @After("execution(* com.example.demo.controller.IndexController.*(..))")
    public  void afterMethod(JoinPoint joinPoint){
        logger.info("after method");
    }
}
```



## 第三节课

### 加入mybatis的Maven支持以及JDBC连接



##### 1.pom.xml引入mybatis和mysql-connector-java

```java
<dependency>
   <groupId>org.mybatis.spring.boot</groupId>
   <artifactId>mybatis-spring-boot-starter</artifactId>
   <version>1.3.2</version>
</dependency>


<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <version>8.0.11</version>
</dependency>
```



2.配置application.properties，增加spring配置的数据库的地址（mysql里面的demo数据库）

```xml
spring.datasource.url=jdbc:mysql:///demo
useUnicode=true&
serverTimezone=UTC&
spring.datasource.username=root
spring.datasource.password=LHN123456
mybatis.config-location=classpath:mybatis-config.xml
```



3.mybtis-config.xml  

```java
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
        
        <!-- 参数设置 -->
<settings>

        <!-- Globally enables or disables any caches configured in any mapper under this configuration -->
        <setting name="cacheEnabled" value="true"/>
        <!-- Sets the number of seconds the driver will wait for a response from the database -->
        <setting name="defaultStatementTimeout" value="3000"/>
        <!-- Enables automatic mapping from classic database column names A_COLUMN to camel case classic Java property names aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>

            </settings>
</configuration>
```



### DAO层与MODEL类

#### 1.Model创建

​	声明News,User是JAVA内与数据库表所对应的一些类。在MODEL文件夹下建立News，User类，并赋予它们属性值及生成getter setter方法。

##### 	

#### 2.DAO层创建数据库操作（增删改查）

​	用@Mapper声明是与数据库操作所对应的DAO，在java操作中，将表名，插入字段域，选择字段域独立出来

```java
@Mapper
public interface UserDAO {
 
    //里层存放针对User的操作
}
```

```java
String TABLE_NAME="user";
String INSERT_FIELDS=" name, password, salt, head_url ";
String SELECT_FIELDS=" id , name , password , salt , head_url ";
```



操作示例：**操作对象User需引入Model类的User**

```java
@Insert({"INSERT INTO",TABLE_NAME, "(" ,INSERT_FIELDS,
        ") values (#{name},#{password},#{salt}," +
                "#{headUrl})"})
int addUser(User user);
```

```java
@Select({"select",SELECT_FIELDS,"from",TABLE_NAME,"where id=#{id}"})
User selectById(int id);
```

```java
@Update({"update",TABLE_NAME,"set password=#{password} where id=#{id}"})
void updataPassword(User user);
```



#### 3.User数据库增删改查测试用例

####  

##### 	1.test文件夹下的resources存放创建数据库表的SQL语句

​	init-schema.sql



##### 	2.在执行测试用例之前执行SQL语句，创建数据库

```java
@Sql("/init-schema.sql")
```

​	

##### 	3.注入DAO层操作数据库的方法

```java
@Autowired
UserDAO userDAO;  //导入操作数据库的DAO接口，利用MAPPER的映射执行插入SQL语句   
```

##### 	4.Test,用注释@

```java
@Test
public void contextLoads() {
   Random random=new Random();
   for (int i=0;i<11;i++){
      User user=new User();
      user.setHeadUrl(String.format("http://images.nowcoder.com/head/%dt.png",random.nextInt(1000)));
      user.setName(String.format("USER%d",i));
      user.setPassword("");
      user.setSalt("");
      userDAO.addUser(user);

      user.setPassword("123123");
      userDAO.updataPassword(user);
   }}
```





#### 用XML文件代替@Mapper操作NewsDAO

​		XML配置必须为在相同的包目录下定义的同名XML

​		比如被操作的DAO为存放在com.example.demo.dao.NewsDAO的接口，则对应的XML存放在下图文件夹内

![1543839391768](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1543839391768.png)





### 第四节课(登录与注册)

#### 注册功能

1. 用户名合法性检测（敏感词，长度，重复） 特殊字符（HTML字符，颜文字）
2. 密码长度要求
3. 密码salt加密，密码强度检测（md5库）
4. 用户邮件/短信激活



##### 密码安全：

后台为了安全性，通常在用户输入的密码后面随机加几位数，然后再转换成MD5码存在后台，这样即使拿到了MD5码也难以破解原密码。



##### 实现











