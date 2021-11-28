## 1、回顾MVC

### 1.1、什么是MVC

-   MVC是模型（Model）、视图（View）、控制器（Controller）的简写，是一种软件设计规范。
-   是将业务逻辑、数据、显示分离的方法来组织代码
-   MVC主要作用是降低了视图与业务逻辑的双向耦合。
-   MVC不是一种设计模式，是一种架构模式。不同的MVC存在差异

#### Moldel（模型）：

数据模型，提供要展示的数据，包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao）和服务层（Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。

#### View（视图）：

负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

#### Controller（控制器）：

接收用户的请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示，也就是说控制器做了一个调度员的工作。

最典型的MVC就是JSP+servlet+JavaBean的模式

### MVC框架要做哪些事情

1.  将url映射到Java类或者Java类的方法
2.  封装用户提交的数据
3.  处理请求——调用相关的业务处理——封装响应数据
4.  将响应的数据进行渲染。jsp/html等表示层数据

## 2、SpringMVC

SpringMVC是Spring Framework的一部分 基于Java实现MVC的轻量级Web框架

### Spring MVC的特点：

1.  轻量级，简单易学
2.  高效率，基于请求响应的MVC框架
3.  与Spring兼容性好，无缝连接
4.  约定优于配置
5.  功能强大：RESTful、数据验证、格式化、本地化、主题等
6.  简约灵活

Spring的web框架围绕DispatcherServlet[调度Servlet]设计。

DispatcherServlet的作用是将请求分发到不同的处理器。可以采用基于注解@Controller声明

Dispatcherservlet是一个实际的Servlet（它继承HttpServlet基类）

#### SpringMVC的原理：

当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

## 3、第一个MVC程序

### 3.1、配置版

新建一个Moudle，添加wbe的支持！导入SpringMVC的依赖！

#### 配置web.xml，注册DispatchServlet

```xml
<!--1.注册DispatcherServlet-->
   <servlet>
       <servlet-name>springmvc</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!--关联一个springmvc的配置文件:【servlet-name】-servlet.xml-->
        <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:springmvc.xml</param-value>
       </init-param>
       <!--启动级别-1-->
       <load-on-startup>1</load-on-startup>
   </servlet>

	<!--/ 匹配所有的请求；（不包括.jsp）-->
   	<!--/* 匹配所有的请求；（包括.jsp）-->
   <servlet-mapping>
       <servlet-name>springmvc</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>
```

#### 添加 视图解析器

```xml
<!--视图解析器:DispatcherServlet给他的ModelAndView-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
   <!--前缀-->
   <property name="prefix" value="/WEB-INF/jsp/"/>
   <!--后缀-->
   <property name="suffix" value=".jsp"/>
</bean>
```

编写我们要操作业务Controller ，要么实现Controller接口，要么增加注解；需要返回一个ModelAndView，装数据，封视图；

```java
//注意：这里我们先导入Controller接口
public class HelloController implements Controller {

   public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
       //ModelAndView 模型和视图
       ModelAndView mv = new ModelAndView();

       //封装对象，放在ModelAndView中。Model
       mv.addObject("msg","HelloSpringMVC!");
       //封装要跳转的视图，放在ModelAndView中
       mv.setViewName("hello"); //: /WEB-INF/jsp/hello.jsp
       return mv;
  }
}
```

#### 将自己的类交给SpringIOC容器，注册bean

```xml
<!--Handler-->
<bean id="/hello" class="nuc.ss.controller.HelloController"/>
```

#### 写要跳转的jsp页面，显示ModelandView存放的数据，以及我们的正常页面；

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
   <title>Kuangshen</title>
</head>
<body>
${msg}
</body>
</html>
```

#### **可能遇到的问题：访问出现404，排查步骤：**

1.  查看控制台输出，看一下是不是缺少了什么jar包。
2.  如果jar包存在，显示无法输出，就在IDEA的项目发布中，添加lib依赖！
3.  重启Tomcat 即可解决！

小结：我们来看个注解版实现，这才是SpringMVC的精髓。

### 3.2注解版

新建一个Moudle，添加web支持

在pom.xml文件引入相关的依赖：主要有Spring框架核心库、SpringMVC、servlet、JSTL等

#### 配置web.xml

```xml
 <!--1.注册servlet-->
   <servlet>
       <servlet-name>SpringMVC</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!--通过初始化参数指定SpringMVC配置文件的位置，进行关联-->
       <init-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:springmvc-servlet.xml</param-value>
       </init-param>
       <!-- 启动顺序，数字越小，启动越早 -->
       <load-on-startup>1</load-on-startup>
   </servlet>

   <!--所有请求都会被springmvc拦截 -->
   <servlet-mapping>
       <servlet-name>SpringMVC</servlet-name>
       <url-pattern>/</url-pattern>
   </servlet-mapping>

</web-app>
```

#### /和*的区别：

1.  < url-pattern > / </ url-pattern > 不会匹配到.jsp， 只针对我们编写的请求；即：.jsp 不会进入spring的 DispatcherServlet类 。
2.  < url-pattern > /* </ url-pattern > 会匹配 *.jsp，会出现返回 jsp视图 时再次进入spring的DispatcherServlet 类，导致找不到对应的controller所以报404错。

#### 添加SpringMVC配置文件

在resource目录下添加Springmvc.xml配置文件。配置形式与Spring容器配置基本类似，未来支持基于注解的IOC，设置了自动扫描包的功能，具体配置信息如下。

```xml
<!-- 自动扫描包，让指定包下的注解生效,由IOC容器统一管理 -->
   <context:component-scan base-package="com.lsuhiwu.controller"/>
   <!-- 让Spring MVC不处理静态资源 -->
   <mvc:default-servlet-handler />
   <!--
   支持mvc注解驱动
       在spring中一般采用@RequestMapping注解来完成映射关系
       要想使@RequestMapping注解生效
       必须向上下文中注册DefaultAnnotationHandlerMapping
       和一个AnnotationMethodHandlerAdapter实例
       这两个实例分别在类级别和方法级别处理。
       而annotation-driven配置帮助我们自动完成上述两个实例的注入。
    -->
   <mvc:annotation-driven />

   <!-- 视图解析器 -->
   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
         id="internalResourceViewResolver">
       <!-- 前缀 -->
       <property name="prefix" value="/WEB-INF/jsp/" />
       <!-- 后缀 -->
       <property name="suffix" value=".jsp" />
   </bean>

</beans>
```

在视图解析器中我们把所有的视图都存放在/WEB-INF目录下，这样可以保证视图安全，则应为这个目录下的文件，客户端不能直接访问。

-   让IOC的注解生效
-   静态资源过滤：HTML、JS、CSS、图片、视频……
    -   MVC的注解驱动
    -   配置视图解析器

#### 创建COntroller

编写一个Java控制类：com.lushiwu.controller.MyController,注意编码规范

```java
@Controller
@RequestMapping("/HelloController")
public class HelloController {

   //真实访问地址 : 项目名/HelloController/hello
   @RequestMapping("/hello")
   public String sayHello(Model model){
       //向模型中添加属性msg与值，可以在JSP页面中取出并渲染
       model.addAttribute("msg","hello,SpringMVC");
       //web-inf/jsp/hello.jsp
       return "hello";
  }
}
```

1.  @Controller是为了让给你Spring IOC容器初始化时自动扫描到；
2.  @RequestMapping是为了映射请求路径，这里应为类与方法上都有映射所以访问时应该是/HelloController/hello；
3.  方法中声明Model类型的参数是为了把Action中的数据带到视图中；
4.  方法返回的结果是视图的名称hello，加上配置文件中的前后缀编程WEB-INF/jsp/hello.jsp

#### 创建视层图

在WEB-INF/jsp目录中创建hello.jsp，视图可以直接取出并展示Controller带回的信息，可以通过El表示取出Model中存放的值或者对象。

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
   <title>SpringMVC</title>
</head>
<body>
${msg}
</body>
</html
```

### 小结

实现步骤其实非常的简单：

1.  新建一个web项目
2.  导入相关jar包
3.  编写web.xml , 注册DispatcherServlet
4.  编写springmvc配置文件
5.  接下来就是去创建对应的控制类 , controller
6.  最后完善前端视图和controller之间的对应
7.  测试运行调试.

使用springMVC必须配置的三大件

**处理器映射器、处理器适配器、视图解析器**

通常，我们只需要**手动配置视图解析器**，而**处理器映射器**和**处理器适配器**只需要开启**注解驱动**即可，而省去了大段的xml配置

### 使用method属性指定请求类型

用于约束请求的类型,指定请求谓词的类型如GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE等

我们来测试一下:

```java
//映射访问路径,必须是POST请求
@RequestMapping(value = "/hello",method = {RequestMethod.POST})
public String index2(Model model){
   model.addAttribute("msg", "hello!");
   return "test";
}
```

我们使用浏览器地址栏进行访问默认是Get请求，会报错405：

![image-20211127213339377](/Users/lushiwu/Library/Application Support/typora-user-images/image-20211127213339377.png)

如果将POST修改为GET则正常了；

```java
//映射访问路径,必须是Get请求
@RequestMapping(value = "/hello",method = {RequestMethod.GET})
public String index2(Model model){
   model.addAttribute("msg", "Hello MVC dosome映射消息");
   return "test";
}
```

![image-20211127213659959](/Users/lushiwu/Library/Application Support/typora-user-images/image-20211127213659959.png)

### 小结

SpringMVC的@RequestMapping注解能够处理HTTP请求的方法,比如GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE等

所有的地址请求默认都会是**HTTP GET**类型的

方法级别的注解变体有如下几个:组合注解

@GetMapping @PostMapping @PutMapping @DeleteMapping @PatchMapping 

@GetMapping是一个组合注解,平时使用的会比较多!

它所扮演的是@RequestMapping(mehtod=RequestMethod.GET)的快捷方式

## 4、结果跳转方式

### 4.1、ModelAndView

设置ModelAndView对象,根据view的名称,和视图解析器跳到指定的页面.

页面:{视图解析器前缀} + viewName + {视图解析器后缀}

```xml
<!-- 视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
     id="internalResourceViewResolver">
   <!-- 前缀 -->
   <property name="prefix" value="/WEB-INF/jsp/" />
   <!-- 后缀 -->
   <property name="suffix" value=".jsp" />
</bean>
```

对应的controller类

```java
public class ControllerTest1 implements Controller {
   public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse 					response) throws Exception {
       //返回一个模型视图对象
       ModelAndView mv = new ModelAndView();
       mv.addObject("msg","ControllerTest1");
       mv.setViewName("test");
       return mv;
  }
}
```

### 4.2、ServletAPI

通过设置ServletAPI,不需要视图解析器.

-   通过HttpServletResponse进行输出

-   通过HttpServletresponse实现重定向
-   通过HttpServletResponse实现转发

```java
@Controller
public class ResultGo {

   @RequestMapping("/result/t1")
   public void test1(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
       rsp.getWriter().println("Hello,Spring BY servlet API");
  }

   @RequestMapping("/result/t2")
   public void test2(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
       rsp.sendRedirect("/index.jsp");
  }

   @RequestMapping("/result/t3")
   public void test3(HttpServletRequest req, HttpServletResponse rsp) throws Exception {
       //转发
       req.setAttribute("msg","/result/t3");
       req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,rsp);
  }

}
```

### 5.3、SpringMVC

#### 通过SpringMVC来实现转发和重定向——无需视图解析器:

试前，需要将视图解析器注释掉

-   默认为forward转发（也可以加上）
-   redirect转发需要特别加

```java
@Controller
public class ResultSpringMVC {
   @RequestMapping("/rsm/t1")
   public String test1(){
       //转发
       return "/index.jsp";
  }

   @RequestMapping("/rsm/t2")
   public String test2(){
       //转发二
       return "forward:/index.jsp";
  }

   @RequestMapping("/rsm/t3")
   public String test3(){
       //重定向
       return "redirect:/index.jsp";
  }
}
```

#### 通过SpringMVC来实现转发和重定向 - 有视图解析器；

重定向 , 不需要视图解析器 , 本质就是重新请求一个新地方嘛 , 所以注意路径问题.

可以重定向到另外一个请求实现 .

-   默认为forward转发（不可以加上）
-   redirect转发需特别加

```java
@Controller
public class ResultSpringMVC2 {
   @RequestMapping("/rsm2/t1")
   public String test1(){
       //转发
       return "test";
  }

   @RequestMapping("/rsm2/t2")
   public String test2(){
       //重定向
       return "redirect:/index.jsp";
       //return "redirect:hello.do"; //hello.do为另一个请求/
  }

}

```

## 5、数据处理

### 5.1、处理提交数据

#### 1、提交的域名称和处理方法的参数名一致

提交数据:http://localhost:8080/hello?name=lushiwu

处理方法:

```java
@RequestMapping(“/hello")
public String hello(String name){
	System.out.println(name);
	return "hello";
}
// 后台输出lushiwu
```

#### 2、提交的域名和处理方法的参数名不一致

提交数据:http://localhost:8080/hello?username=lushiwu

处理方法:

```java
@RequestMapping("/hello")
public String hello(@RequestParam("username") String name)
    //@RequestParam("username") : username提交的域的名称 
    System.out.println(name);
	return "hello";
// 后台输出lushiwu
```

### 3、提交的是一个对象

要求提交的表单域和对象的属性名一致,参数使用对象即可

```java
// 实体类
public class User(){
    private int id;
    private String name;
    private int age;
    // 构造
    // get/set
    // toString
}
```

提交数据:http://localhost:8080/mvc/user?id=1&name=lushiwu&age=20

```java
@RequestMapping("/user")
public String user(User user){
    System.out.println(user);
    return "hello"
} // 后台输出:User{id=1,name=lushiwu,age=20}
```

说明: 如果使用对象的话,前段传递的参数名和对象属性名必须一致,否则激素null;

### 5.2、数据显示到前端

#### 第一种:通过ModelAndView

```java
@Controller
public class ControllerTest1
{
    @RequestMaping("/hello")
    public ModelandView test1(){
    	ModelAndView mv = new ModelandView();
        mv.addObject("msg", "Hello MVC");
        mv.setViewName("test");
        return mv;
    }
}
```

#### 第二种:通过ModelMap

```java
@RequestMapping("/hello")
public String test2(@RequestParam("username") String name, ModelMap modelMap){
   //封装要显示到视图中的数据
   //相当于req.setAttribute("name",name);
   modelMap.addAttribute("name",name);
   System.out.println(name);
   return "hello";
}
```

#### 第三种 : 通过Model

```java
@RequestMapping("/hello")
public String test3(@RequestParam("username") String name, Model model){
   //封装要显示到视图中的数据
   //相当于req.setAttribute("name",name);
   model.addAttribute("msg",name);
   System.out.println(name);
   return "test";
}
```

#### 对比

就对于型手而烟简单来说使用的区别就是:

Model 只有寥寥几个方法只适合用于存储数据,简化了新手对于Model对象的操作和理解;

ModelMap 继承了LinkedMap,除了实现了自身的一些方法,同样继承LinkedMap的方法和特性;

ModelAndView 可以在存储数据的同时,可以进行设置返回的逻辑视图,进行控制展示层的跳转.

###### 当然更多的以后开发考虑的更多的是性能和优化，就不能单单仅限于此的了解。

## 6、乱码问题

乱码问题通过过滤器解决,SpringMVC给我们提供 了一个过滤器,在web.xml中配置,修改了xml文件需要重启服务器

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

**注意：这里写/*，写/的话过滤不了jsp页面，不能解决乱码**

 有些极端情况下.这个过滤器对get的支持不好 .

处理方法 :

​		修改tomcat配置文件 ：设置编码！

```xmlm
<Connector URIEncoding="utf-8" port="8080" protocol="HTTP/1.1"
          connectionTimeout="20000"
          redirectPort="8443" />

```

## 7、Json交互处理

### 8.1、什么是JSON?

-   JSON(JavaScript Object Notation, JS 对象标记) 是一种轻量级的数据交换格式，目前使用特别广泛。

-   采用更为安全独立于编程语言的文本格式来存储和表示数据.
-   简洁和清晰的层次结构使得JSON成为理想的数据交换语言.
-   易于人阅读和编写,同时也易于机器解析和生产,并有效地提升网络传输效率

在JavaScript语言中,一切都是对象.因此,任何JavaScript支持的类型都可以通过JSON来表示,例如字符串、

数字、对象、数组等.

-   对象表示为键值对,数据由逗号分隔
-   花括号保存对象
-   方括号保存数组

JSON键值对是用来保存JavaScript对象的一种方式,和JavaScript对象的写法也大同小异,键/值对组合中的键名系在前面并用双引号“”包裹,使用冒号:分隔,然后紧接者值:

```json
{"name":"lushiwu"}
{"age":"20"}
{"sex":"男"}
```

### 8.2、JSON和JavaScript对象互转

要实现从JSON字符串转换为JavaScript对象,使用JSON.parse()方法:

```javascript
var obj = JSON.parse('{"a": "Hello", "b": "World"}');
```

要实现从JavaScript 对象转换为JSON字符串，使用 JSON.stringify() 方法：

```javascript
var json = JSON.stringify({a: 'Hello', b: 'World'});
//结果是 '{"a": "Hello", "b": "World"}'
```

