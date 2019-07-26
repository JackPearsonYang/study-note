#  Spring Boot



## 简述



核心思想：约定大于配置





优点：

- 快速创建运行的spring项目
- 使用嵌入式的Servlet容器，无需打成war包
- starters的自动依赖和版本控制
- 大量自动配置













## 微服务简述



微服务：架构风格

一个应用应该是一组小型服务；可通过http方式进行互通





## @Configuration



![1561695452337](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1561695452337.png)









## Web开发



使用SpringBoot

1. 创建应用，选中需要的模块
2. SpringBoot已经默认将场景配置好了，只需要在配置文件中指定少量配置
3. 自己编写业务代码



**自动配置原理？**

需要弄清楚它帮我们配置了什么？能不能修改？能修改哪些配置？

> xxxxAutoConfiguration ：帮我们给容器中自动配置组件
>
> xxxxProperties: 配置类来封装配置文件的内容





### Springboot静态资源映射规则



```
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties implements ResourceLoaderAware {

```

**ResourcesProperties类里可以设置和静态资源有关的参数，比如缓存时间等**



**快捷键 Ctrl+N  查找类名级相关变量**

输入WebMVCAuto,进入该类的定义文件。

有下面一行代码，规定静态资源的映射规则



```java
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Integer cachePeriod = this.resourceProperties.getCachePeriod();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(cachePeriod));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(this.resourceProperties.getStaticLocations()).setCachePeriod(cachePeriod));
                }

            }
        }
```



1、所有/webjars/**，都去classpath:/META-INF/resources/webjars/ z找资源，

**webjars：以jar包的方式引入静态资源**



http://www.webjars.org



在pom文件中引入jquery的webjar依赖

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1-2</version>
</dependency>
```



那么访问路径应为：

localhost:8080/webjars/jquery/3.3.1-2/jquery.js





2、 “/**” 访问当前项目的任何资源，若无人处理，则寻找下列的静态文件的文件夹

```
//resources目录为springboot项目中的classpath类路径

"classpath:/META-INF/resources/", 
"classpath:/resources/", 
"classpath:/static/", 
"classpath:/public/"
"/" :当前项目的根路径
```



localhost:8080/abc === 去静态资源文件夹里面找abc



http://localhost:8080/assets/js/src/search.js  

找到assets目录里的search.js文件



3、 配置欢迎页，静态资源文件夹下的所有index.html页面；被"/**"映射



localhost:8080/  找index.html

```
@Bean
public WebMvcAutoConfiguration.WelcomePageHandlerMapping welcomePageHandlerMapping(ResourceProperties resourceProperties) {
    return new WebMvcAutoConfiguration.WelcomePageHandlerMapping(resourceProperties.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
}
```



4、配置喜欢的图标



所有的**/favicon.ico的路径映射到faviconRequestHandler,都是在静态资源文件夹下找

```java
@Bean
public SimpleUrlHandlerMapping faviconHandlerMapping() {
    SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
    mapping.setOrder(-2147483647);
    mapping.setUrlMap(Collections.singletonMap("**/favicon.ico", this.faviconRequestHandler()));
    return mapping;
}


            @Bean
            public ResourceHttpRequestHandler faviconRequestHandler() {
                ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
                requestHandler.setLocations(this.resourceProperties.getFaviconLocations());
                return requestHandler;
            }
```





### 模板引擎

JSP、Velocity、Thymeleaf



springboot推荐的Thymeleaf



#### 引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```



修改thymeleaf版本

```xml
<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
<thymeleaf-layout-dialect.version>2.1.1</thymeleaf-layout-dialect.version>
```



#### thymeleaf使用&语法



在类路径下的templates目录内，找相应的HTML文件

```java
@RequestMapping("/success")
public String success(){
    //classpath:/templates/success.html
    return "success";
}
```

![1550046552733](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550046552733.png)



使用：

1）、导入thymeleaf的名称空间

```xml
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

使用thymeleaf语法；

<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>



2）、表达式？

> ```properties
> Simple expressions:（表达式语法）
>     Variable Expressions: ${...}：获取变量值；OGNL；
>     		1）、获取对象的属性、调用方法
>     		2）、使用内置的基本对象：
>     			#ctx : the context object.
>     			#vars: the context variables.
>                 #locale : the context locale.
>                 #request : (only in Web Contexts) the HttpServletRequest object.
>                 #response : (only in Web Contexts) the HttpServletResponse object.
>                 #session : (only in Web Contexts) the HttpSession object.
>                 #servletContext : (only in Web Contexts) the ServletContext object.
>                 
>                 ${session.foo}
>             3）、内置的一些工具对象：
> #execInfo : information about the template being processed.
> #messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
> #uris : methods for escaping parts of URLs/URIs
> #conversions : methods for executing the configured conversion service (if any).
> 
> #dates : methods for java.util.Date objects: formatting, component extraction, etc.
> 
> #calendars : analogous to #dates , but for java.util.Calendar objects.
> 
> #numbers : methods for formatting numeric objects.
> #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
> 
> #objects : methods for objects in general.
> #bools : methods for boolean evaluation.
> 
> #arrays : methods for arrays.
> #lists : methods for lists.
> #sets : methods for sets.
> #maps : methods for maps.
> 
> #aggregates : methods for creating aggregates on arrays or collections.
> #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
> 
>     Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
>     	补充：配合 th:object="${session.user}：
>    <div th:object="${session.user}">
>    	#以下的p标签中的*{}都从属于session.user对象
>     <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
>     <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
>     <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
>     </div>
>     
>     Message Expressions: #{...}：获取国际化内容
>     #URL，地址拼接使用@{}，例如@{/order/process(execId=${execId},execType='FAST')}
>     Link URL Expressions: @{...}：定义URL；
>     		@{/order/process(execId=${execId},execType='FAST')}
>     
>     #片段引用~{}，用于引用公共的片段
>     Fragment Expressions: ~{...}：片段引用表达式   
>     		<div th:insert="~{commons :: main}">...</div>
>     		
> Literals（字面量）
>       Text literals: 'one text' , 'Another one!' ,…
>       Number literals: 0 , 34 , 3.0 , 12.3 ,…
>       Boolean literals: true , false
>       Null literal: null
>       Literal tokens: one , sometext , main ,…
> Text operations:（文本操作）
>     String concatenation: +
>     Literal substitutions: |The name is ${name}|
> Arithmetic operations:（数学运算）
>     Binary operators: + , - , * , / , %
>     Minus sign (unary operator): -
> Boolean operations:（布尔运算）
>     Binary operators: and , or
>     Boolean negation (unary operator): ! , not
> Comparisons and equality:（比较运算）
>     Comparators: > , < , >= , <= ( gt , lt , ge , le )
>     Equality operators: == , != ( eq , ne )
> Conditional operators:条件运算（三元运算符）
>     If-then: (if) ? (then)
>     If-then-else: (if) ? (then) : (else)
>     Default: (value) ?: (defaultvalue)
> Special tokens:
>     No-Operation: _ 
> ```

## 



## SpringMVC自动配置



### Spring MVC Auto-configuration



springboot自动配置好了springmvc

以下是springboot对springmvc的默认:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.（视图解析器：根据方法的返回值得到视图对象，视图决定渲染方式（转发，重定向））

- **定制方法：自己给容器中添加一个视图解析器；自动的将其组合进来**

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-static-content))). 静态文件夹路径与webjars

- 自动注册了 of `Converter`, `GenericConverter`, and `Formatter` beans.

  - Converter :转换器，类型转换使用converter
  - Formatter 格式化器 2017/12/17===Date

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-message-converters)).

  - HttpMessageConverters :消息转换器，SpringMVC用来转换http请求和响应的 User类型的对象需要以json形式写出

  - HttpMessageConverters 是从容器中确定的，获取所有的httpmessageconverter

    **自己给容器中添加httpmessageconverter，只需要自己的组件注册进组件中**

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-spring-message-codes)).

- Static `index.html` support.静态首页访问

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-favicon)).

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.2.RELEASE/reference/htmlsingle/#boot-features-spring-mvc-web-binding-initializer)).



### springmvc的扩展

 

springmvc的配置文件案例如下：

```
<mvc:view-controller path="/hello" view-name="/success"/>
<mvc:interceptors>
    <
</mvc:interceptors>
```

**如果想在springboot中添加额外功能，如拦截器等，需要编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型，不能标注@EnableWebMvc**（既保留了自动配置，也能用扩展配置）

If you want to keep Spring Boot MVC features and you want to add additional [MVC configuration](https://docs.spring.io/spring/docs/5.1.4.RELEASE/spring-framework-reference/web.html#mvc) (interceptors拦截器, formatters格式化器, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, you can declare a `WebMvcRegistrationsAdapter` instance to provide such components.



**如果想全面接管springmvc的配置，取消springboot的自动配置，则需要在配置类添加@EnableWebMvc**

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.



**Alt+enter快捷键导入继承类**

```java
@Configuration
public class MymvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
        //浏览器发送/jack请求来到success页面
        registry.addViewController("/jack").setViewName("success");
    }
}
```



**Ctrl+O 打开可以重写的方法**

![1550122302440](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550122302440.png)







### 如何修改Spring Boot的自动配置



模式：

1. Springboot在自动配置很多组件的时候，先看容器中有没有用户配置的（@bean,@Component）,如果有就用用户配置的，如果没有，才自动配置；如果组件有多个（ViewResolver）将用户配置和默认配置组合起来





## RestfulCRUD





### 访问首页

![1550216794159](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550216794159.png)



在controller，service层的同级创建config目录，加入MymvcConfig类来添加Springmvc的配置

**以下的@Configuration包含了两组WebMvcConfigurerAdapter，第一组用来将jack请求转发到success模板，第二组用来"/"请求与"/index.html"请求转发到login模板**，会共同起作用。



另一种使得默认访问首页的方法是使用controller，使“/"与”/index.html“的requestedmapping都返回login模板

```java
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

@Override
public void addViewControllers(ViewControllerRegistry registry) {
   // super.addViewControllers(registry);
    //浏览器发送 /jack 请求来到 success
    registry.addViewController("/jack").setViewName("success");
}

//所有的WebMvcConfigurerAdapter组件都会一起起作用
@Bean //将组件注册在容器
public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
    WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("login");
            registry.addViewController("/index.html").setViewName("login");
        }
    };
    return adapter;
}
```


### 登录



开发期间模板引擎页面修改以后，要实时生效

修改登录表单：

```html
<form class="form-signin" action="dashboard.html" th:action="@{/user/login}" method="post">
    
    <!-- 给表单添加/user/login的行为，方式为post方法-->
```



设立LoginController处理登录请求







1）、**禁用模板引擎的缓存（如何创建application.properties文件）？？**

```
# 禁用缓存
spring.thymeleaf.cache=false 
```

2）、页面修改完成以后ctrl+f9：重新编译；



登陆错误消息的显示

```html
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```





写负责登录的controller



```java
@Controller
public class LoginController {

    @PostMapping(value = "/user/login")
    public  String login(@RequestParam("username") String username,
                         @RequestParam("password") String password,
                        Map<String,Object> map)
    {
        //注意判断用户名为空前要加!
        if(!StringUtils.isEmpty(username) && "123456".equals(password)){
            //登陆成功
            return "dashboard";
        }
        else{
            //登录失败
        map.put("msg","用户名密码错误");
        return "index";}
    }
```



为模板文件中的各项资源重新定义资源路径，@{}会生成相对路径，当类改变时无需修改。

给FORM表单添加对应的变量名name（username与password，与controller中的@RequestParam一一对应）

```HTML
<link href="asserts/css/bootstrap.min.css" th:href="@{webjars/bootstrap/4.2.1/css/bootstrap.css}" rel="stylesheet">



			<input type="text" name="username" class="form-control" placeholder="Username" required="" autofocus="">
			<label class="sr-only">Password</label>
			<input type="password" name="password" class="form-control" placeholder="Password" required="">
```



利用后台返回的“msg"，先查看msg是否为空，若不为空，则显示文本${msg}，th:if优先级高于th:text

```html
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```



登陆成功后链接转到/user/login，刷新会产生**表单重复提交的问题**，故而应该添加一个视图映射来实现登录成功页面的重定向，同时需修改controller内的返回方法为重定向。



```java
Config:

registry.addViewController("/main.html").setViewName("dashboard");


Controller:

return "redirect:/main.html";
```



![1550478122279](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550478122279.png)





### 拦截器机制



1.为实现登录检查，创建组件目录compoents，建立loginhandlerinterceptors类，实现handlerinterceptor，

其中使用到了seesion的用户名loginUser,需要在controller中预先设置好

```java
HttpSession session

session.setAttribute("loginUser",username);
```



```java
/**
 * 进行登录的检查
 */
public class LoginHandlerInterceptors implements HandlerInterceptor {
    //目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        Object user=httpServletRequest.getSession().getAttribute("loginUser");
        if(user == null)
        {//未登录，显示错误消息msg
            httpServletRequest.setAttribute("msg","没有权限，请先登录");
            //返回到登录页面，获取到转发器来转发到登录页面
            httpServletRequest.getRequestDispatcher("/index.html").forward(httpServletRequest,httpServletResponse);
            return false;
        }
        else{
            //已登录，放行请求
            return true;
        }

    }
```





2.在MyMVC Config中注册定义好的拦截器LoginHandlerInterceptors



```java
//注册拦截器
@Override
public void addInterceptors(InterceptorRegistry registry) {
    //
    // uper.addInterceptors(registry);

    registry.addInterceptor(new LoginHandlerInterceptors()).addPathPatterns("/**").excludePathPatterns("/index.html","/","/user/login");
}
```





3.模板引擎内引用session内的用户名



```java
[[${session.loginUser}]]
```



4）、员工列表：

#### thymeleaf公共页面元素抽取

```html
1、抽取公共片段
<div th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</div>

2、引入公共片段
<div th:insert="~{footer :: copy}"></div>
~{templatename::selector}：模板名::选择器
~{templatename::fragmentname}:模板名::片段名

3、默认效果：
insert的公共片段在div标签中
如果使用th:insert等属性进行引入，可以不用写~{}：
行内写法可以加上：[[~{}]];[(~{})]；
```





### CRUD-员工列表



实验要求：

1）、RestfulCRUD：CRUD满足Rest风格；

URI：  /资源名称/资源标识       HTTP请求方式区分对资源CRUD操作

|      | 普通CRUD（uri来区分操作） | RestfulCRUD       |
| ---- | ------------------------- | ----------------- |
| 查询 | getEmp                    | emp---GET         |
| 添加 | addEmp?xxx                | emp---POST        |
| 修改 | updateEmp?id=xxx&xxx=xx   | emp/{id}---PUT    |
| 删除 | deleteEmp?id=1            | emp/{id}---DELETE |

2）、实验的请求架构;

| 实验功能                             | 请求URI | 请求方式 |
| ------------------------------------ | ------- | -------- |
| 查询所有员工                         | emps    | GET      |
| 查询某个员工(来到修改页面)           | emp/1   | GET      |
| 来到添加页面                         | emp     | GET      |
| 添加员工                             | emp     | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/1   | GET      |
| 修改员工                             | emp     | PUT      |
| 删除员工                             | emp/1   | DELETE   |



1.将侧边栏的导航按钮的th:href修改为定向到@{/emps}请求

```html
<a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#" th:href="@{/emps}">
```



2.创建员工列表的controller，在生成employee集合时，运用小技巧：

**先使用employeeDao.getAll()，使用快捷键Ctrl+Alt+V，自动生成变量声明Collection<Employee> employees****

```java
@Controller
public class EmployeeController {
    @Autowired
    EmployeeDao employeeDao;

    @GetMapping("/emps")
    public  String list(Model model){
        Collection<Employee> employees = employeeDao.getAll();
        model.addAttribute("emps",employees);
        return "/emps/list";
    }
}
```



3.list模板修改，去除假数据。**在tr一级加入th:each，对emps进行遍历，在td标签内取值**，

日期型数据可用dates.format进行格式化，性别字段可以用三元运算符将0，1变换成男女

```java
  <thead>
	<tr>
      <th>#</th>
      <th>lastName</th>
      <th>email</th>
      <th>gender</th>
      <th>department</th>
      <th>birth</th>
   </tr>
</thead>
<tbody>
   <tr th:each="emp : ${emps}">
      <td th:text="${emp.id}"></td>
      <td th:text="${emp.lastName}"></td>
      <td th:text="${emp.email}"></td>
      <td th:text="${emp.gender}==0?'女':'男'"></td>
      <td th:text="${emp.department.departmentName}"></td>
      <td th:text="${#dates.format(emp.birth,'yyyy-MM-dd')}"></td>
   </tr>
</tbody>
```





### CRUD-添加员工



1.跳转到添加页面，路径为/emp



list 页面内的“添加员工”按钮改成a标签，跳转到/emp路径

```html
<h2>
   <a class="btn btn-sm btn-success" href="emp" th:href="@{/emp}">员工添加</a>
</h2>
```



在EmployeeController内添加转到添加员工页面的方法，请求方式为GET。需在templates文件夹下创建emps下的add界面，由themeleaf完成解析。

```java
@GetMapping("/emp")
public  String toAddPage(Model model){
    //来到添加页面
        Collection<Department> departments = departmentDao.getDepartments();
        model.addAttribute("depts",departments);
    return "/emps/add";
}
```



<form>
    <div class="form-group">
        <label>LastName</label>
        <input type="text" class="form-control" placeholder="zhangsan">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input type="email" class="form-control" placeholder="zhangsan@atguigu.com">
    </div>
    <div class="form-group">
        <label>Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender"  value="1">
            <label class="form-check-label">男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender"  value="0">
            <label class="form-check-label">女</label>
        </div>
    </div>
    <div class="form-group">
        <label>department</label>
        <select class="form-control">
            <option>1</option>
            <option>2</option>
            <option>3</option>
            <option>4</option>
            <option>5</option>
        </select>
    </div>
    <div class="form-group">
        <label>Birth</label>
        <input type="text" class="form-control" placeholder="zhangsan">
    </div>
    <button type="submit" class="btn btn-primary">添加</button>
</form>





使前台显示后台的部门名选项

```html
<select class="form-control">
   <option th:value="${dept.id}" th:each="dept:${depts}" th:text="${dept.departmentName}">1</option>
</select>
```



form表达改换成执行/emp请求，方式为post方式

```html
<form th:action="@{/emp}" method="post">
```



```java
    @PostMapping("/emp")
    public String AddEmp(Employee employee){

        //来到员工列表页面
        System.out.println("保存的员工信息"+employee);
        //保存员工数据
        employeeDao.save(employee);
        return "redirect:/emps";
    }
}
```



**list模板中，显示员工数据信息的单元，应加入name="email"等，与department内的定义一致**







## 启动mysql容器，并连接





```
[root@localhost ~]# docker run -p 3306:3306 --name mysql01 -e MYSQL_ROOT_PASSWORD=huawei@2018 -d mysql:5.7
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```



