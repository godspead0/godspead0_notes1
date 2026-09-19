# **主题**：Web 核心知识与 Spring 分层开发实战指南（重点补充 Cookie 与 Session），包括 HTTP 协议基础（请求和响应数据格式）、Java Web 中请求/响应封装，以及 Cookie 和 Session 的核心知识（定义、特性、生命周期、配置等）、对比与选型

# Web 核心知识与 Spring 分层开发实战指南（补充：Cookie 与 Session）

## 一、HTTP 协议基础

HTTP（HyperText Transfer Protocol，超文本传输协议）是前后端通信的核心协议，定义了客户端与服务器之间请求和响应的数据格式、传输规则，基于 TCP/IP 协议族，属于**无状态协议**（每次请求独立，不保留上下文）。

无状态协议的核心问题：无法识别连续请求的关联性（如用户登录后，后续请求无法证明“已登录”）。为解决该问题，诞生了 **Cookie** 和 **Session** 两种状态保持机制。

### 1.1 请求数据格式

HTTP 请求由 **请求行、请求头、请求体** 三部分组成，三部分之间以空行（`\r\n`）分隔，缺一不可。这是 HTTP 协议的强制分隔规则，少了会导致服务器解析失败。

#### 1.1.1 核心组成

| 部分   | 格式说明                                                     | 示例                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 请求行 | `请求方式 + 资源路径 + 协议版本`（空格分隔），是请求的核心标识 | `POST /api/user/add HTTP/1.1`                                |
| 请求头 | 键值对格式（`Key: Value`），传递客户端环境、请求附加信息，可多个 | `Host: localhost:8080`<br>`Content-Type: application/json`<br>`Cookie: userId=1; username=test` |
| 请求体 | 存放请求的核心数据（如表单、JSON），仅特定请求方式支持       | `{"username":"test","password":"123456"}`                    |

#### 1.1.2 关键细节

- **请求方式分类**：
    - 安全请求（不修改服务器数据）：`GET`（查询）、`HEAD`（仅返回响应头）、`OPTIONS`（预检请求）；
    - 非安全请求（修改服务器数据）：`POST`（新增）、`PUT`（全量更新）、`PATCH`（部分更新）、`DELETE`（删除）。
- **请求体使用限制**：
    - `GET` 请求无请求体，参数通过 URL 拼接（`?key1=value1&key2=value2`），长度受浏览器限制（通常 2KB-8KB）；
    - `POST/PUT/PATCH` 支持请求体，可传输大量数据（如 JSON、文件流），大小无严格限制（由服务器配置决定）。
- **核心请求头说明**：
    - `Content-Type`：指定请求体格式（`application/json` 为 JSON 数据，`application/x-www-form-urlencoded` 为表单数据，`multipart/form-data` 为文件上传）；
    - `Authorization`：身份验证信息（如 JWT Token，格式 `Bearer <token>`）；
    - `Accept`：客户端可接收的响应数据格式（如 `application/json`）；
    - `Cookie`：客户端携带的 Cookie 数据（键值对用分号分隔，是状态保持的核心请求头）。

### 1.2 响应数据格式

HTTP 响应与请求对应，由 **响应行、响应头、响应体** 三部分组成，同样以空行分隔。

#### 1.2.1 核心组成

| 部分   | 格式说明                                                     | 示例                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 响应行 | `协议版本 + 状态码 + 状态描述`（空格分隔），标识响应结果     | `HTTP/1.1 200 OK`                                            |
| 响应头 | 键值对格式，传递服务器环境、响应附加信息                     | `Content-Type: application/json`<br>`Set-Cookie: userId=1; Max-Age=3600; Path=/` |
| 响应体 | 服务器返回的实际数据（如 JSON、HTML、图片流），是前后端交互的核心内容 | `{"code":200,"msg":"success","data":{"id":1,"username":"test"}}` |

#### 1.2.2 关键细节

- **状态码分类**（核心状态码必须掌握）：
    - 2xx 成功：`200 OK`（请求成功）、`201 Created`（资源创建成功）；
    - 3xx 重定向：`301 永久重定向`、`302 临时重定向`、`304 Not Modified`（缓存命中）；
    - 4xx 客户端错误：`400 Bad Request`（请求参数错误）、`401 Unauthorized`（未授权）、`403 Forbidden`（权限不足）、`404 Not Found`（资源不存在）；
    - 5xx 服务器错误：`500 Internal Server Error`（服务器内部错误）、`503 Service Unavailable`（服务不可用）。
- **核心响应头说明**：
    - `Content-Type`：指定响应体格式（与请求头对应，前端据此解析数据）；
    - `Set-Cookie`：服务器向客户端设置 Cookie 的核心响应头（包含 Cookie 键值、生命周期、作用域等配置）；
    - `Access-Control-Allow-Origin`：解决跨域问题（指定允许访问的客户端域名）。

### 1.3 Java Web 中的请求/响应封装

在 Java Web（Spring MVC）中，HTTP 请求和响应会被自动封装为以下对象：

- **`HttpServletRequest`**：封装所有请求信息，可通过其获取：
    - 请求方式：`request.getMethod()`；
    - 资源路径：`request.getRequestURI()`；
    - 请求参数：`request.getParameter("key")`（表单参数）、`request.getInputStream()`（请求体流）；
    - 请求头：`request.getHeader("Content-Type")`；
    - Cookie 数据：`request.getCookies()`（获取客户端携带的所有 Cookie）。
- **`HttpServletResponse`**：封装响应信息，可通过其设置：
    - 状态码：`response.setStatus(200)`；
    - 响应头：`response.setHeader("Content-Type", "application/json")`；
    - 响应体：`response.getWriter().write("json字符串")`；
    - 设置 Cookie：`response.addCookie(Cookie cookie)`（向客户端写入 Cookie）。

## 二、状态保持机制：Cookie 与 Session 详解

HTTP 协议的无状态性导致无法维持用户会话（如登录后后续请求需重复验证），Cookie 和 Session 是解决该问题的核心方案，二者协同工作实现“状态保持”。

### 2.1 Cookie 核心知识

#### 2.1.1 定义与本质

Cookie 是**客户端浏览器端的小型文本数据存储机制**，基于 HTTP 协议的 `Set-Cookie` 响应头和 `Cookie` 请求头实现：

- 服务器通过 `Set-Cookie` 响应头向客户端发送 Cookie 数据；
- 客户端（浏览器）将 Cookie 存储在本地（内存或硬盘）；
- 后续客户端向同一服务器发送请求时，会自动在 `Cookie` 请求头中携带该 Cookie 数据，供服务器识别用户身份。

#### 2.1.2 Cookie 的核心特性

1. **存储位置**：客户端浏览器（内存/硬盘，由生命周期决定）；
2. **数据格式**：键值对（`key=value`），多个 Cookie 用分号 `;` 分隔；
3. **大小限制**：单个 Cookie 容量通常不超过 4KB（不同浏览器略有差异）；
4. **数量限制**：单个域名下的 Cookie 数量通常不超过 50 个（浏览器限制）；
5. **安全性**：默认明文存储（敏感数据需加密），可被客户端修改（需谨慎使用）；
6. **跨域限制**：Cookie 遵循“同源策略”，仅向设置它的域名发送（如 `www.baidu.com` 的 Cookie 不会发送给 `www.google.com`）。

#### 2.1.3 Cookie 生命周期（`setMaxAge(int seconds)`）

Cookie 的生命周期决定了其存储位置和存活时间，通过 `Cookie.setMaxAge(int seconds)` 方法设置（单位：秒），不同参数值对应不同行为：

| `setMaxAge` 参数值 | 生命周期说明                                                 | 存储位置 | 典型场景                   |
| ------------------ | ------------------------------------------------------------ | -------- | -------------------------- |
| 正数（如 3600）    | Cookie 存活指定秒数，到期后自动删除                          | 硬盘     | 记住登录状态（1 小时有效） |
| 负数（默认 -1）    | Cookie 仅在当前浏览器会话有效，浏览器关闭后立即销毁（会话级 Cookie） | 内存     | 临时会话保持（无需持久化） |
| 0                  | 立即删除客户端已存储的该 Cookie（强制失效）                  | -        | 退出登录（清除登录状态）   |

注意：若不调用 `setMaxAge` 方法，Cookie 默认为会话级（`maxAge=-1`），浏览器关闭后即消失。

#### 2.1.4 Cookie 核心配置（Java 代码示例）

在 Spring MVC 中，通过 `javax.servlet.http.Cookie` 类创建 Cookie 并配置核心属性：

```Java
@RestController
public class CookieController {
    // 向客户端设置 Cookie
    @GetMapping("/set-cookie")
    public String setCookie(HttpServletResponse response) {
        // 1. 创建 Cookie（键值对）
        Cookie userIdCookie = new Cookie("userId", "1001");
        Cookie usernameCookie = new Cookie("username", "test_user");
        
        // 2. 配置生命周期（1 小时 = 3600 秒）
        userIdCookie.setMaxAge(3600);
        // 3. 配置作用路径（仅 /api 路径下的请求会携带该 Cookie）
        userIdCookie.setPath("/api");
        // 4. 配置作用域（仅指定域名下有效，默认为当前域名）
        userIdCookie.setDomain("localhost");
        // 5. 配置 HttpOnly（true 表示客户端 JS 无法读取，防止 XSS 攻击）
        userIdCookie.setHttpOnly(true);
        // 6. 配置 Secure（true 表示仅 HTTPS 协议下传输，HTTP 不传输）
        // userIdCookie.setSecure(true); // 生产环境推荐开启
        
        // 7. 向响应中添加 Cookie（通过 Set-Cookie 头发送给客户端）
        response.addCookie(userIdCookie);
        response.addCookie(usernameCookie);
        
        return "Cookie 设置成功";
    }
    
    // 读取客户端携带的 Cookie
    @GetMapping("/get-cookie")
    public String getCookie(HttpServletRequest request) {
        StringBuilder cookieInfo = new StringBuilder();
        // 获取所有 Cookie（无 Cookie 时返回 null，需判空）
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                String name = cookie.getName();
                String value = cookie.getValue();
                cookieInfo.append(name).append("=").append(value).append("; ");
            }
        }
        return "客户端 Cookie：" + cookieInfo;
    }
    
    // 删除 Cookie
    @GetMapping("/delete-cookie")
    public String deleteCookie(HttpServletResponse response) {
        // 删除 Cookie 的核心：创建同名、同路径的 Cookie，设置 maxAge=0
        Cookie userIdCookie = new Cookie("userId", "");
        userIdCookie.setPath("/api"); // 必须与设置时的路径一致，否则删除失败
        userIdCookie.setMaxAge(0); // 立即删除
        response.addCookie(userIdCookie);
        
        return "Cookie 删除成功";
    }
}
```

#### 2.1.5 Cookie 关键配置说明

- **`setPath(String path)`**：指定 Cookie 的作用路径（如 `/api`），仅客户端向该路径发送请求时才携带该 Cookie（默认路径为当前请求路径的父路径）；
- **`setDomain(String domain)`**：指定 Cookie 的作用域（如 `xxx.com`），子域名（如 `a.xxx.com`）可共享该 Cookie（默认仅当前域名有效）；
- **`setHttpOnly(boolean httpOnly)`**：开启后客户端 JavaScript 无法通过 `document.cookie` 读取该 Cookie，有效防止 XSS 攻击（生产环境推荐开启）；
- **`setSecure(boolean secure)`**：开启后仅在 HTTPS 协议下传输 Cookie，HTTP 协议不传输（防止 Cookie 被窃听，生产环境推荐开启）；
- **`setComment(String comment)`**：Cookie 描述信息（浏览器通常不显示，仅调试用）。

### 2.2 Session 核心知识

#### 2.2.1 定义与本质

Session 是**服务器端的会话存储机制**，用于存储用户会话数据（如登录状态、用户信息）。核心逻辑：

- 服务器为每个新用户创建唯一的 Session 对象（包含 Session ID）；
- 服务器通过 Cookie 将 Session ID 发送给客户端（默认 Cookie 名为 `JSESSIONID`）；
- 客户端后续请求携带该 Cookie（Session ID），服务器通过 Session ID 找到对应的 Session 对象，从而识别用户身份。

核心关联：Session 依赖 Cookie 传递 Session ID（无 Cookie 时需通过 URL 重写传递，但不推荐）。

#### 2.2.2 Session 的核心特性

1. **存储位置**：服务器端（内存、硬盘、数据库等，默认内存）；
2. **数据格式**：键值对（`String -> Object`），可存储复杂对象（如 User 实体）；
3. **大小限制**：无明确限制（由服务器内存决定，避免存储大量数据）；
4. **安全性**：数据存储在服务器端，客户端仅持有 Session ID（敏感数据更安全）；
5. **生命周期**：默认由服务器管理（超时自动销毁，默认超时时间通常为 30 分钟）；
6. **跨域限制**：Session 依赖 Cookie 传递 Session ID，受同源策略限制（跨域需特殊处理）。

#### 2.2.3 Session 生命周期与配置

1. **创建时机**：用户第一次访问服务器（如未携带有效 Session ID）时，服务器自动创建 Session；
2. **销毁时机**：
    1. 会话超时（服务器未收到该 Session 的请求，达到超时时间）；
    2. 服务器主动调用 `session.invalidate()` 方法（如退出登录）；
    3. 服务器重启或 Session 存储介质（如内存）被清理。
3. **Session 超时配置（Spring Boot 项目）**：
    1. 配置文件方式（`application.yml`）：
        - ```YAML
            server:
              servlet:
                session:
                  timeout: 1800s # 超时时间 30 分钟（默认值，单位可省略，默认秒）
                  cookie:
                    http-only: true #  Session ID 对应的 Cookie 开启 HttpOnly
                    secure: true # 仅 HTTPS 传输（生产环境推荐）
            ```
    2. 代码方式（手动设置超时时间）：
        - ```Java
            @GetMapping("/set-session-timeout")
            public String setSessionTimeout(HttpSession session) {
                session.setMaxInactiveInterval(3600); // 超时时间 1 小时（单位：秒）
                return "Session 超时时间设置为 1 小时";
            }
            ```

#### 2.2.4 Session 核心操作（Java 代码示例）

在 Spring MVC 中，通过 `HttpSession` 对象操作 Session 数据：

```Java
@RestController
public class SessionController {
    // 登录：创建 Session 并存储用户信息
    @PostMapping("/login")
    public String login(@RequestBody UserLoginDTO loginDTO, HttpSession session) {
        // 模拟数据库校验用户名密码
        if ("test".equals(loginDTO.getUsername()) && "123456".equals(loginDTO.getPassword())) {
            // 存储用户信息到 Session（键值对，可存储对象）
            session.setAttribute("loginUser", new User(1001, "test", "test@xxx.com"));
            // 获取 Session ID（服务器自动生成，通过 Cookie 发送给客户端）
            String sessionId = session.getId();
            return "登录成功，Session ID：" + sessionId;
        }
        return "用户名或密码错误";
    }
    
    // 验证登录状态：从 Session 获取用户信息
    @GetMapping("/check-login")
    public String checkLogin(HttpSession session) {
        User loginUser = (User) session.getAttribute("loginUser");
        if (loginUser != null) {
            return "已登录，用户信息：" + loginUser.getUsername() + "(" + loginUser.getEmail() + ")";
        }
        return "未登录";
    }
    
    // 退出登录：销毁 Session
    @GetMapping("/logout")
    public String logout(HttpSession session) {
        session.invalidate(); // 强制销毁当前 Session（清除所有数据）
        return "退出登录成功";
    }
}
```

### 2.3 Cookie 与 Session 对比与选型

| 对比维度     | Cookie                                     | Session                                          |
| ------------ | ------------------------------------------ | ------------------------------------------------ |
| 存储位置     | 客户端（浏览器）                           | 服务器端                                         |
| 数据安全性   | 较低（明文存储，可被客户端修改）           | 较高（数据在服务器，客户端仅持 Session ID）      |
| 数据类型限制 | 仅支持字符串（需手动序列化复杂数据）       | 支持任意 Object 类型（直接存储对象）             |
| 大小限制     | 单个 4KB，数量限制（50 个/域名）           | 无明确限制（受服务器内存影响）                   |
| 生命周期     | 可通过 `setMaxAge` 精确控制（持久/会话）   | 默认超时销毁（30 分钟），可手动配置              |
| 服务器压力   | 无（数据在客户端）                         | 有（数据在服务器，高并发需考虑存储方案）         |
| 跨域支持     | 受同源策略限制（可通过 CORS 配置部分支持） | 依赖 Cookie，跨域需特殊处理（如共享 Session ID） |
| 典型使用场景 | 存储非敏感数据（如记住登录状态、主题配置） | 存储敏感数据（如用户信息、权限信息）             |

#### 选型建议

1. 存储**非敏感、少量数据**（如是否记住登录、语言偏好）→ 用 Cookie；
2. 存储**敏感、复杂数据**（如用户 ID、权限角色）→ 用 Session；
3. 高并发场景：Session 需避免存储在内存（可改用 Redis 存储 Session 数据，提高扩展性）；
4. 跨域场景：优先使用 JWT（替代 Cookie+Session），或通过 CORS 配置允许跨域携带 Cookie。

### 2.4 常见问题与解决方案

#### 2.4.1 问题 1：Session 失效（登录状态丢失）

- 原因：
    - Session 超时（默认 30 分钟无操作）；
    - 服务器重启（内存中的 Session 被清理）；
    - 客户端禁用 Cookie（无法传递 Session ID）；
    - 跨域请求未携带 Cookie（同源策略限制）。
- 解决方案：
    - 延长 Session 超时时间（根据业务需求配置）；
    - 用 Redis 存储 Session（持久化，服务器重启不丢失）；
    - 跨域场景：配置 `Access-Control-Allow-Credentials: true` 允许跨域携带 Cookie；
    - 客户端禁用 Cookie 时：通过 URL 重写传递 Session ID（不推荐，安全性低）。

#### 2.4.2 问题 2：Cookie 无法被客户端存储

- 原因：
    - Cookie 大小超过 4KB 或数量超过限制；
    - 客户端浏览器禁用 Cookie；
    - `setSecure(true)` 开启后，使用 HTTP 协议访问（Cookie 不传输）；
    - `setDomain` 配置错误（如跨域设置）。
- 解决方案：
    - 拆分大型 Cookie（减少单个 Cookie 大小）；
    - 提示用户启用 Cookie（核心功能依赖时）；
    - 生产环境使用 HTTPS 协议，或关闭 `setSecure`（仅测试环境）；
    - 修正 `setDomain` 配置（确保与当前访问域名一致）。

#### 2.4.3 问题 3：Cookie 数据泄露（XSS 攻击）

- 原因：客户端 JavaScript 可读取 Cookie 数据，XSS 攻击脚本可窃取敏感信息；
- 解决方案：
    - 开启 `setHttpOnly(true)`（禁止 JS 读取 Cookie）；
    - 敏感数据（如用户 ID）加密后存储在 Cookie 中；
    - 开启 `setSecure(true)`（仅 HTTPS 传输）。

## 三、分层解耦：三层架构设计

为解决代码耦合、提高可维护性，后端开发普遍采用「三层架构」，核心思想是「职责分离、逐层依赖」，每层仅关注自身核心功能，不干预其他层逻辑。

### 3.1 三层架构核心职责

| 层级                     | 核心职责                                                     | 技术注解                                    | 依赖关系                             |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------- | ------------------------------------ |
| 控制层（Controller）     | 接收前端请求、解析参数、校验请求合法性、返回响应结果（不处理业务逻辑） | Spring MVC：`@Controller`/`@RestController` | 依赖 Service 层（不直接依赖 Dao 层） |
| 业务逻辑层（Service）    | 处理核心业务（如数据校验、逻辑计算、事务控制、多 Dao 协同）  | Spring：`@Service`                          | 依赖 Dao 层                          |
| 数据访问层（Dao/Mapper） | 仅负责与数据库交互，执行 CRUD 操作（不包含任何业务逻辑）     | Spring：`@Repository`；MyBatis：`@Mapper`   | 不依赖其他层                         |

#### 3.1.1 分层设计原则

1. 单向依赖：Controller → Service → Dao（禁止反向依赖，如 Dao 调用 Service）；
2. 职责单一：每层只做自己的事（如 Controller 不写业务逻辑，Service 不直接操作 `HttpServletRequest`）；
3. 事务集中：数据库事务（`@Transactional`）必须加在 Service 层（Dao 层方法可能被多个 Service 调用，事务边界不清晰）。

#### 3.1.2 错误示范与正确示例

- 错误示范（Controller 直接调用 Dao）：
    - ```Java
        @RestController
        public class UserController {
            // 错误：跳过 Service 层，直接依赖 Dao
            @Autowired
            private UserDao userDao;
            
            @PostMapping("/user/add")
            public Result addUser(User user) {
                // 错误：业务逻辑（参数校验）写在 Controller
                if (user.getUsername() == null) {
                    return Result.error("用户名不能为空");
                }
                userDao.insert(user);
                return Result.success();
            }
        }
        ```
- 正确示范（分层协作）：
    - ```Java
        // Controller 层：仅接收请求、调用 Service
        @RestController
        public class UserController {
            @Autowired
            private UserService userService;
            
            @PostMapping("/user/add")
            public Result addUser(@RequestBody User user) {
                return userService.addUser(user);
            }
        }
        
        // Service 层：处理业务逻辑、调用 Dao
        @Service
        @Transactional // 事务控制在 Service 层
        public class UserService {
            @Autowired
            private UserDao userDao;
            
            public Result addUser(User user) {
                // 业务逻辑：参数校验
                if (StringUtils.isEmpty(user.getUsername())) {
                    return Result.error("用户名不能为空");
                }
                // 业务逻辑：判断用户名是否已存在
                if (userDao.existsByUsername(user.getUsername())) {
                    return Result.error("用户名已存在");
                }
                userDao.insert(user);
                return Result.success();
            }
        }
        
        // Dao 层：仅做 CRUD
        @Repository
        public interface UserDao {
            void insert(User user);
            boolean existsByUsername(String username);
        }
        ```

## 四、Spring 核心：IOC 与 DI 机制

Spring 框架的核心是「控制反转（IOC）」和「依赖注入（DI）」，其目的是解除代码间的硬耦合，让对象的创建和依赖管理由 Spring 容器统一负责。

### 4.1 IOC（Inversion of Control）：控制反转

#### 4.1.1 核心思想

- 传统开发：开发者手动 `new` 对象（如 `UserService userService = new UserServiceImpl()`），对象的创建、初始化、销毁全由开发者控制；
- IOC 开发：开发者不再手动创建对象，而是将对象的创建权利「反转」给 Spring 容器，Spring 会自动创建对象并管理其生命周期（创建、初始化、销毁）。

#### 4.1.2 组件注册：让 Spring 识别对象

要让 Spring 容器管理对象，需通过「组件注解」标记类，再通过「组件扫描」让 Spring 发现这些类。

##### （1）常用组件注解（语义化分层注解）

| 注解              | 适用层级              | 说明                                                         |
| ----------------- | --------------------- | ------------------------------------------------------------ |
| `@Component`      | 通用组件              | 无明确分层的组件（如工具类），Spring 通用注解                |
| `@Controller`     | 控制层                | Spring MVC 专用，标识控制器类，支持请求映射（`@RequestMapping`） |
| `@RestController` | 控制层（接口）        | 等价于 `@Controller + @ResponseBody`，直接返回 JSON 响应（无需视图解析） |
| `@Service`        | 业务逻辑层            | 标识 Service 类，Spring 会自动为其添加事务支持（需配合 `@Transactional`） |
| `@Repository`     | 数据访问层            | 标识 Dao 类，Spring 会自动捕获数据库操作异常并转换为 Spring 统一异常 |
| `@Mapper`         | 数据访问层（MyBatis） | MyBatis 专用注解，标识 Mapper 接口，需配合 `@MapperScan` 扫描 |

##### （2）组件扫描（`@ComponentScan`）

Spring 需通过组件扫描才能发现带有上述注解的类，将其创建为对象（Bean）存入容器。

- 扫描规则：扫描指定包及其子包下的所有类；
- 默认行为：Spring Boot 启动类若在根包（如 `com.imooc.springboot.Application`）下，无需手动添加 `@ComponentScan`，Spring 会自动扫描「启动类所在包及其子包」；
- 手动配置：若组件不在默认扫描路径下，需显式配置：
    - ```Java
        // Spring Boot 启动类
        @SpringBootApplication
        // 扫描多个包：逗号分隔
        @ComponentScan({"com.imooc.service", "com.imooc.dao"})
        // MyBatis Mapper 扫描（单独配置）
        @MapperScan("com.imooc.mapper")
        public class Application {
            public static void main(String[] args) {
                SpringApplication.run(Application.class, args);
            }
        }
        ```

### 4.2 DI（Dependency Injection）：依赖注入

#### 4.2.1 核心思想

依赖注入是 IOC 的具体实现：Spring 容器在创建对象时，自动将该对象依赖的其他对象（如 Controller 依赖的 Service、Service 依赖的 Dao）注入到对象中，无需开发者手动 `set` 或构造。

#### 4.2.2 三种注入方式（对比与选型）

Spring 支持三种依赖注入方式，各有适用场景，推荐优先使用构造函数注入。

| 注入方式     | 写法示例                                                     | 优点                                                         | 缺点                                                         | 适用场景                           |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------- |
| 属性注入     | 注解直接加在字段上                                           | 写法最简单，代码简洁                                         | 1. 无法注入 `final` 字段；<br>2. 易引发循环依赖；<br>3. 测试时难以 Mock | 小型项目、非核心组件               |
| 构造函数注入 | 注解加在构造函数上（Spring 4.3+ 单构造函数可省略 `@Autowired`） | 1. 支持 `final` 字段；<br>2. 避免循环依赖；<br>3. 测试易 Mock；<br>4. 强制依赖初始化 | 字段较多时，构造函数代码较长                                 | 核心组件（如 Service、Controller） |
| Setter 注入  | 注解加在 `setXXX()` 方法上                                   | 灵活性高，可动态修改依赖                                     | 1. 不支持 `final` 字段；<br>2. 易被外部修改依赖，破坏封装    | 依赖可选、需动态调整的场景         |

| **注入方式**                              | **写法示例**                                                 | **优点**                                            | **缺点**                                                     |
| ----------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ |
| 属性注入（@Autowired 加在字段上）         | `@Autowired private UserService userService;`                | 写法最简单                                          | 1. 不能注入 final 字段；2. 容易导致循环依赖；3. 测试时难以 mock |
| 构造函数注入（@Autowired 加在构造函数上） | `private UserService userService; @Autowired public UserController(UserService userService) { this.userService; }` | 1. 支持 final 字段；2. 避免循环依赖；3. 测试易 mock | 写法稍繁琐（字段多时光构造函数长）                           |
| Setter 注入（@Autowired 加在 set 方法上） | `private UserService userService; @Autowired public void setUserService(UserService userService) { this.userService = userService; }` | 灵活性高（可动态修改依赖）                          | 不推荐：易被外部修改依赖，破坏封装                           |

##### 代码示例

1. 属性注入：
    1. ```Java
        @RestController
        public class UserController {
            // 直接在字段上添加 @Autowired
            @Autowired
            private UserService userService;
        }
        ```
2. 构造函数注入（推荐）：
    1. ```Java
        @RestController
        public class UserController {
            // 可加 final（强制依赖，必须注入）
            private final UserService userService;
            
            // 单构造函数，省略 @Autowired
            public UserController(UserService userService) {
                this.userService = userService;
            }
        }
        ```
3. Setter 注入：
    1. ```Java
        @RestController
        public class UserController {
            private UserService userService;
            
            // 注解加在 set 方法上
            @Autowired
            public void setUserService(UserService userService) {
                this.userService = userService;
            }
        }
        ```

#### 4.2.3 注入冲突解决方案（多实现类场景）

`@Autowired` 默认按「类型（Type）」注入，若某个接口有多个实现类，Spring 会因无法确定注入哪个而报错（`NoUniqueBeanDefinitionException`），解决方案如下：

##### 方案 1：`@Qualifier` + 实例名称（精准匹配）

通过 `@Service`/`@Component` 指定 Bean 名称，再用 `@Qualifier` 明确注入哪个 Bean。

```Java
// 接口
public interface UserService {
    Result addUser(User user);
}

// 实现类 1：指定 Bean 名称为 userServiceA
@Service("userServiceA")
public class UserServiceImplA implements UserService {
    // 实现逻辑
}

// 实现类 2：指定 Bean 名称为 userServiceB
@Service("userServiceB")
public class UserServiceImplB implements UserService {
    // 实现逻辑
}

// Controller 中指定注入 userServiceA
@RestController
public class UserController {
    @Autowired
    @Qualifier("userServiceA") // 匹配 @Service 定义的名称
    private UserService userService;
}
```

##### 方案 2：`@Primary` 指定默认 Bean

在多个实现类中，用 `@Primary` 标记「默认优先注入」的 Bean，无需额外指定名称。

```Java
@Service
@Primary // 优先注入该实现类
public class UserServiceImplA implements UserService {
    // 实现逻辑
}

@Service
public class UserServiceImplB implements UserService {
    // 实现逻辑
}

// 直接 @Autowired 会注入 UserServiceImplA
@RestController
public class UserController {
    @Autowired
    private UserService userService;
}
```

##### 方案 3：`@Resource` 按名称注入

`@Resource` 是 JDK 原生注解（非 Spring 注解），默认按「名称（Name）」注入，可替代 `@Autowired + @Qualifier`，兼容性更强。

```Java
@RestController
public class UserController {
    // name 匹配 @Service 定义的 Bean 名称
    @Resource(name = "userServiceB")
    private UserService userService;
}
```

### 4.3 核心概念总结

| 术语     | 通俗理解                                                     |
| -------- | ------------------------------------------------------------ |
| IOC 容器 | Spring 管理对象的「容器」，存储所有通过注解/配置创建的 Bean  |
| Bean     | 被 Spring 容器管理的对象（如 Service、Dao、Controller 实例） |
| IOC      | 对象创建权利由开发者转移到 Spring 容器，解耦硬编码           |
| DI       | Spring 容器自动将依赖的 Bean 注入到目标对象中，无需手动管理依赖 |

## 五、实战流程串联（前后端交互完整链路）

结合上述知识点，以下是一个完整的前后端交互开发流程（包含 Cookie+Session 状态保持）：

### 5.1 开发步骤

1. **定义接口协议**：前后端约定 HTTP 请求方式、路径、参数、响应格式（例：`POST /api/login` 登录接口，`GET /api/user/info` 获取用户信息接口）；
2. **创建实体类**（Entity/DTO）：定义 `UserLoginDTO`（登录请求参数）、`User`（用户实体）、`Result`（统一响应格式）；
3. **开发 Dao 层**：用 `@Mapper` 定义 `UserMapper` 接口，编写查询用户的 SQL 方法；
4. **开发 Service 层**：用 `@Service` 定义 `UserService` 类，注入 `UserMapper`，实现登录校验逻辑；
5. **开发 Controller 层**：用 `@RestController` 定义 `UserController`，注入 `UserService`，实现登录（创建 Session）、获取用户信息（读取 Session）、退出登录（销毁 Session）接口；
6. **配置 Cookie 与 Session**：在 `application.yml` 中配置 Session 超时时间、Cookie 安全属性；
7. **前后端联调**：前端调用登录接口后，后续请求自动携带 Cookie（Session ID），后端通过 Session 识别登录状态。

### 5.2 核心代码结构

```Plain
com.imooc.springboot
├── Application.java        // 启动类（@SpringBootApplication + @MapperScan）
├── controller
│   └── UserController.java  // 控制层（登录、用户信息接口）
├── service
│   └── UserService.java     // 业务层（登录校验逻辑）
├── mapper
│   └── UserMapper.java      // 数据访问层（查询用户）
├── entity
│   ├── User.java            // 用户实体类
│   └── UserLoginDTO.java    // 登录请求DTO
└── common
    └── Result.java          // 统一响应类
```

## 六、常见问题与避坑指南

1. **`@ComponentScan`** **扫描不到 Bean**：
    1. 检查注解是否正确（如 Controller 用 `@Controller` 而非 `@Component`）；
    2. 确认组件所在包是启动类的子包，或手动配置 `@ComponentScan` 扫描路径。
2. **依赖注入报错（NoSuchBeanDefinitionException）**：
    1. 检查被注入的类是否加了组件注解（如 `@Service`）；
    2. 检查组件是否被 `@ComponentScan` 扫描到。
3. **循环依赖问题**（A 依赖 B，B 依赖 A）：
    1. 优先用构造函数注入（Spring 会检测循环依赖并报错，提前发现问题）；
    2. 重构代码，提取公共依赖为第三方组件，解除循环。
4. **事务不生效**：
    1. 事务注解 `@Transactional` 必须加在 Service 层（Controller 层事务不生效）；
    2. 确保事务方法是「public」（private 方法事务不生效）；
    3. 避免在同一个 Service 类中调用事务方法（AOP 无法拦截）。
5. **HTTP 请求体解析失败**：
    1. 确保请求头 `Content-Type` 与请求体格式一致（如 JSON 对应 `application/json`）；
    2. Controller 方法参数需加 `@RequestBody` 注解（解析 JSON 请求体）。
6. **Session 登录状态丢失**：
    1. 检查 Session 超时时间是否过短（调整 `server.servlet.session.timeout`）；
    2. 跨域场景需配置 `Access-Control-Allow-Credentials: true` 和 `Access-Control-Allow-Origin`（指定具体域名，不能为 `*`）；
    3. 确认客户端未禁用 Cookie（或开启了隐私模式，会话级 Cookie 失效）。
7. **Cookie 无法跨域携带**：
    1. 后端配置 CORS 时，需同时设置 `allowed-origins`（具体域名）、`allow-credentials: true`；
    2. 前端请求时需设置 `withCredentials: true`（Axios 等框架配置）。

## 七、总结

本文围绕 Web 核心知识、Spring 分层开发、Cookie 与 Session 状态保持展开，核心要点包括：

1. HTTP 协议：请求/响应格式、核心头字段（`Set-Cookie`/`Cookie`）；
2. Cookie 与 Session：状态保持的核心机制，Cookie 存储在客户端（传递 Session ID），Session 存储在服务器端（存储敏感数据）；
3. 三层架构：Controller（接收请求）→ Service（业务逻辑）→ Dao（数据访问）的职责与依赖关系；
4. Spring IOC/DI：组件注册、组件扫描、三种注入方式及冲突解决；
5. 实战流程：包含登录状态保持的前后端交互完整链路与避坑指南。

这些知识点是 Java 后端开发的基石，掌握后可应对大部分常规项目开发场景。实际开发中需注意 Cookie/Session 的安全配置、分层职责边界、依赖注入规范，同时结合框架特性（如 Spring Boot 自动配置、Redis 存储 Session）提高系统的安全性和扩展性。