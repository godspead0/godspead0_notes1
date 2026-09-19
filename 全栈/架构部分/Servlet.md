# 

# JavaWeb Servlet 全解析（基础+进阶）

Servlet 是 JavaWeb 核心组件之一，本质是运行在 Web 服务器（如 Tomcat）上的 Java 类，用于扩展 HTTP 服务器功能，处理客户端（浏览器）请求并返回响应，是客户端与服务器端数据交互的核心桥梁。

## 一、Servlet 核心基础

### 1.1 定义与核心作用

- **本质**：运行在 Web 服务器（Tomcat）中的 Java 对象，不直接独立运行，需由服务器调用
- **核心职责**：
    - 接收客户端（浏览器）的 HTTP 请求（包含请求行、请求头、请求体数据）
    - 调用后端业务逻辑处理请求数据
    - 构建 HTTP 响应（响应行、响应头、响应体）并返回给客户端
- **核心价值**：解决了静态网页无法动态处理请求的问题，实现了动态网页开发与数据交互

### 1.2 环境搭建（Maven Web 项目）

#### 1.2.1 项目创建（骨架方式）

1. 选择 Maven 骨架：`org.apache.maven.archetypes:maven-archetype-webapp`
2. 配置项目坐标（GroupId、ArtifactId、Version）
3. 选择 Maven 环境（本地仓库、settings.xml）
4. 补全项目结构：
    1. 手动创建 `src/main/java`（源代码目录）
    2. 手动创建 `src/main/resources`（资源文件目录）
    3. 确认 `webapp/WEB-INF` 目录下存在 `web.xml`（部署描述符）

#### 1.2.2 依赖配置（pom.xml）

需引入 Servlet API 依赖（Tomcat 内置该 API，但开发时需依赖编译）：

```XML
<!-- Servlet API 依赖 -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope> <!-- 编译时有效，运行时由 Tomcat 提供，避免冲突 -->
</dependency>
```

### 1.3 Servlet 实现方式

#### 1.3.1 注解方式（推荐，Servlet 3.0+ 支持）

1. 创建 Java 类，继承 `HttpServlet`（对 Servlet 接口的 HTTP 协议封装，无需重写所有接口方法）
2. 重写 `doGet()`（处理 GET 请求）和 `doPost()`（处理 POST 请求）方法
3. 添加 `@WebServlet` 注解，配置访问路径

**示例代码**：

```Java
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

// 配置访问路径：/demo（可配置多个路径）
@WebServlet("/demo")
public class ServletDemo extends HttpServlet {
    // 处理 GET 请求
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.getWriter().write("Hello Servlet!"); // 响应字符数据
    }

    // 处理 POST 请求
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        doGet(req, resp); // 复用 GET 逻辑（实际开发可按需单独实现）
    }
}
```

#### 1.3.2 XML 配置方式（兼容旧版本）

若不使用注解，可在 `web.xml` 中配置 Servlet 映射关系：

```XML
<!-- 配置 Servlet 类信息 -->
<servlet>
    <servlet-name>ServletDemo</servlet-name> <!-- 逻辑名称，需与映射一致 -->
    <servlet-class>com.example.ServletDemo</servlet-class> <!-- 全类名 -->
</servlet>

<!-- 配置访问路径映射 -->
<servlet-mapping>
    <servlet-name>ServletDemo</servlet-name> <!-- 与上方 servlet-name 对应 -->
    <url-pattern>/demo</url-pattern> <!-- 访问路径 -->
</servlet-mapping>
```

## 二、Servlet 生命周期

Servlet 由 Web 服务器（Tomcat）管理生命周期，共分为 4 个阶段，核心是「单例多线程」（一个 Servlet 类仅创建一个实例，多请求并发调用 `service()` 方法）。

### 2.1 生命周期四阶段

| 阶段         | 触发时机                                                     | 核心方法                                                    | 说明                                                         |
| ------------ | ------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ |
| 加载与实例化 | 1. 默认：第一次访问 Servlet 时<br>2. 配置 `loadOnStartup`：服务器启动时 | 类的构造方法（由容器调用）                                  | 仅执行 1 次，创建 Servlet 实例                               |
| 初始化       | 实例化后立即执行                                             | `init(ServletConfig config)`                                | 仅执行 1 次，用于初始化资源（如连接数据库、加载配置文件）    |
| 请求处理     | 每次客户端请求该 Servlet 时                                  | `service(HttpServletRequest req, HttpServletResponse resp)` | 多次执行，核心业务逻辑处理（自动根据请求方式调用 `doGet()`/`doPost()`） |
| 服务终止     | 服务器关闭（正常关闭）或应用卸载时                           | `destroy()`                                                 | 仅执行 1 次，用于释放资源（如关闭数据库连接、释放文件句柄）  |

### 2.2 关键配置：loadOnStartup

通过 `@WebServlet` 的 `loadOnStartup` 属性控制实例化时机：

- 负数（默认值）：第一次访问时实例化
- 0 或正整数：服务器启动时实例化，数字越小，创建优先级越高（例如 `loadOnStartup=1` 比 `loadOnStartup=2` 先创建）

**示例**：

```Java
// 服务器启动时创建 Servlet 实例，优先级 1
@WebServlet(value = "/demo", loadOnStartup = 1)
public class ServletDemo extends HttpServlet {
    // ...
}
```

## 三、Servlet 体系结构

Servlet 核心是接口与实现类的分层设计，开发者无需直接实现 `Servlet` 根接口，而是继承封装后的 `HttpServlet`，简化开发。

### 3.1 体系结构层级

```Plain
Servlet（根接口） ← GenericServlet（抽象类，适配 Servlet 接口，简化配置处理） ← HttpServlet（抽象类，HTTP 协议封装） ← 自定义 Servlet 类（开发者实现）
```

- **Servlet 接口**：定义核心生命周期方法（`init()`、`service()`、`destroy()`）
- **GenericServlet**：实现 `Servlet` 接口，提供 `ServletConfig` 配置获取、日志输出等通用功能
- **HttpServlet**：继承 `GenericServlet`，针对 HTTP 协议优化：
    - 重写 `service()` 方法，自动解析请求方式（GET/POST/PUT/DELETE 等）
    - 提供 `doGet()`、`doPost()` 等方法，开发者按需重写即可（无需处理请求方式判断）

## 四、Servlet 访问路径配置

一个 Servlet 可配置多个访问路径，路径规则分为 4 类，优先级：**精确匹配 > 目录匹配 > 扩展匹配 > 任意匹配**。

### 4.1 路径分类与示例

| 路径类型 | 格式规则                 | 示例                   | 说明                                                         |
| -------- | ------------------------ | ---------------------- | ------------------------------------------------------------ |
| 精确匹配 | 完整路径（不含通配符）   | `/demo`、`/user/login` | 仅匹配指定路径，优先级最高                                   |
| 目录匹配 | 以 `/` 开头，以 `*` 结尾 | `/user/*`、`/admin/*`  | 匹配指定目录下的所有请求（如 `/user/list`、`/user/add` 均匹配 `/user/*`） |
| 扩展匹配 | 以 `*` 开头，后跟扩展名  | `*.do`、`*.action`     | 匹配指定扩展名的请求（如 `/login.do`、`/user/list.do` 均匹配 `*.do`） |
| 任意匹配 | `/` 或 `/*`              | `/`、`/*`              | - `/`：覆盖 Tomcat 默认的 DefaultServlet，处理所有未匹配的请求（包括静态资源）<br>- `/*`：匹配所有请求（包括其他 Servlet 路径），慎用 |

### 4.2 多路径配置

通过 `@WebServlet` 的 `value` 或 `urlPatterns` 属性配置多个路径：

```Java
// 多个路径均可访问该 Servlet
@WebServlet(urlPatterns = {"/demo", "/test", "/user/info"})
public class ServletDemo extends HttpServlet {
    // ...
}
```

### 4.3 静态资源访问问题

- 若 Servlet 路径配置为 `/`，会覆盖 Tomcat 内置的 `DefaultServlet`（负责处理静态资源如 HTML、CSS、JS、图片等），导致静态资源无法访问。
- 解决方案：
    - 避免使用 `/` 作为 Servlet 路径，改用精确匹配或目录匹配
    - 若需使用 `/`，需手动配置静态资源映射（如 SpringMVC 中的 `<mvc:resources>`，或 Tomcat 配置）

## 五、Request 对象（获取请求数据）

`HttpServletRequest` 封装了客户端的 HTTP 请求信息，提供一系列方法获取请求行、请求头、请求体数据，是 Servlet 接收请求的核心对象。

### 5.1 继承体系

```Plain
ServletRequest（根接口） ← HttpServletRequest（HTTP 协议专用接口） ← RequestFacade（Tomcat 实现类，由服务器创建并传入方法）
```

开发者无需关注实现类，直接通过方法参数使用 `HttpServletRequest` 即可。

### 5.2 核心方法（获取请求数据）

#### 5.2.1 获取请求行数据

请求行格式：`请求方法 请求URI?查询字符串 HTTP/协议版本`（例：`GET /demo?name=zhangsan HTTP/1.1`）

| 方法                           | 作用                                    | 示例结果                              |
| ------------------------------ | --------------------------------------- | ------------------------------------- |
| `String getMethod()`           | 获取请求方法（GET/POST/PUT 等）         | `GET`                                 |
| `String getRequestURI()`       | 获取请求 URI（虚拟目录+资源路径）       | `/web-demo/demo`                      |
| `String getContextPath()`      | 获取项目虚拟目录（部署时配置）          | `/web-demo`                           |
| `StringBuffer getRequestURL()` | 获取完整请求 URL（含协议、域名、端口）  | `http://localhost:8080/web-demo/demo` |
| `String getQueryString()`      | 获取 URL 中的查询字符串（GET 请求参数） | `name=zhangsan&age=20`                |

#### 5.2.2 获取请求头数据

请求头是客户端向服务器传递的附加信息（如浏览器类型、请求编码、Cookie 等）：

| 方法                                   | 作用               | 示例                                   |
| -------------------------------------- | ------------------ | -------------------------------------- |
| `String getHeader(String name)`        | 根据头名称获取头值 | `getHeader("User-Agent")` → 浏览器类型 |
| `Enumeration<String> getHeaderNames()` | 获取所有请求头名称 | 遍历所有头信息                         |

**示例**：获取浏览器类型

```Java
String userAgent = req.getHeader("User-Agent");
System.out.println("浏览器类型：" + userAgent);
```

#### 5.2.3 获取请求体数据

请求体用于存储 POST 请求的参数（如表单提交、JSON 数据），GET 请求无请求体（参数在 URL 中）：

| 方法                                  | 作用                           | 适用场景                                                   |
| ------------------------------------- | ------------------------------ | ---------------------------------------------------------- |
| `BufferedReader getReader()`          | 获取字符流（按行读取文本数据） | 表单提交（`application/x-www-form-urlencoded`）、JSON 文本 |
| `ServletInputStream getInputStream()` | 获取字节流                     | 文件上传（`multipart/form-data`）、二进制数据              |

#### 5.2.4 通用参数获取（GET/POST 兼容）

Request 对象会自动封装所有请求参数（GET 从查询字符串、POST 从请求体）到一个 Map 中，提供通用方法获取，无需区分请求方式：

| 方法                                       | 作用                             | 示例                                                    |
| ------------------------------------------ | -------------------------------- | ------------------------------------------------------- |
| `String getParameter(String name)`         | 根据参数名获取单个值             | `req.getParameter("name")` → `zhangsan`                 |
| `String[] getParameterValues(String name)` | 根据参数名获取多个值（如复选框） | `req.getParameterValues("hobby")` → `["game", "music"]` |
| `Map<String, String[]> getParameterMap()`  | 获取所有参数的 Map 集合          | 遍历所有参数                                            |

**示例**：获取表单参数

```Java
// 表单提交：name=zhangsan&age=20&hobby=game&hobby=music
String name = req.getParameter("name"); // zhangsan
String age = req.getParameter("age"); // 20
String[] hobbies = req.getParameterValues("hobby"); // ["game", "music"]
Map<String, String[]> paramMap = req.getParameterMap(); // 所有参数
```

### 5.3 乱码问题解决

#### 5.3.1 POST 请求乱码

POST 请求参数在请求体中，默认编码为 `ISO-8859-1`（不支持中文），需在获取参数前设置编码：

```Java
// 设置请求体编码（仅对 POST 有效）
req.setCharacterEncoding("UTF-8");
```

#### 5.3.2 GET 请求乱码（Tomcat 8+ 已解决）

- Tomcat 8 及以上版本：默认对 GET 请求的查询字符串使用 `UTF-8` 编码，无需处理
- Tomcat 7 及以下版本：需修改 `conf/server.xml` 配置，添加 `URIEncoding="UTF-8"`：
    - ```XML
        <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" URIEncoding="UTF-8"/>
        ```

### 5.4 请求转发（服务器内部跳转）

请求转发是服务器内部的资源跳转方式，通过 `Request` 对象实现，适用于多个资源协同处理请求。

#### 5.4.1 实现方式

```Java
// 1. 获取转发器（目标资源路径，无需虚拟目录）
RequestDispatcher dispatcher = req.getRequestDispatcher("/target");
// 2. 执行转发（传递 request 和 response 对象）
dispatcher.forward(req, resp);
```

#### 5.4.2 核心特点

1. 浏览器地址栏路径**不发生变化**（始终显示第一次请求的路径）
2. 仅能转发到**当前服务器的内部资源**（不能转发到外部网站）
3. 本质是**一次请求**（request 和 response 对象被复用）
4. 可通过 `Request` 域传递数据（多个资源共享数据）

#### 5.4.3 Request 域对象（数据传递）

Request 域是基于一次请求的临时数据存储区域，适用于转发场景下的资源间数据共享：

| 方法                                           | 作用           |
| ---------------------------------------------- | -------------- |
| `void setAttribute(String name, Object value)` | 存储数据到域中 |
| `Object getAttribute(String name)`             | 从域中获取数据 |
| `void removeAttribute(String name)`            | 从域中删除数据 |

**示例**：转发时传递数据

```Java
// 源 Servlet
req.setAttribute("msg", "转发成功！"); // 存储数据
req.getRequestDispatcher("/target").forward(req, resp); // 转发

// 目标 Servlet
String msg = (String) req.getAttribute("msg"); // 获取数据
resp.getWriter().write(msg); // 输出：转发成功！
```

## 六、Response 对象（设置响应数据）

`HttpServletResponse` 封装了服务器对客户端的 HTTP 响应信息，用于设置响应行、响应头、响应体，最终返回给客户端。

### 6.1 核心方法（设置响应数据）

#### 6.1.1 设置响应行

响应行格式：`HTTP/协议版本 状态码 状态描述`（例：`HTTP/1.1 200 OK`）

| 方法                     | 作用           | 示例                                                   |
| ------------------------ | -------------- | ------------------------------------------------------ |
| `void setStatus(int sc)` | 设置响应状态码 | `setState(200)`（成功）、`setState(404)`（资源未找到） |

#### 6.1.2 设置响应头

响应头是服务器向客户端传递的附加信息（如响应编码、重定向地址、缓存策略等）：

| 方法                                        | 作用                       | 示例                                                   |
| ------------------------------------------- | -------------------------- | ------------------------------------------------------ |
| `void setHeader(String name, String value)` | 设置响应头（覆盖原有值）   | `setHeader("Content-Type", "text/html;charset=utf-8")` |
| `void addHeader(String name, String value)` | 添加响应头（不覆盖原有值） | `addHeader("Set-Cookie", "username=zhangsan")`         |

#### 6.1.3 设置响应体

响应体是服务器返回给客户端的核心数据（如 HTML 页面、JSON 数据、图片等）：

| 方法                                    | 作用           | 适用场景                               |
| --------------------------------------- | -------------- | -------------------------------------- |
| `PrintWriter getWriter()`               | 获取字符输出流 | 输出文本数据（HTML、JSON、普通字符串） |
| `ServletOutputStream getOutputStream()` | 获取字节输出流 | 输出二进制数据（图片、文件下载）       |

### 6.2 响应乱码解决

响应数据默认编码为 `ISO-8859-1`（不支持中文），需通过响应头设置编码：

```Java
// 方式 1：设置响应头（推荐，同时指定客户端解码方式）
resp.setHeader("Content-Type", "text/html;charset=utf-8");

// 方式 2：简化方法（等价于方式 1）
resp.setContentType("text/html;charset=utf-8");
```

注意：需在调用 `getWriter()` 或 `getOutputStream()` 前设置，否则无效。

### 6.3 重定向（客户端跳转）

重定向是服务器通知客户端重新向新地址发送请求的跳转方式，适用于资源位置变更或跨资源跳转。

#### 6.3.1 实现方式

```Java
// 方式 1：手动设置状态码和响应头
resp.setStatus(302); // 302 是重定向状态码
resp.setHeader("Location", "/web-demo/target"); // 目标资源路径（需含虚拟目录）

// 方式 2：简化方法（推荐）
resp.sendRedirect("/web-demo/target");
```

#### 6.3.2 核心特点

1. 浏览器地址栏路径**发生变化**（显示目标资源路径）
2. 可重定向到**任意资源**（当前服务器内部或外部网站，如 `https://www.baidu.com`）
3. 本质是**两次请求**（第一次请求的 request 和 response 对象被销毁，第二次请求是新的对象）
4. 不能通过 Request 域传递数据（两次请求是独立的）

### 6.4 路径问题总结

在 Servlet 开发中，路径是否需要包含虚拟目录（项目部署名），取决于使用场景：

| 使用场景                                | 是否需要虚拟目录                   | 示例                                    |
| --------------------------------------- | ---------------------------------- | --------------------------------------- |
| 浏览器端跳转（重定向 `sendRedirect()`） | 是（客户端需知道完整路径）         | `resp.sendRedirect("/web-demo/target")` |
| 服务器端跳转（请求转发 `forward()`）    | 否（服务器内部资源，无需虚拟目录） | `req.getRequestDispatcher("/target")`   |
| HTML 中的路径（如 `<a>`、`<form>`）     | 是（客户端发起请求）               | `<a href="/web-demo/demo">跳转</a>`     |

优化方案：可通过 `req.getContextPath()` 获取虚拟目录，避免硬编码：

```Java
String contextPath = req.getContextPath(); // /web-demo
resp.sendRedirect(contextPath + "/target"); // 动态拼接虚拟目录
```

## 七、Servlet 核心区别与对比

### 7.1 请求转发 vs 重定向

| 对比维度   | 请求转发（forward）   | 重定向（redirect）       |
| ---------- | --------------------- | ------------------------ |
| 地址栏路径 | 不变                  | 改变                     |
| 请求次数   | 1 次                  | 2 次                     |
| 数据共享   | 可通过 Request 域共享 | 不可共享（两次独立请求） |
| 跳转范围   | 仅当前服务器内部      | 任意资源（内部/外部）    |
| 路径要求   | 无需虚拟目录          | 需包含虚拟目录           |

### 7.2 doGet() vs doPost()

| 对比维度     | doGet()                        | doPost()                                 |
| ------------ | ------------------------------ | ---------------------------------------- |
| 请求参数位置 | URL 查询字符串                 | 请求体                                   |
| 参数长度限制 | 有限制（取决于浏览器和服务器） | 无限制                                   |
| 安全性       | 低（参数明文显示在 URL 中）    | 高（参数在请求体中，需加密可进一步提升） |
| 适用场景     | 查询数据（如搜索、列表查询）   | 提交数据（如表单提交、文件上传、登录）   |

## 八、常见问题与注意事项

1. **Servlet 单例多线程安全问题**：
    1. 问题：多个请求并发调用 `service()` 方法，若 Servlet 类中有成员变量，会导致线程安全问题
    2. 解决方案：避免在 Servlet 中定义成员变量，使用局部变量（每个线程独立）或线程安全的对象（如 `ThreadLocal`）
2. **流关闭问题**：
    1. Response 的 `PrintWriter` 和 `ServletOutputStream` 无需手动关闭，服务器响应结束后会自动关闭
    2. 若手动关闭，可能导致后续响应数据无法输出
3. **404 错误排查**：
    1. 检查访问路径是否正确（是否包含虚拟目录、路径拼写错误）
    2. 检查 Servlet 类是否添加 `@WebServlet` 注解或 XML 配置是否正确
    3. 检查项目是否已部署到 Tomcat 且启动成功
4. **500 错误排查**：
    1. 查看 Tomcat 日志（`logs/catalina.out`），定位代码异常位置
    2. 常见原因：空指针异常、类找不到异常、数据库连接异常等
5. **静态资源无法访问**：
    1. 检查 Servlet 路径是否配置为 `/`（覆盖了 DefaultServlet）
    2. 解决方案：修改 Servlet 路径为精确匹配或目录匹配，避免使用 `/`