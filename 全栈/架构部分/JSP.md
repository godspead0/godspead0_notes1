# JSP 核心知识点总结

# JSP 技术核心知识点总结

JSP（Java Server Pages）是Java的服务端界面技术，核心是将Java代码与HTML页面合体，实现动态网页开发。

## 一、环境准备

使用JSP前需在`pom.xml`中添加相关依赖（Maven项目），随后创建`.jsp`文件即可开发：

```XML
<!-- JSP核心依赖示例 -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope> <!-- 由Tomcat等服务器提供，无需打包 -->
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```

## 二、JSP本质

JSP的本质是**Servlet**，其运行流程如下：

1. 当JSP页面被首次访问时，Tomcat服务器会将`.jsp`文件转换为Java源文件（Servlet）；
2. Tomcat对生成的Java文件进行编译，生成`.class`字节码文件；
3. 服务器执行字节码文件，处理请求并生成响应（动态HTML），返回给客户端。

## 三、JSP脚本分类（核心语法）

JSP提供三种脚本标签，用于在HTML中嵌入Java代码，不同标签的作用范围和执行方式不同：

| 脚本标签 | 作用说明                                                     | 代码示例                                                     |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `<% %>`  | 脚本片段：内容直接嵌入到生成的Servlet的`_jspService()`方法中，可编写任意Java代码（变量、循环、判断等） | `<% int num = 10; for(int i=0; i<num; i++){ System.out.println(i); } %>` |
| `<%= %>` | 输出表达式：内容会被包裹到`out.print()`中，作为输出参数直接显示在页面上（**无分号结尾**） | `<%= "当前时间：" + new Date() %>`（页面直接显示当前时间）   |
| `<%! %>` | 声明标签：内容被定义在`_jspService()`方法之外，属于Servlet类的成员（可定义成员变量、成员方法、静态代码块等） | `<%! private String name = "JSP"; public String getName(){ return name; } %>` |

⚠️ 注意：

- `<%= %>` 中不能加分号（`;`），否则编译报错；
- `<%! %>` 定义的是类成员，多线程环境下成员变量可能存在线程安全问题（慎用）。

## 四、EL表达式（Expression Language）

### 1. 作用

简化JSP中Java代码的编写，用于**获取四大域对象中存储的数据**，替代繁琐的`request.getAttribute()`等代码。

### 2. 语法格式

```Java
${ 域中数据的键名 }
```

### 3. 四大域对象（数据存储范围）

EL表达式默认按「从小到大」的范围查找数据，找到即返回；若未找到则返回空字符串（而非`null`）：

| 域对象             | 作用范围                             | 对应的EL隐式对象 | 说明                     |
| ------------------ | ------------------------------------ | ---------------- | ------------------------ |
| PageContext        | 当前JSP页面（仅自身可见）            | pageScope        | 最小范围，页面跳转后失效 |
| HttpServletRequest | 一次请求（请求转发有效，重定向失效） | requestScope     | 多页面共享一次请求数据   |
| HttpSession        | 一次会话（浏览器打开到关闭）         | sessionScope     | 同一浏览器多次请求共享   |
| ServletContext     | 整个Web应用（服务器启动到关闭）      | applicationScope | 所有用户共享应用数据     |

### 示例

```Java
<!-- 1. 向request域存入数据（Java脚本） -->
<% request.setAttribute("username", "张三"); %>

<!-- 2. EL表达式获取request域数据（简化写法） -->
<div>用户名：${username}</div>

<!-- 3. 指定域获取（避免重名） -->
<div>用户名（指定request域）：${requestScope.username}</div>
```

## 五、JSTL（JSP Standard Tag Library）

### 1. 简介

JSP标准标签库，提供一系列标签替代Java脚本，简化循环、判断、数据格式化等操作，使JSP页面更简洁易维护。

### 2. 使用步骤

1. **添加依赖**（pom.xml）：

```XML
<!-- JSTL核心标签库依赖 -->
<dependency>
    <groupId>jstl</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>taglibs</groupId>
    <artifactId>standard</artifactId>
    <version>1.1.2</version>
</dependency>
```

1. **在JSP页面引入标签库**：

```Java
<!-- 引入核心标签库（常用：循环、判断等） -->
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!-- 引入格式化标签库（日期、数字格式化） -->
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```

- `prefix="c"`：自定义标签前缀（通常用`c`表示core核心库）；
- `uri`：标签库的唯一标识，用于定位标签库资源。

### 3. 常用JSTL标签示例

#### （1）循环标签 `<c:forEach>`

遍历集合或数组：

```Java
<!-- 遍历request域中的list集合 -->
<% 
    List<String> cities = new ArrayList<>();
    cities.add("北京");
    cities.add("上海");
    request.setAttribute("cities", cities);
%>

<c:forEach items="${cities}" var="city">
    <div>${city}</div> <!-- 输出：北京、上海 -->
</c:forEach>
```

#### （2）判断标签 `<c:if>`

条件判断（无else，可嵌套）：

```Java
<c:if test="${username == '张三'}">
    <div>欢迎管理员登录！</div>
</c:if>
```

#### （3）选择标签 `<c:choose>`+`<c:when>`+`<c:otherwise>`

多条件判断（类似Java的`if-else if-else`）：

```Java
<c:choose>
    <c:when test="${score >= 90}">优秀</c:when>
    <c:when test="${score >= 60}">及格</c:when>
    <c:otherwise>不及格</c:otherwise>
</c:choose>
```

### 4. 学习资源

JSTL详细用法可参考视频教程：[JSTL标签库详解](https://www.bilibili.com/video/BV1Qf4y1T7Hx?t=88.1&p=116)

## 核心总结

1. JSP = HTML + Java脚本 + EL表达式 + JSTL标签；
2. 运行本质：JSP → Servlet（Java文件）→ 编译为.class → 执行；
3. 数据存储：优先使用`request`/`session`域，根据数据范围选择合适的域对象；
4. 最佳实践：尽量减少JSP中的Java脚本，用EL表达式获取数据，JSTL标签实现逻辑控制，使页面结构更清晰。