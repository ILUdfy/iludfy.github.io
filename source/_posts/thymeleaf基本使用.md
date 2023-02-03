---
title: thymeleaf使用
date: 2023-01-20 17:33:29
tags: thymeleaf
categories: thymeleaf
---
# thymeleaf基本使用

## 一、引入依赖

Spring Boot项目中引入依赖：

```XML
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 二、配置文件

在aplication.yml文件中写入如下配置：

```yml
spring:
   thymeleaf:
     mode: LEGACYHTML5
     encoding: UTF-8    # 编码格式
     prefix: classpath:/template/  # 静态页面所在的路径，一般在resources文件加下创建
     suffix: .html   # 页面后缀
     cache: false   # 关闭缓存，开发时可以看到实时页面
```
## 三、编写控制层

控制层在返回页面的同时将键值对传给前端，给前端传值有四种方法：

通过Model对象：
```java
@Controller
public class ViewTestController {
    @GetMapping("/hello")
    public String hello(Model model){
        model.addAttribute("msg","这世上没有纯粹的自由，风也会有吹到头的时候");
        model.addAttribute("link","http://www.baidu.com");
        return "success";
    }
}
```


通过ModelAndView对象：
```java
@Controller
public class ViewTestController {
    @GetMapping("/hello")
    public String hello(ModelAndView model){
        model.addAttribute("msg","这世上没有纯粹的自由，风也会有吹到头的时候");
        model.addAttribute("link","http://www.baidu.com");
        return "success";
    }
}
```

通过HttpServletRequest对象：
```java
@Controller
public class ViewTestController {
    @GetMapping("/hello")
    public String hello(HttpServletRequest request){
        request.setAttribute("msg","这世上没有纯粹的自由，风也会有吹到头的时候");
        request.setAttribute("link","http://www.baidu.com");
        return "success";
    }
}
```

通过Map对象：
```java
@Controller
public class ViewTestController {
    @GetMapping("/hello")
    public String hello(Map<String, String> map){
        map.put("msg","这世上没有纯粹的自由，风也会有吹到头的时候");
        map.put("link","http://www.baidu.com");
        return "success";
    }
}
```
## 四、基础语法

### 编写html
建议在html标签上写入：
```HTML
<html xmlns:th="http://www.thymeleaf.org">
```

不加不影响thymeleaf正常使用，但是加入以后开发过程中th:会自动弹出提示

`/templates/success.html`：
```HTML
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">nice</h1>
<h2>
    <a href="www.baidu.com" th:href="${link}">去百度</a>  <br/>
    <a href="www.google.com" th:href="@{/link}">去百度</a>
</h2>
</body>
</html>
```

### 基本语法

#### 表达式

| 表达式名字 | 语法   |                用途                |
| ---------- | ------ | :--------------------------------: |
| 变量取值   | ${...} |  获取请求域、session域、对象等值   |
| 选择变量   | *{...} |          获取上下文对象值          |
| 消息       | #{...} |           获取国际化等值           |
| 链接       | @{...} |              生成链接              |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |

##### 获取变量值${...}
```HTML
    <h1 th:text="${msg}">nice</h1>
```
上面代码默认从request作用域取值，若未取到则为null
若从session域和application域取值，分别加上前缀即可：
```HTML
    <h1 th:text="${session.msg}">nice</h1> <!--从session域取值  -->
```
```HTML
    <h1 th:text="${application.msg}">nice</h1> <!-- 从application域取值 -->
```
##### 选择变量表达式*{...}
通常结合th:object使用。某一标签使用th:object标签声明一个变量后，在其子标签内都可以通过选择变量表达式*{...}来取值，示例如下：
```HTML
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p> 
    <p>Nationality: <span th:text={nationality}">Saturn</span>.</p>
</div> 
```
上述代码等价于：
```HTML
<div>
    <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p> 
    <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p> 
    <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
</div>
```
若父标签未使用th:object声明任何变量，那么*{...}和${...}完全等价。

##### 链接表达式@{...}
一般用于页面跳转和静态资源的引用，可搭配th:src,th:href,th:action等标签使用。
```HTML
<!-- 静态资源引用 -->
<link rel="icon" th:href="@{/images/favicon.ico}" type="image/ico" />

<script th:src="@{/vendors/jquery/dist/jquery.min.js}"></script>

<!-- 页面跳转 -->
<a th:href="@{/index}">首页</a>
```
若希望页面跳转时携带参数，可使用@{/URL(K1=V1,K2=V2...)}的格式
```HTML
<!-- 通过链接跳转为GET请求 -->
<a th:href="@{/index(id=${id},pageNum=1)}">首页</a> 
```

链接表达式中写的是在项目中的相对路径，thymeleaf会自动把项目根路径补全在表达式的前面。例如，你的项目路径是http://localhost:8080/myapp,那么th:href="@{/index}"会被解析成href="http://localhost:8080/myapp/index"。

推荐使用链接表达式@{...}来进行静态资源的引用、页面跳转和表单提交，这样就不需要关心项目的根路径，只需要写入相对路径即可，尤其在rest风格中，必须使用。

##### 片段表达式~{...}
通常用于模板布局，搭配th:fragement,th:include,th:insert,th:remove等标签使用。
详细教程可以参考博客<https://blog.csdn.net/wangmx1993328/article/details/84747497>


#### 字面量

- 文本值: **'one text'** **,** **'Another one!'** **,…**
- 数字: **0** **,** **34** **,** **3.0** **,** **12.3** **,…**
- 布尔值: **true** **,** **false**
- 空值: **null**
- 变量： one，two，.... 变量不能有空格

#### 文本操作

- 字符串拼接: **+**
```HTML
<a th:href="@{${baseURL}+'/add'}">新增</a>
```
- 变量替换: **|The name is ${name}|** 
```HTML
<!-- 二者等价（只能包含表达式变量，而不能有条件判断等！) -->
<h1 th:text="'后端发来的消息：'+${msg}">nice</h1>
<h1 th:text="|后端发来的消息：${msg}|">nice</h1>
```

#### 数学运算

- 运算符: + , - , * , / , %
```HTML
<th scope="row" th:text="${objStat.index}+*{startRow}">1</th>
```

#### 布尔运算

- 运算符:  **and** **,** **or**
- 一元运算: **!** **,** **not** 

#### 比较运算

- 比较: **>** **,** **<** **,** **>=** **,** **<=** **(** **gt** **,** **lt** **,** **ge** **,** **le** **)**
- 等式: **==** **,** **!=** **(** **eq** **,** **ne** **)** 
```HTML
<option value="10" th:selected="*{pageSize} eq '10'">10</option>
```

#### 条件运算

- If-then: **(if) ? (then)**
- If-then-else: **(if) ? (then) : (else)**
- Default: (value) **?: (defaultvalue)** 
```HTML
<div class="row" th:with="feature=${feature} == null or ${feature} == '' ? '':'/'+${feature}">
```

#### 特殊操作

- 无操作： _

### 设置属性值-th:attr

- 设置单个值

```HTML
<form action="subscribe.html" th:attr="action=@{/subscribe}">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
  </fieldset>
</form>
```

- 设置多个值

```HTML
<img src="../../images/gtvglogo.png"  
     th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

[官方文档 - 5 Setting Attribute Values](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#setting-attribute-values)

### 迭代
* 基本语法：
```HTML
<div th:each="变量名 : 集合"> 
	<p th:text="${变量名}"></p> 
</div>
```

```HTML
<tr th:each="prod : ${prods}">
    <td th:text="${prod.name}">Onions</td>
    <td th:text="${prod.price}">2.41</td>
    <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```
* 迭代状态变量的使用：
```HTML
<div th:each = "变量名，状态变量名 : 集合" > 
	<p th:text = "${状态变量.属性}" ></p> 
</div>
```




```HTML
<tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
    <td th:text="${prod.name}">Onions</td>
    <td th:text="${prod.price}">2.41</td>
    <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```

>注：如果缺省状态变量名，则迭代器会 默认以变量名开头的状态变量 xxxStat

>状态变量的属性
index：当前迭代对象的序号，从0开始，这是索引属性
count：当前迭代对象的序号，从1开始，这个是统计属性
size：迭代变量元素的总量，这是被迭代对象的大小属性
even/odd：布尔值，当前循环是否是偶数/奇数（从0开始计算）
first：布尔值，当前循环是否是第一个
last：布尔值，当前循环是否是最后一个
current：当前迭代变量

### 条件运算

```HTML
<a href="comments.html"
	th:href="@{/product/comments(prodId=${prod.id})}"
	th:if="${not #lists.isEmpty(prod.comments)}">view</a>
```
th:if标签：当条件成立时，该标签及其子标签才会存在
th:unless标签：与th:if标签相反，条件不成立时存在

```HTML
<div th:switch="${user.role}">
      <p th:case="'admin'">User is an administrator</p>
      <p th:case="#{roles.manager}">User is a manager</p>
      <p th:case="*">User is some other thing</p>
</div>
```

### 属性优先级

| Order | Feature                         | Attributes                                 |
| :---- | :------------------------------ | :----------------------------------------- |
| 1     | Fragment inclusion              | `th:insert` `th:replace`                   |
| 2     | Fragment iteration              | `th:each`                                  |
| 3     | Conditional evaluation          | `th:if` `th:unless` `th:switch` `th:case`  |
| 4     | Local variable definition       | `th:object` `th:with`                      |
| 5     | General attribute modification  | `th:attr` `th:attrprepend` `th:attrappend` |
| 6     | Specific attribute modification | `th:value` `th:href` `th:src` `...`        |
| 7     | Text (tag body modification)    | `th:text` `th:utext`                       |
| 8     | Fragment specification          | `th:fragment`                              |
| 9     | Fragment removal                | `th:remove`                                |

[官方文档 - 10 Attribute Precedence](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#attribute-precedence)

### thymeleaf常用标签
![thymeleaf常用标签](thymeleaf1.png)

### 在js中使用thymeleaf
模板引擎除了直接渲染页面之外还可以在js中进行预处理，其中的thymeleaf代码可以先注释起来，静态时默认使用注释后面的默认值：
```javascript
<script th:inline="javascript">
    var msg = /*[[thymeleaf]]*/静态默认值
</script>
```