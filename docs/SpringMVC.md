# `Spring MVC`

## 一 简介和体验

### 1.1 介绍

> `Spring Web MVC`是基于`Servlet API`构建的原始`Web`框架，从一开始就包含在`Spring Framework`中。正式名称`Spring Web MVC`来自其源模块的名称（ `spring-webmvc` ），但它通常被称为`Spring MVC`。
>
> * **优势**
    >   * **Spring 家族原生产品，与`IOC`容器等基础设施无缝对接**
>   * 代码清新简洁
>   * 性能卓著
>   * 内部组件化程度高，可插拔式组件即插即用，想要什么功能配置相应组件即可

### 1.2 主要作用
![image-20240320180013965](/images/springMvc/image-20240320180013965.png)

> 其中的`SpringMVC`负责**表述层**（**控制层**）实现简化
>
> - 简化前端参数接收( 形参列表 )：我们能更方便的拿到前端传递过来的参数！
>
> 2. 简化后端数据响应(返回值)：数据响应给前端也更方便！

### 1.3 核心组件和调用流程理解

* 围绕前端控制器模式设计的，其中中央 `Servlet  DispatcherServlet` 做整体请求处理调度！还提供了其他特殊的组件协作完成请求处理和响应呈现！

![image.png (1605×656)](/images/springMvc/image2.png)

`SpringMVC`涉及组件理解

>1. `DispatcherServlet` :  `SpringMVC`提供，需要使用`web.xml`配置使其生效，它是整个流程处理的**核心**，所有请求都经过它的处理和分发！
>2. `HandlerMapping` :  `SpringMVC`提供，我们需要进行`IoC`配置使其加入`IoC`容器方可生效，它内部缓存 `handler(controller方法)和handler访问路径`数据，被`DispatcherServlet`调用，用于查找路径对应的`handler`！
>3. `HandlerAdapter` : `SpringMVC`提供，我们需要进行`IoC`配置使其加入`IoC`容器方可生效，它可以处理请求参数和处理响应数据数据，每次`DispatcherServlet`都是通过`handlerAdapter`间接调用`handler`，他是`handler`和`DispatcherServlet`之间的**适配器**！
>4. `Handler` : `handler`又称**处理器**，他是`Controller`类**内部的方法简称**，是由**我们自己定义**，用来接收参数，向后调用业务，最终返回响应结果！
>5. `ViewResovler` : `SpringMVC`提供，我们需要进行`IoC`配置使其加入`IoC`容器方可生效！**视图解析器**主要作用简化模版视图页面查找的，但是需要注意，前后端分离项目，后端只返回`JSON`数据，**不返回页面，那就不需要视图解析器**！所以，视图解析器，相对其他的组件**不是必须的**！

### 1.4 快速入门

1. 创建maven项目，需要转成web程序项目。

   ![image-20240318153737016](/images/springMvc/image-20240318153737016.png)

   ![image-20240318153905262](/images/springMvc/image-20240318153905262.png)

2. 导入依赖

   ```xml
   <dependencies>
       <!-- springioc相关依赖  -->
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-context</artifactId>
           <version>${spring.version}</version>
       </dependency>
       <!-- web相关依赖  -->
       <!-- 在 pom.xml 中引入 Jakarta EE Web API 的依赖 -->
       <!--
           在 Spring Web MVC 6 中，Servlet API 迁移到了 Jakarta EE API，因此在配置 DispatcherServlet 时需要使用
            Jakarta EE 提供的相应类库和命名空间。错误信息 “‘org.springframework.web.servlet.DispatcherServlet’
            is not assignable to ‘javax.servlet.Servlet,jakarta.servlet.Servlet’” 表明你使用了旧版本的
            Servlet API，没有更新到 Jakarta EE 规范。
       -->
       <dependency>
           <groupId>jakarta.platform</groupId>
           <artifactId>jakarta.jakartaee-web-api</artifactId>
           <version>${servlet.version}</version>
           <scope>provided</scope>
       </dependency>
       <!-- springwebmvc相关依赖  -->
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-webmvc</artifactId>
           <version>${spring.version}</version>
       </dependency>
       <!--解决启动tomcat报错：java：无法访问javax.servlet.ServletException-->
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>javax.servlet-api</artifactId>
           <version>3.1.0</version>
           <scope>provided</scope>
       </dependency>
   </dependencies>
   ```

3. 声明一个**Controller**		`HelloController.java`

   ```java
   @Controller
   public class HelloController {
   
     // handlers
   
     /*
      * handler就是controller内部的具体方法
      * @RequestMapping("/hello") 就是用来向handlerMapping中注册的方法注解!
      * @ResponseBody 代表向浏览器直接返回数据!
      */
     @RequestMapping("/hello")
     @ResponseBody
     public String hello() {
       System.out.println("Hello Spring MVC!");
       return "Hello Spring MVC!";
     }
   }
   ```

4. `Spring MVC`**核心组件配置类**编写 		`SpringMvcConfig.java`

   ```java
   /*
    导入handlerMapping和handlerAdapter的三种方式
     1. 自动导入handlerMapping和handlerAdapter [推荐]
     2. 可以不添加,springmvc会检查是否配置handlerMapping和handlerAdapter,没有配置默认加载
     3. 使用@Bean方式配置handlerMapper和handlerAdapter
   */
   @Configuration  // 配置类
   @EnableWebMvc
   @ComponentScan("com.yjx.controller") // 扫描
   public class SpringMvcConfig implements WebMvcConfigurer {
   
     @Bean
     public HandlerMapping handlerMapping() {
       return new RequestMappingHandlerMapping();
     }
   
     @Bean
     public HandlerAdapter handlerAdapter() {
       return new RequestMappingHandlerAdapter();
     }
   }
   ```

5. `SpringMVC`环境搭建		`MyWebAppInitializer.java`

   ```java
   public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
     /**
      * 指定service / mapper层的配置类
      */
     @Override
     protected Class<?>[] getRootConfigClasses() {
       return null;
     }
     /**
      * 指定springmvc的配置类
      */
     @Override
     protected Class<?>[] getServletConfigClasses() {
       return new Class<?>[]{SpringMvcConfig.class};
     }
     /**
      * 设置dispatcherServlet的处理路径!
      * 一般情况下为 / 代表处理所有请求!
      */
     @Override
     protected String[] getServletMappings() {
       return new String[]{"/"};
     }
   }
   ```

6. `tomcat`配置

   ![image-20240318155044776](/images/springMvc/image-20240318155044776.png)

**启动服务测试**

![image-20240318155241001](/images/springMvc/image-20240318155241001.png)

## 二 接收数据

### 2.1 访问路径设置

#### 2.1.1 精准路径匹配

```java
@RequestMapping("/user/login")   // 精准路径匹配   /user/login才能访问到此方法
@ResponseBody 				// 代表向浏览器直接返回数据，不写的话访问路径会404
public String login() {
    System.out.println("UserController login");
    return "login success";
}
```

#### 2.1.2 模糊路径匹配

```java
/**
*  路径设置为 /*
*    /* 为单层任意字符串  /user/a  /user/aaa 可以访问此handler
*    /user/a/a 不可以
*  路径设置为 /user/**
*    /** 为任意层任意字符串  /user/a  /user/aaa 可以访问此handler
*    /user/a/a 也可以访问
*/
@RequestMapping("/user/*") // 模糊路径匹配
@ResponseBody
public String fuzzyPath() {
    System.out.println("UserController user/*");
    return "user/*";
}
```

#### 2.1.3 `@RequestMapping`作用于类上

> `@RequestMapping`注解可以作用于方法上，也可以作用于类上

```java
@Controller
@RequestMapping("/user") // @RequestMapping作用于类上，相当于给请求路径设置统一前缀
public class UserController {
	@RequestMapping("/login") // 当@RequestMapping也作于用类上了，访问该类下的handler方法必须加上/user前缀，所以/user/login才能访问到此方法
  	@ResponseBody
    public String login() {
        System.out.println("UserController login");
        return "login success";
    }
}    
```

#### 2.1.4 请求方式限制
> **HTTP** 协议定义了**八种请求方式**，`SpringMVC` 中封装到了`RequestMethod`这个**枚举类**中
>
> ```java
> public enum RequestMethod {
> GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
> }
> ```

* 默认情况下：`@RequestMapping("/xxx")` 任何请求方式都可以访问！

```java
// method = {RequestMethod.POST} 只有post请求可以访问，其它请求访问时 报405：方法不允许
@RequestMapping(value = "/register", method = {RequestMethod.POST}) 
@ResponseBody
public String register() { 
    System.out.println("UserController register");
    return "user register";
}

// post、get请求都可以访问
@RequestMapping(value = "/register2", method = {RequestMethod.POST, RequestMethod.GET}) 
@ResponseBody
public String register2() {
    System.out.println("UserController register2");
    return "user register2";
}
```

#### 2.1.5 进阶组合注解

> 特定快捷方式变体
>
> * `@GetMapping`
>
> - `@PostMapping`
> - `@PutMapping`
> - `@DeleteMapping`
> - `@PatchMapping`

```java
@GetMapping("/moveForward") // 相当于 @RequestMapping(value = "/moveForward", method = RequestMethod.GET)
@ResponseBody
public String moveForward() {
    System.out.println("UserController moveForward");
    return "user moveForward";
}
```

* 进阶注解只能写在 handler 方法上，**不可以作用于类上**

### 2.2 接收参数

#### 2.2.1 `param` 和 `json`参数区别

* **参数编码**
    * `param` 类型的参数会被编码为 `ASCII` 码。例如，假设 `name=john doe`，则会被编码为 `name=john%20doe`
    * `JSON` 类型的参数会被编码为 `UTF-8`
* **参数顺序**
    * `param` 类型的参数**没有顺序限制**
    * `JSON` 类型的参数是**有序的**。`JSON` 采用键值对的形式进行传递，其中键值对是有序排列的

1. **数据类型**

* `param` 类型的参数仅支持字符串类型、数值类型和布尔类型等**简单数据类型**
    * `JSON` 类型的参数则支持更**复杂的数据类型**，如数组、对象等
* **嵌套性**
    * `param` 类型的参数**不支持嵌套**
    * `JSON` 类型的参数**支持嵌套**，可以传递更为复杂的数据结构
* **可读性**
    * `param` 类型的参数格式比 `JSON` 类型的参数更加**简单、易读**
    * `JSON` 格式在**传递嵌套数据结构时更加清晰易懂**

> **总结**
>
> `param` 类型的参数适用于单一的数据传递，而 `JSON` 类型的参数则更适用于更复杂的数据结构传递
>
> 在实际开发中，常见的做法是：在 **GET** 请求中采用 `param` 类型的参数，而在 **POST** 请求中采用 `JSON` 类型的参数传递

#### 2.2.2 `param`参数接收

##### 2.2.2.1 直接赋值

```java
/**
* 只要形参数名和类型与传递参数相同，即可自动接收!
* 前端请求：http://localhost:8082/param/value?name=xx&age=1
*/
@GetMapping(value = "/value")
@ResponseBody
public String setupForm(String name, Integer age) {
    System.out.println("name: " + name + ", age: " + age);
    return name + age;
}
```

##### 2.2.2.2 `@RequestParam`注解

```java
/**
* 可以使用 @RequestParam 注解指定新名称 绑定到控制器中的方法参数
* required默认是true，必须传递参数。如果没传递参数：HTTP状态 400 - 错误的请求 -> 所需的请求参数“xxx”不存在
* required = false： 可以不传递这个参数。如果没有设置默认值 值为null
* defaultValue = "Xxx"： 当没有传递参数则使用默认值
* 前端请求：http://localhost:8082/param/paramForm?username=xx&age=1
*/
@GetMapping("/paramForm")
@ResponseBody
public String paramForm(
    @RequestParam(value = "username", required = false, defaultValue = "xxx") String name, 
    @RequestParam("age") Integer age) {
    System.out.println("name: " + name + ", age: " + age);
    return name + age;
}
```

##### 2.2.2.3 特殊场景接值

###### 一名多值

> 此时场景下，**必须加`@RequestParam`注解**
>
> * 为什么不加`@RequestParam`注解就会报错？
    >   * 不加注解，`HandlerAdapter` 会将 `hbs` 对应的一个字符串直接赋值给集合！类型异常
>
> * 为什么加了`@RequestParam`注解就可以？
    >   * 加了注解，`HandlerAdapter` 会将集合 `add` 加入对应的字符串

```java
/**
* 特殊场景赋值
*   一名多值：比如 key=1&key=2&key=3
*   注意：此时场景下，必须加@RequestParam注解，否则会报错  500 java.lang.IllegalStateException: No primary or single unique constructor found for interface java.util.List
*   前端请求：http://localhost:8082/param/paramForm?hbs=11&hbs=111
*/
@GetMapping("/mulForm")
@ResponseBody
public String mulForm(@RequestParam List<String> hbs) {
    System.out.println("hbs: " + hbs);
    return "ok";
}
```

###### 实体类接收参数

> 这里传入的参数名必须等于实体类属性名，否则无法映射，属性值为null(有默认值的则采取默认值)

```java
/**
* 特殊场景赋值
* 实体类接收参数
* 前端请求：这里需要用到postman测试~
*/
@PostMapping("addUser")
@ResponseBody
public String addUser(User user) {
    System.out.println("user: " + user);
    return "success";
}
```

* **Postman测试**

  ![image-20240318214458720](/images/springMvc/image-20240318214458720.png)

#### 2.2.3 路径参数接收

> 使用 `@PathVariable` 注解

```java
/**
* 路径 参数接收
* @PathVariable注解 来处理路径传递参数，如果没使用@PathVariable注解是接收不到路径上传递过来的值的，值为null
* 动态路径设计: /user/{动态部分}/{动态部分}   动态部分使用{}包含即可! {}内部动态标识!
* 前端请求：http://localhost:8082/path/user/1/yjx
* 形参列表取值：
*  - 形参名 和 动态标识 一致，自动赋值
*  - 形参名 和 动态标识 不一致，使用@PathVariable("动态标识")指定
*/
@GetMapping("/user/{id}/{name}")
@ResponseBody
public String getUser(@PathVariable Long id, @PathVariable("name") String username) {
    System.out.println("id:" + id + ",name:" + username);
    return "ok";
}
```

#### 2.2.4 `json`参数接收
> 前端传递 `JSON` 数据时，使用 `@RequestBody` 注解来将 `JSON` 数据**转换**为 `Java` 对象。`@RequestBody` 注解表示当前方法参数的值应该从请求体中获取

```java
/**
* 前端传递 JSON 数据时，Spring MVC 框架可以使用 @RequestBody 注解来将 JSON 数据转换为 Java 对象。@RequestBody 注解表示当前方法参数的值应该从请求体中获取
*/
@PostMapping("/addUser")
@ResponseBody
public String addUser(@RequestBody User user) {
    System.out.println(user);
    return "ok";
}
```

* **Postman测试**

![image-20240318223109434](/images/springMvc/image-20240318223109434.png)

> 报错原因：
>
> * 不支持`json`数据类型处理
>
> - 没有`json`类型处理的工具（`jackson`）
>
> 解决：
>
>  1. `springmvc handlerAdpater`配置`json`转化器：`SpringMvcConfig.class`配置类上面使用`@EnableWebMvc`注解
      >
      >     ![image-20240318223339966](/images/springMvc/image-20240318223339966.png)
>
>  2. `pom.xml` 加入`jackson`依赖
      >
      >     ![image-20240318223354518](/images/springMvc/image-20240318223354518.png)

* **测试**

  ![image-20240318223439265](/images/springMvc/image-20240318223439265.png)


#### 2.2.5 接收文件

##### `@RequestPart`

```java
@ApiOperation("上传文件接口")
@PostMapping(value = "/upload/coursefile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public UploadFileResultDto upload(@RequestPart("filedata") MultipartFile upload) {
    return null;
}
```

> 1. `@RequestPart`这个注解用在`multipart/form-data`表单提交请求的方法上。
> 2. 支持的请求方法的方式`MultipartFile`，属于Spring的`MultipartResolver`类。这个请求是通过`http`协议传输的

##### `@RequestParam`

```java
@ApiOperation("上传文件接口")
@PostMapping(value = "/upload/coursefile", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public UploadFileResultDto upload(@RequestParam("filedata") MultipartFile upload) {
    return null;
}
```

> 1. 用来处理`Content-Type:` 为 `application/x-www-form-urlencoded`编码的内容。（Http协议中，如果不指定`Content-Type`，则默认传递的参数就是`application/x-www-form-urlencoded`类型）
> 2. `RequestParam`可以接受简单类型的属性，也可以接受对象类型。 实质是将`Request.getParameter()` 中的`Key-Value`参数Map利用Spring的转化机制`ConversionService`配置，转化成参数接收对象或字段。
> 3. 在`Content-Type: application/x-www-form-urlencoded`的请求中， `GET`方式中`queryString`的值，和`POST`方式中 body data的值都会被Servlet接受到并转化到`Request.getParameter()`参数集中，所以`@RequestParam`可以获取的到。



##### 区别

> 1. `@RequestPart`这个注解用在`multipart/form-data`表单提交请求的方法上。 支持的请求方法的方式`MultipartFile`，属于[Spring](https://spring.io/)的`MultipartResolver`类。这个请求是通过http协议传输的。
> 2. `@RequestParam`也同样支持`multipart/form-data`请求。（即两者都能用于后端接收文件） 他们最大的不同是，当请求方法的请求参数类型不再是String类型的时候。
> 3. `@RequestParam`适用于`name-valueString`类型的请求域，`@RequestPart`适用于复杂的请求域（像JSON，XML）。


### 2.3 接收Cookie数据

> 使用 `@CookieValue` 注解将 `HTTP Cookie` 的值绑定到控制器中的方法参数

```java
@RequestMapping("/getCookie")
@ResponseBody
public String getCookie(@CookieValue("Idea-a1d2fb68") String cookie) {
    System.out.println(cookie); // 获取Cookie里面Idea-a1d2fb68 的值
    return "ok";
}
```

### 2.4 接收请求头数据

>  使用 `@RequestHeader` 注解将请求标头绑定到控制器中的方法参数

```java
@GetMapping("/demo")
public void handle(@RequestHeader("Accept-Encoding") String encoding,
                   @RequestHeader("Accept-Language") String Language) {
    System.out.println("encoding" + encoding); // 获取请求头Accept-Encoding的值
    System.out.println("Language" + Language); // 获取请求头Accept-Language的值
}
```

### 2.5 原生`API`对象操作

> 获取请求或者响应对象,或者会话等,可以直接在形参列表传入,不分前后顺序！
>
> 注意: 接收原生对象,并不影响参数接收!

```java
@GetMapping("/demo")
@ResponseBody
public String getProtogenesisAPI(HttpSession session, HttpServletRequest request, HttpServletResponse response) {
    String method = request.getMethod();
    System.out.println("method = " + method);
    return "api";
}
```

### 2.6 共享域对象操作

#### 2.6.1 什么是属性(共享)域？

> 在 `JavaWeb` 中，共享域指的是在 `Servlet` 中存储数据，以便在同一 Web 应用程序的多个组件中进行共享和访问。常见的共享域有四种：`ServletContext`、`HttpSession`、`HttpServletRequest`、`PageContext`。
>
> 1. `ServletContext` 共享域：`ServletContext` 对象可以在整个 Web 应用程序中共享数据，是最大的共享域。一般可以用于保存整个 Web 应用程序的全局配置信息，以及所有用户都共享的数据。在 `ServletContext` 中保存的数据是线程安全的。
> 2. `HttpSession` 共享域：`HttpSession` 对象可以在同一用户发出的多个请求之间共享数据，但只能在同一个会话中使用。比如，可以将用户登录状态保存在 `HttpSession` 中，让用户在多个页面间保持登录状态。
> 3. `HttpServletRequest` 共享域：`HttpServletRequest` 对象可以在同一个请求的多个处理器方法之间共享数据。比如，可以将请求的参数和属性存储在 `HttpServletRequest` 中，让处理器方法之间可以访问这些数据。
> 4. `PageContext` 共享域：`PageContext` 对象是在 `JSP` 页面`Servlet` 创建时自动创建的。它可以在 `JSP` 的各个作用域中共享数据，包括`pageScope`、`requestScope`、`sessionScope`、`applicationScope` 等作用域。
>
> 共享域的作用是提供了方便实用的方式在同一 Web 应用程序的多个组件之间传递数据，并且可以将数据保存在不同的共享域中，根据需要进行选择和使用。

#### 2.6.2 Request级别属性(共享)域

##### 2.6.2.1 使用 `Model` 类型的形参

```java
@RequestMapping("/useModel")
@ResponseBody
public String demo(Model model) { // 在形参位置声明Model类型变量，用于存储模型数据
    // 我们将数据存入模型，SpringMVC 会帮我们把模型数据存入请求域
    // 存入请求域这个动作也被称为暴露到请求域
    model.addAttribute("请求范围消息模型","i am very happy[model]");
    return "target";
}
```

##### 2.6.2.2 使用 `ModelMap` 类型的形参

```java
@RequestMapping("/demo")
@ResponseBody
public String demo(ModelMap modelMap) {  // 在形参位置声明ModelMap类型变量，用于存储模型数据
    // 我们将数据存入模型，SpringMVC 会帮我们把模型数据存入请求域
    // 存入请求域这个动作也被称为暴露到请求域
    modelMap.addAttribute("requestScopeMessageModelMap","i am very happy[model map]");
    return "target";
}
```

##### 2.6.2.3 使用` Map `类型的形参

```java
@RequestMapping("/demo")
@ResponseBody
public String demo(Map<String, Object> map) { // 在形参位置声明Map类型变量，用于存储模型数据
    // 我们将数据存入模型，SpringMVC 会帮我们把模型数据存入请求域
    // 存入请求域这个动作也被称为暴露到请求域
    map.put("requestScopeMessageMap","i am very happy[map]");
    return "target";
}
```

##### 2.6.2.4 使用原生 `request` 对象

```java
@RequestMapping("/demo")
@ResponseBody
public String demo(HttpServletRequest request) {  // 拿到原生对象，就可以调用原生方法执行各种操作
    request.setAttribute("requestScopeMessageRequest","i am very happy[request]");
    return "target";
}
```

##### 2.6.2.5 使用 `ModelAndView` 对象

```java
@RequestMapping("/demo")
public ModelAndView demo() {
    // 1.创建ModelAndView对象
    ModelAndView modelAndView = new ModelAndView();
    // 2.存入模型数据
    modelAndView.addObject("requestScopeMessageMAV", "i am very happy[mav]");
    // 3.设置视图名称
    modelAndView.setViewName("target");
    return modelAndView;
}
```

#### 2.6.3 Session级别属性(共享)域

```java
@RequestMapping("/demo")
@ResponseBody
public String demo(HttpSession session) {
    //直接对session对象操作,即对会话范围操作!
    session.setAttribute("1", "1");
    System.out.println(session.getAttribute("1"));
    return "target";
}
```

#### 2.6.4 Application级别属性(共享)域

```java
@RequestMapping("test")
@ResponseBody
public String testAttrSession(HttpServletRequest request) {
    ServletContext servletContext = request.getServletContext(); // 通过request获取ServletContext
    servletContext.setAttribute("appScopeMsg", "i am hungry...");
    return "target";
}
```



## 三 响应数据

### 3.1 handler方法分析

> **一个`controller`的方法是控制层的一个处理器,我们称为`handler`**
>
> `handler`需要使用`@RequestMapping/@GetMapping`等声明路径,在`HandlerMapping`中注册,供`DispatcherServlet` 查找!
>
> `handler`**作用总结**
>
> 1. 接收请求参数(`param, json, pathVariable`, **共享域**等)
>
> 2. 调用业务逻辑
>
> 3. 响应前端数据(页面, `json`, 转发和**重定向**等)
>
> `handler`**如何处理**？
>
> 1. 接收参数: `handler`(形参列表: 主要的作用就是用来接收参数)
> 2. 调用业务: { 方法体  可以向后调用业务方法 `service.xx()` }
> 3. 响应数据: `return` 返回结果,可以快速响应前端数据

* 总结
    * 请求数据**接收**，我们都是通过**handler**的形参列表
    * 前端数据**响应**，我们都是通过**handler**的**return**关键字快速处理！
    * `springMvc `**简化了参数接收和响应**！

```java
@GetMapping
public Object handler(简化请求参数接收) {
    调用业务方法
    返回的结果 (页面跳转，返回数据(json))
    return 简化响应前端数据;
}
```

### 3.2 页面跳转控制

#### 3.2.1 快速返回模板视图

##### 开发模式

* 前后端分离模式：**[重点]**
    * 前端的界面和后端的业务逻辑**通过接口分离开发**的一种方式，前后端分离模式可以**提高开发效率**，同时也有助于**代码重用和维护**
* 混合开发模式
    * 将前端和后端的代码**集成在同一个项目中**，共享相同的技术栈和框架。例如`jsp`技术，小项目比较常见。在大型项目中，这种模式会导致**代码耦合性很高**，**维护和升级难度较大**

##### **`JSP`技术**

> `JSP`（`JavaServer Pages`）是一种动态网页开发技术，**Sun** 公司提出的一种基于 `Java` 技术的 `Web` 页面制作技术，可以在 `HTML` 文件中嵌入 `Java` 代码，生成动态内容的编写更加简单。

##### `SpringMvc`中利用`JSP`返回模板视图案例

> `pom.xml` 依赖

```xml
<dependencies>
    <!-- springioc相关依赖  -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- web相关依赖  -->
    <!-- 在 pom.xml 中引入 Jakarta EE Web API 的依赖 -->
    <!--
        在 Spring Web MVC 6 中，Servlet API 迁移到了 Jakarta EE API，因此在配置 DispatcherServlet 时需要使用
         Jakarta EE 提供的相应类库和命名空间。错误信息 “‘org.springframework.web.servlet.DispatcherServlet’
         is not assignable to ‘javax.servlet.Servlet,jakarta.servlet.Servlet’” 表明你使用了旧版本的
         Servlet API，没有更新到 Jakarta EE 规范。
    -->
    <dependency>
        <groupId>jakarta.platform</groupId>
        <artifactId>jakarta.jakartaee-web-api</artifactId>
        <version>${servlet.version}</version>
      <scope>provided</scope>
    </dependency>
    <!-- springwebmvc相关依赖  -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!--解决启动tomcat报错：java：无法访问javax.servlet.ServletException-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    <!--  jackson依赖, 用于解析json-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.0</version>
    </dependency>
    <!-- jsp需要依赖! jstl-->  
    <dependency> 
        <groupId>jakarta.servlet.jsp.jstl</groupId>  
        <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>  
        <version>3.0.0</version> 
    </dependency> 
</dependencies>
```

> `SpringMvcConfig`配置类
>
> `jsp`视图解析器方法：`configureViewResolvers` 方法来对视图做了一个映射。使之能找到`views`下的`jsp`文件

```java
@EnableWebMvc // 启用Spring MVC的配置
@Configuration // 定义一个Spring配置类
@ComponentScan(basePackages = {"com.yjx.controller"}) // 声明扫描组件的包路径，以便自动发现和注册组件
public class SpringMvcConfig implements WebMvcConfigurer {
    /**
    * 创建并返回一个HandlerMapping实例。
    * 这个方法没有参数。
    *
    * @return HandlerMapping - 一个用于映射请求到处理器的实例。
    */
    @Bean
    public HandlerMapping handlerMapping() {
        return new RequestMappingHandlerMapping();
    }

    /**
    * 配置并返回一个处理程序适配器。
    * 这个方法没有参数。
    *
    * @return HandlerAdapter - 返回一个RequestMappingHandlerAdapter实例，它是一个处理HTTP请求的适配器，能够适应多种类型的处理器（handler）。
    */
    @Bean
    public HandlerAdapter handlerAdapter() {
        return new RequestMappingHandlerAdapter();
    }
    /**
    * 配置视图解析器。
    * 该方法用于指定使用JSP视图解析器，并设置其查找视图的目录和文件后缀。
    *
    * @param registry ViewResolverRegistry对象，用于注册和管理视图解析器。
    */
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // 注册JSP视图解析器，设置视图位于"/WEB-INF/views/"目录下，文件后缀为".jsp"
        registry.jsp("/WEB-INF/views/", ".jsp");
    }
```

> `MyWebAppInitializer`应用初始化器类

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
    * 该函数用于指定service / mapper层的配置类。
    * 这些配置类将被Spring Boot用来进行自动配置。
    *
    * @return 返回一个Class<?>数组，数组中包含了所有需要进行配置的类。
    *         在此函数中，返回null表示不指定额外的配置类。
    */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }
    /**
    * 指定Spring MVC的配置类。
    * 该方法用于覆盖默认的配置，以指定应用使用的特定于Spring MVC的配置类。
    *
    * @return Class<?>[] 返回一个Class对象数组，数组中包含所有要加载的Spring MVC配置类。
    */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        // 指定Spring MVC的配置类为SpringMvcConfig.class
        return new Class<?>[]{SpringMvcConfig.class};
    }
    /**
    * 重写getServletMappings方法来设置dispatcherServlet的处理路径。
    * 该方法不接受参数。
    *
    * @return 返回一个String数组，其中包含dispatcherServlet应该处理的请求路径。
    *         在这个例子中，返回的数组包含单个元素"/"，表示dispatcherServlet将处理所有请求。
    */
    @Override
    protected String[] getServletMappings() {
        // 设置dispatcherServlet处理所有请求
        return new String[]{"/"};
    }
}
```

> `Jsp`页面，放置 `/WEB-INF/`下，避免外部直接访问！
>
> 位置：`/WEB-INF/views/home.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <!-- 可以获取共享域的数据,动态展示!-->
    ${msg}
</body>
</html>
```

> `JSPController.java` 返回视图

```java
@Controller
@RequestMapping("/jsp")
public class JSPController {
    /**
    * 处理跳转到主页的请求。
    *
    * @param model 用于在视图和控制器之间传递数据的模型对象。
    * @return 返回视图的名称，此处为"home"，即将跳转到的JSP页面。
    */
    @GetMapping("/jump")
    public String jumpJsp(Model model) {
        // 向模型中添加一个属性"msg"，其值为"hhh"，这个属性可以在对应的JSP页面中访问
        model.addAttribute("msg", "hhh");
        // 返回视图名，指示视图解析器解析并跳转到"home"页面
        return "home";
    }
}
```

**测试**

![image-20240320154132320](/images/springMvc/image-20240320154132320.png)

#### 3.2.2 转发和重定向

> `Handler `方法返回值来实现快速转发，可以使用 `redirect` 或者 `forward` 关键字来实现重定向或转发
>
> 注意：
>
> 1. 转发和重定向到项目下资源路径都是相同，都不需要添加项目根路径！填写项目下路径即可！换句话说：可以省略项目根路径不写
> 2. 转发重定向都是指定的请求路径。如果想返回页面 直接 return "home"；视图名即可。（参考3.2.1快速返回模板视图）

**转发**

```java
@GetMapping("/forward-demo")
public String forwardDemo() {
    // forward关键字实现转发。 转发到 /jsp/jump 路径
    return "forward:/jsp/jump";
}
```

**重定向**

```java
@GetMapping("/redirect-demo")
public String redirectDemo() {
    // redirect关键字实现重定向。 重定向到 /jsp/jump 路径
    return "redirect:/jsp/jump";
}
```

### 3.3 返回`JSON`数据
#### 前置准备

> 导入`jackson`依赖，用于解析`json`，场景：返回一个实体类对象,会使用`jackson`的序列化工具,转成`json`返回给前端! 否则报415-不支持的媒体类型

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
    <version>2.15.0</version>
</dependency>
```

> 添加`json`数据转化器，在**配置类**上面添加`@EnableWebMvc`注解

```java
@Configuration
@EnableWebMvc // json数据处理,必须使用此注解,因为他会加入json处理器
@ComponentScan(basePackages = {"com.yjx.controller"})
public class SpringMvcConfig implements WebMvcConfigurer {
```

#### `@ResponseBody`

> 方法上使用 `@ResponseBody` 注解，用于将方法返回的对象序列化为 `JSON` 或 `XML` 格式的数据，并发送给客户端

```java
@Controller
@RequestMapping("/json")
public class JSONController {
    
    @PostMapping("/test")
    @ResponseBody // 将方法返回的对象序列化为 JSON 或 XML 格式的数据，并发送给客户端
    public User test(@RequestBody User user) {
        System.out.println("user=" + user);
        // 返回的对象,会使用jackson的序列化工具,转成json返回给前端!
        return user;
    }
}
```

![image-20240320170009835](/images/springMvc/image-20240320170009835.png)

**`@ResponseBody`作用于类上**
> 作用于类上，表示里面每个方法都会有`@ResponseBody`注解

```java
@Controller
@RequestMapping("/json")
@ResponseBody  //responseBody可以添加到类上,代表默认类中的所有方法都生效!
public class JSONController {}
```

#### ` @RestController`

> `@RestController` = `@Controller` + `@ResponseBody`

```java
@RequestMapping("/rest")
@RestController
public class RestControllerTest {

    @PostMapping("/test")
    public User test(@RequestBody User user) {
        System.out.println("user=" + user);
        return user;
    }
}
```

### 3.4 返回静态资源处理

> **概念**
>
> 资源本身已经是可以直接拿到浏览器上使用的程度了，**不需要在服务器端做任何运算、处理**。典型的静态资源包括：
>
> - 纯HTML文件
> - 图片
> - CSS文件
> - JavaScript文件
> - ……

#### 静态资源访问问题

![image-20240320174552711](/images/springMvc/image-20240320174552711.png)

![image-20240320174715230](/images/springMvc/image-20240320174715230.png)

![image-20240320174638665](/images/springMvc/image-20240320174638665.png)

* 很明显，可以看到访问**404**，为什么呢

>**问题分析**
>
>- `DispatcherServlet` 的 `url-pattern` 配置的是“/”
>- `url-pattern` 配置 “/” 表示整个 `Web` 应用范围内所有请求都由 `SpringMVC` 来处理
>- 对 `SpringMVC` 来说，必须有对应的 `@RequestMapping` 才能找到处理请求的方法
>- 现在 `/images/springMvc/shopping_cart.png` 请求没有对应的 `@RequestMapping` 所以返回 **404**

> **解决**
> > 在 `SpringMVC`配置类中重写`configureDefaultServletHandling`方法开启默认`servlet`处理
>
> ```java
> @Configuration
> @EnableWebMvc
> public class SpringMvcConfig implements WebMvcConfigurer {
>  @Bean
>  public HandlerMapping handlerMapping() {
>      return new RequestMappingHandlerMapping();
>  }
>  @Bean
>  public HandlerAdapter handlerAdapter() {
>      return new RequestMappingHandlerAdapter();
>  }
>  /**
>     * 开启静态资源处理
>     *
>     * 配置默认的Servlet处理。
>     * 该方法用于启用默认的Servlet处理机制。当调用此方法时，它会将默认的Servlet处理配置为启用状态。
>     *
>     * @param configurer 默认Servlet处理配置器，用于启用或禁用默认Servlet处理。
>     */
>     @Override
>     public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
>         configurer.enable(); // 启用默认的Servlet处理
>     }
> }
> ```

![image-20240320175120783](/images/springMvc/image-20240320175120783.png)
- 配置完后新的问题：其他原本正常的`handler`请求访问不了了，看看是不是配置类上没有加`@EnableWebMvc`注解

## 四 `RESTFul`风格设计和实战

### 4.1 风格概述
#### 简介

> `RESTful`是一种基于 **HTTP** 和标准化的设计原则的软件架构风格，用于设计和实现可靠、可扩展和易于集成的 Web 服务和应用程序！

![image-20240321223424898](/images/springMvc/image-20240321223424898.png)

#### 风格特点

>1. 每一个`URI`代表1种资源（`URI` 是名词）；
>2. 客户端使用`GET`、`POST`、`PUT`、`DELETE` 4个表示操作方式的动词对服务端资源进行操作：`GET`用来获取资源，`POST`用来新建资源（也可以用于更新资源），`PUT`用来更新资源，`DELETE`用来删除资源；
>3. 资源的表现形式是XML或者`JSON`；
>4. 客户端与服务端之间的交互在请求之间是无状态的，从客户端到服务端的每个请求都必须包含理解请求所必需的信息。

#### 设计规范

##### HTTP协议请求方式要求

* REST 风格主张在项目设计、开发过程中，具体的操作符合**HTTP协议定义的请求方式的语义**。

| 操作     | 请求方式 |
| -------- | :------- |
| 查询操作 | GET      |
| 保存操作 | POST     |
| 删除操作 | DELETE   |
| 更新操作 | PUT      |

##### URL路径风格要求

* REST风格下每个资源都应该有一个唯一的标识符，例如一个 URI（统一资源标识符）或者一个 URL（统一资源定位符）。资源的标识符应该能明确地说明该资源的信息，同时也应该是可被理解和解释的！

* 使用URL+请求方式确定具体的动作，他也是一种标准的HTTP协议请求！

| 操作 | 传统风格                | REST 风格                              |
| ---- | ----------------------- | -------------------------------------- |
| 保存 | /CRUD/saveEmp           | URL 地址：/CRUD/emp 请求方式：POST     |
| 删除 | /CRUD/removeEmp?empId=2 | URL 地址：/CRUD/emp/2 请求方式：DELETE |
| 更新 | /CRUD/updateEmp         | URL 地址：/CRUD/emp 请求方式：PUT      |
| 查询 | /CRUD/editEmp?empId=2   | URL 地址：/CRUD/emp/2 请求方式：GET    |

#### 好处

1. 含蓄，安全

   使用问号键值对的方式给服务器传递数据太明显，容易被人利用来对系统进行破坏。使用 REST 风格携带数据不再需要明显的暴露数据的名称。

2. 风格统一

   URL 地址整体格式统一，从前到后始终都使用斜杠划分各个单词，用简单一致的格式表达语义。

3. 无状态

   在调用一个接口（访问、操作资源）的时候，可以不用考虑上下文，不用考虑当前状态，极大的降低了系统设计的复杂度。

4. 严谨，规范

   严格按照 HTTP1.1 协议中定义的请求方式本身的语义进行操作。

5. 简洁，优雅

   过去做增删改查操作需要设计4个不同的URL，现在一个就够了。

| 操作 | 传统风格                | REST 风格                              |
| :--- | ----------------------- | -------------------------------------- |
| 保存 | /CRUD/saveEmp           | URL 地址：/CRUD/emp 请求方式：POST     |
| 删除 | /CRUD/removeEmp?empId=2 | URL 地址：/CRUD/emp/2 请求方式：DELETE |
| 更新 | /CRUD/updateEmp         | URL 地址：/CRUD/emp 请求方式：PUT      |
| 查询 | /CRUD/editEmp?empId=2   | URL 地址：/CRUD/emp/2 请求方式：GET    |

6. 丰富的语义

   通过 URL 地址就可以知道资源之间的关系。它能够把一句话中的很多单词用斜杠连起来，反过来说就是可以在 URL 地址中用一句话来充分表达语义。

   > `http://localhost:8080/shop`
   >
   > `http://localhost:8080/shop/product`
   >
   > `http://localhost:8080/shop/product/cellPhone`
   >
   > `http://localhost:8080/shop/product/cellPhone/iPhone`
   >

### 4.2 实战

#### 需求

> * **实体类**：User {`id` 唯一标识, `name` 用户名, `age` 用户年龄}
>
> * 功能
    >
    >   * 用户数据分页展示功能（条件：`page` 页数 默认1，`size` 每页数量 默认 `10`）
    >
    >   - 保存用户功能
>   - 根据用户id查询用户详情功能
>   - 根据用户id更新用户数据功能
>   - 根据用户id删除用户数据功能
>   - 多条件模糊查询用户功能（条件：`keyword` 模糊关键字，`page` 页数 默认1，`size` 每页数量 默认 10）

#### `RESTFul`风格接口设计

**接口设计**

| 功能     | 接口和请求方式   | 请求参数                      | 返回值       |
| -------- | ---------------- | ----------------------------- | ------------ |
| 分页查询 | GET  /user       | page=1&size=10                | { 响应数据 } |
| 用户添加 | POST /user       | { user 数据 }                 | {响应数据}   |
| 用户详情 | GET /user/1      | 路径参数                      | {响应数据}   |
| 用户更新 | PUT /user        | { user 更新数据}              | {响应数据}   |
| 用户删除 | DELETE /user/1   | 路径参数                      | {响应数据}   |
| 条件模糊 | GET /user/search | page=1&size=10&keywork=关键字 | {响应数据}   |

**问题**

> 为什么查询用户详情，就使用路径传递参数，多条件模糊查询，就使用请求参数传递？
>
> 误区：restful风格下，不是所有请求参数都是路径传递！可以使用其他方式传递！
>
> 在 `RESTful API` 的设计中，路径和请求参数和请求体都是用来向服务器传递信息的方式。
>
> - 对于查询用户详情，使用路径传递参数是因为这是一个单一资源的查询，即查询一条用户记录。使用路径参数可以明确指定所请求的资源，便于服务器定位并返回对应的资源，也符合 `RESTful` 风格的要求。
> - 而对于多条件模糊查询，使用请求参数传递参数是因为这是一个资源集合的查询，即查询多条用户记录。使用请求参数可以通过组合不同参数来限制查询结果，路径参数的组合和排列可能会很多，不如使用请求参数更加灵活和简洁。
>
> 此外，还有一些通用的原则可以遵循：
>
> - 路径参数应该用于指定资源的唯一标识或者 ID，而请求参数应该用于指定查询条件或者操作参数。
> - 请求参数应该限制在 10 个以内，过多的请求参数可能导致接口难以维护和使用。
> - 对于敏感信息，最好使用 POST 和请求体来传递参数。

**代码实现**

> 用户实体类`User.java`

````java
@Data
public class User {
  private Integer id;
  private String name;
  private Integer age;
}
````

> 控制层 `UserController.java`

```java
@RestController
@RequestMapping("/user")
public class UserController {
    /**
    * 模拟分页查询业务接口
    * 用户数据分页展示功能（条件：page 页数 默认1，size 每页数量 默认 10）
    */
    @GetMapping
    public Object findUserByPage(@RequestParam(value = "page", required = false, defaultValue = "1") Integer page,
                                 @RequestParam(value = "pageSize", required = false, defaultValue = "10") Integer size) {
        System.out.println("page = " + page + ", size = " + size);
        System.out.println("分页查询业务");
        return "{'status':'ok'}";
    }
    
    /**
    * 模拟用户保存业务接口
    */
    @PostMapping
    public Object saveUser(@RequestBody User user) {
        System.out.println(user);
        System.out.println("保存用户业务");
        return "{'status':'ok'}";
    }

    /**
    * 模拟用户详情业务接口
    */
    @GetMapping("/{id}")
    public Object findUserById(@PathVariable("id") Integer id) {
        System.out.println(id);
        System.out.println("查询用户详情业务");
        return "{'status':'ok'}";
    }

    /**
    * 模拟用户更新业务接口
    */
    @PutMapping("/{id}")
    public Object updateUserById(@PathVariable("id") Integer id) {
        System.out.println(id);
        System.out.println("更新用户业务");
        return "{'status':'ok'}";
    }

    /**
    * 模拟用户删除业务接口
    */
    @DeleteMapping("/{id}")
    public Object DeleteUserById(@PathVariable("id") Integer id) {
        System.out.println(id);
        System.out.println("删除用户业务");
        return "{'status':'ok'}";
    }

    /**
    * 模拟条件分页查询业务接口
    */	
    @GetMapping("/search")
    public Object fuzzySearchUser(@RequestParam(value = "page", required = false, defaultValue = "1") Integer page,
                                  @RequestParam(value = "pageSize", required = false, defaultValue = "10") Integer size,
                                  @RequestParam(value = "keyword", required = false) String keyword) {
        System.out.println("page = " + page + ", size = " + size + ", keyword = " + keyword);
        System.out.println("模糊查询用户业务");
        return "{'status':'ok'}";
    }
}
```

## 五 扩展

### 5.1 全局异常处理机制

#### 5.1.1 异常处理两种方式

> 开发过程中是不可避免地会出现各种异常情况的，例如网络连接异常、数据格式异常、空指针异常等等。异常的出现可能导致程序的运行出现问题，甚至直接导致程序崩溃。因此，在开发过程中，合理处理异常、避免异常产生、以及对异常进行有效的调试是非常重要的。

* 编程式异常处理
    * 开发人员在代码中**显式**地编写处理异常的逻辑(例如 `try-catch`)，异常处理代码**混杂**在业务代码中，导致代码**可读性较差**！
* 声明式异常处理
    * 配置等方式进行**统一**的管理和处理。开发人员只需要为方法或类**标注相应的注解**（如 `@Throws` 或 `@ExceptionHandler`），就可以处理特定类型的异常。代码更加**简洁**、**易于维护和扩展**

#### 5.1.2 基于注解异常声明异常处理

1. **声明异常处理控制器类**

```java
@RestControllerAdvice // 使用@RestControllerAdvice注解标记，使其成为一个处理全局异常的控制器顾问，可以捕获并处理应用中抛出的任何异常。
public class GlobalExceptionHandler {

}
```

2. **声明异常处理hander方法**

```java
@RestControllerAdvice // 使用@RestControllerAdvice注解标记，使其成为一个处理全局异常的控制器顾问，可以捕获并处理应用中抛出的任何异常。
public class GlobalExceptionHandler {


  // @ExceptionHandler(NullPointerException.class) 该注解标记异常处理Handler,并且指定发生异常调用该方法!
  /**
   * 处理空指针异常的异常处理器。
   *
   * @param e 异常对象，表示发生的空指针异常。
   * @return 返回一个表示空指针异常信息的Object对象，此处为字符串"空指针异常"。
   */
  @ExceptionHandler(NullPointerException.class)
  public Object handleNullPointerException(NullPointerException e) {
    System.out.println("发生了空指针异常");
    e.printStackTrace();
    // 处理空指针异常，返回相应的异常信息
    return "NullPointerException";
  }


  /**
   * 所有异常都会触发此方法!
   * 但是如果有具体的异常处理Handler，具体异常处理Handler优先级更高。
   * 例如: 发生NullPointerException异常!
   *         会触发 handleNullPointerException 方法,不会触发 handlerException 方法!
   *
   * @param e 抛出的异常对象。
   * @return 返回一个表示异常信息的对象，这里是一个简单的字符串"异常"。
   */
  @ExceptionHandler(Exception.class)
  public Object handleException(Exception e) {
    System.out.println("发生了异常");
    System.out.println(e.getMessage());
    return "Exception";
  }
}
```

3. **配置文件扫描控制器类配置**

![image-20240321222530557](/images/springMvc/image-20240321222530557.png)

**测试**

![image-20240321222732156](/images/springMvc/image-20240321222732156.png)![image-20240321222800214](/images/springMvc/image-20240321222800214.png)

### 5.2 拦截器

#### 5.2.1 概念

* 生活中：为了提高乘车效率，在乘客进入站台前统一检票

  ![img](/images/springMvc/img008.png)

    * 程序中：使用拦截器在请求到达具体 handler 方法前，统一执行检测

![img](/images/springMvc/img009.png)

>  拦截器 `Springmvc`       VS       过滤器 `javaWeb`
>
>  - 相似点
     >   - 拦截：必须先把请求拦住，才能执行后续操作
>   - 过滤：拦截器或过滤器存在的意义就是对请求进行统一处理
>   - 放行：对请求执行了必要操作后，放请求过去，让它访问原本想要访问的资源
>  - 不同点
     >   - 工作平台不同
           >     - 过滤器工作在 `Servlet` 容器中
>     - 拦截器工作在 `SpringMVC `的基础上
>   - 拦截的范围
      >     - 过滤器：能够拦截到的最大范围是整个 `Web` 应用
>     - 拦截器：能够拦截到的最大范围是整个 `SpringMVC` 负责的请求
>   - IOC 容器支持
      >     - 过滤器：想得到 IOC 容器需要调用专门的工具方法，是间接的
>     - 拦截器：它自己就在 `IOC` 容器中，所以可以直接从 `IOC` 容器中装配组件，也就是可以直接得到 `IOC` 容器的支持

**选择哪个呢？**

功能需要如果用 `SpringMVC` 的拦截器能够实现，就不使用过滤器

![image-20240321223645551](/images/springMvc/image-20240321223645551.png)



#### 5.2.2 使用

##### 拦截器方法拦截位置
![image-20240321223645551](/images/springMvc/image-20240321230813377.png)

##### **创建拦截器类**

> 实现 `HandlerInterceptor` 接口，重写其中的方法即可！

```java
/**
 * Process01Interceptor 类实现了 HandlerInterceptor 接口，
 * 用作Spring MVC框架中的拦截器，主要拦截处理器（Handler）的执行，
 * 可以在执行Handler之前或之后进行一些额外的操作。
 */
public class Process01Interceptor implements HandlerInterceptor {

    /**
    * 此方法在目标Handler执行前运行。如果此方法返回false，将阻止Handler的执行。
    * 主要用于进行执行Handler前的预处理或判断逻辑。
    *
    * @param request  HttpServletRequest对象，代表客户端的HTTP请求，可用于获取请求信息。
    * @param response HttpServletResponse对象，用于向客户端发送HTTP响应，可用于设置响应信息。
    * @param handler  将要执行的Handler对象，可以是任何类型的Handler，例如Controller、HandlerMethod等。
    * @return boolean 如果返回true，则继续执行Handler；如果返回false，则阻止执行Handler。
    * 通常根据具体逻辑来决定返回值。
    */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("request: " + request + ", response: " + response + ", handler: " + handler);
        System.out.println("Process01Interceptor.preHandle");
        // 此处默认返回true，表示允许继续执行Handler。返回true：放行;返回false：不放行
        return true;
    }

    /**
    * 在目标Handler执行后执行的方法，handler报错则不会执行。
    *
    * @param request      HttpServletRequest对象，代表客户端的请求。
    * @param response     HttpServletResponse对象，用于向客户端发送响应。
    * @param handler      执行的Handler对象，即处理请求的具体对象。
    * @param modelAndView Handler执行后的ModelAndView对象，包含了模型数据和视图信息。可能为null，取决于Handler的实现。
    * @throws Exception 如果在后处理过程中发生异常，则抛出。
    */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 在这里可以添加后置逻辑，比如对Handler执行结果的处理，页面跳转的逻辑等。可以根据需要实现具体的逻辑代码。
        System.out.println("request: " + request + ", response: " + response + ", handler: " + handler + ", modelAndView: " + modelAndView);
        System.out.println("Process01Interceptor.postHandle");
    }

    /**
    * 在处理请求后执行的回调方法。无论handler报不报错，一定会执行!
    * 该方法用于进行后续处理，比如异常处理、日志记录等。
    *
    * @param request  请求对象，代表客户端发起的HTTP请求。
    * @param response 响应对象，用于向客户端发送HTTP响应。
    * @param handler  处理请求的处理器对象，可以是任何形式的处理器，比如控制器、拦截器等。
    * @param ex       在处理过程中发生的异常，如果处理过程中没有异常发生，则为null。
    * @throws Exception 如果在后续处理过程中发生异常，可以抛出Exception。
    */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 在这里可以添加异常处理逻辑，比如记录异常日志等
        System.out.println("request: " + request + ", response: " + response + ", handler: " + handler + ", ex: " + ex);
        System.out.println("Process01Interceptor.afterCompletion");
    }
}

```

##### 配置类添加拦截器

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {"com.yjx.controller"})
public class SpringMvcConfig implements WebMvcConfigurer {
    // 添加拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
        registry.addInterceptor(new Process01Interceptor());
    }
}
```

> 测试下
>
> ![image-20240321231124758](/images/springMvc/image-20240321231124758.png)![image-20240321231131935](/images/springMvc/image-20240321231131935.png)
>
> ![image-20240321231145770](/images/springMvc/image-20240321231145770.png)
>
> 可以看到拦截器按顺序输出了！

#### 5.2.3 配置详解

##### 默认拦截全部

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    //将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
    registry.addInterceptor(new Process01Interceptor());
}
```

##### 精准配置

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    
    //将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
    registry.addInterceptor(new Process01Interceptor());
    
    //精准匹配,设置拦截器处理指定请求 路径可以设置一个或者多个,为项目下路径即可
    //addPathPatterns("/common/request/one") 添加拦截路径
    //也支持 /* 和 /** 模糊路径。 * 任意一层字符串 ** 任意层 任意字符串
    registry.addInterceptor(new Process01Interceptor()).addPathPatterns("/common/request/one","/common/request/tow");
}
```

##### 排除配置

```java
//添加拦截器
@Override
public void addInterceptors(InterceptorRegistry registry) {

    //将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
    registry.addInterceptor(new Process01Interceptor());

    //精准匹配,设置拦截器处理指定请求 路径可以设置一个或者多个,为项目下路径即可
    //addPathPatterns("/common/request/one") 添加拦截路径
    registry.addInterceptor(new Process01Interceptor()).addPathPatterns("/common/request/one","/common/request/tow");
    
    
    //排除匹配,排除应该在匹配的范围内排除
    //addPathPatterns("/common/request/one") 添加拦截路径
    //excludePathPatterns("/common/request/tow"); 排除路径,排除应该在拦截的范围内
    registry.addInterceptor(new Process01Interceptor())
        .addPathPatterns("/common/request/one","/common/request/tow")
        .excludePathPatterns("/common/request/tow");
}
```

#### 5.2.4 多个拦截器

> 1. `preHandle()` 方法：`SpringMVC` 会把所有拦截器收集到一起，然后按照配置顺序调用各个 `preHandle()` 方法。
> 2. `postHandle()` 方法：`SpringMVC` 会把所有拦截器收集到一起，然后按照配置相反的顺序调用各个 `postHandle()` 方法。也就是越后面的拦截器先执行。
> 3. `afterCompletion()` 方法：`SpringMVC` 会把所有拦截器收集到一起，然后按照配置相反的顺序调用各个 `afterCompletion()` 方法。也就是越后面的拦截器先执行。

* 多个拦截器

![image-20240321231619047](/images/springMvc/image-20240321231619047.png)

* 输出结果

  ![image-20240321231812012](/images/springMvc/image-20240321231812012.png)

### 5.3 参数校验

> 在 Web 应用三层架构体系中，表述层(`Controller`)负责接收浏览器提交的数据，业务逻辑层(`Service`)负责数据的处理。为了能够让业务逻辑层基于正确的数据进行处理，我们需要在表述层对数据进行检查，将错误的数据隔绝在业务逻辑层之外。

#### 5.3.1 概述

> `JSR 303` 是 `Java` 为 `Bean` 数据合法性校验提供的标准框架，它已经包含在 `JavaEE 6.0` 标准中。`JSR 303` 通过在 `Bean` 属性上标注类似于 `@NotNull`、`@Max` 等标准的注解指定校验规则，并通过标准的验证接口对`Bean`进行验证。

| 注解                         | 规则                                           |
| ---------------------------- | ---------------------------------------------- |
| `@Null`                      | 标注值必须为 null                              |
| `@NotNull`                   | 标注值不可为 null                              |
| `@AssertTrue`                | 标注值必须为 true                              |
| `@AssertFalse`               | 标注值必须为 false                             |
| `@Min(value)`                | 标注值必须大于或等于 value                     |
| `@Max(value)`                | 标注值必须小于或等于 value                     |
| `@DecimalMin(value)`         | 标注值必须大于或等于 value                     |
| `@DecimalMax(value)`         | 标注值必须小于或等于 value                     |
| `@Size(max,min)`             | 标注值大小必须在 max 和 min 限定的范围内       |
| `@Digits(integer,fratction)` | 标注值值必须是一个数字，且必须在可接受的范围内 |
| `@Past`                      | 标注值只能用于日期型，且必须是过去的日期       |
| `@Future`                    | 标注值只能用于日期型，且必须是将来的日期       |
| `@Pattern(value)`            | 标注值必须符合指定的正则表达式                 |

> `JSR 303` 只是一套标准，需要提供其实现才可以使用。`Hibernate Validator` 是 `JSR 303` 的一个参考实现，除支持所有标准的校验注解外，它还支持以下的扩展注解：

| 注解        | 规则                               |
| ----------- | ---------------------------------- |
| `@Email`    | 标注值必须是格式正确的 Email 地址  |
| `@Length`   | 标注值字符串大小必须在指定的范围内 |
| `@NotEmpty` | 标注值字符串不能是空字符串         |
| `@Range`    | 标注值必须在指定的范围内           |

* `Spring 4.0` 版本引入了自己的数据校验框架，同时也支持 `JSR 303` 标准的校验框架。在 `SpringMVC` 中，可以通过使用注解驱动方式 `@EnableWebMvc` 来进行数据校验。Spring 的 `LocalValidatorFactoryBean` 实现了 Spring 的 `Validator` 接口和` JSR 303` 的 `Validator` 接口。定义一个 `LocalValidatorFactoryBean` 并注入到需要数据校验的 Bean 中，就能够进行数据校验。在配置了 `@EnableWebMvc` 后，`SpringMVC` 会默认配置一个 `LocalValidatorFactoryBean`，通过在处理方法的入参上标注 `@Validated` 注解，`SpringMVC` 在完成数据绑定后会执行数据校验的工作。



#### 5.3.2 实践

##### **导入依赖**

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.2.5.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-annotation-processor</artifactId>
    <version>6.2.5.Final</version>
</dependency>
```

##### 实体类添加注解校验

```java
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull; // 一定要注意这个的导包是否正确
@Data
public class User {
    /**
    * 年龄属性
    * 该属性必须大于等于10。
    */
    @Min(10)
    private Integer age;

    /**
    * name属性
    * 该属性为一个字符串类型，用于表示某个名称。
    * 它具有长度限制，必须在3到6个字符之间。
    */
    @NotNull(message = "用户名不能为空")
    @Length(min = 3, max = 6, message = "用户名长度必须在3到6之间")
    private String name;
}
```

##### 控制层代码一定要注意`@NotNull`的导入包是否正确

> 我这`Tomcat9.x`版本的所以导入的是 `javax`包下`@NotNull`的，如果是`Tomcat 10.x`版本则需要导入的是`jakarta`包下的！

![image-20240324170510817](/images/springMvc/image-20240324170510817.png)


