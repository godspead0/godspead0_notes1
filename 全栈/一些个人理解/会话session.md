# Spring Boot 中 Session 与 Token 的对比及实战

在**Web开发**中，**会话（Session）** 指的是 **客户端（如浏览器）与服务器之间一次连续的、有状态的交互过程**，核心作用是**解决HTTP协议的无状态问题**，保存用户的临时身份和操作信息。

结合你学习的Spring Boot + Vue全栈开发，我会用通俗的语言拆解会话的本质、作用和实现方式。

## 一、先理解：HTTP的「无状态」痛点

HTTP协议是**无状态协议**——服务器不会记住任何两次请求之间的关联。

举个例子：

1. 你在浏览器输入账号密码，登录电商网站（第一次请求）；
2. 登录成功后，点击「我的订单」查看订单（第二次请求）。

如果没有会话机制，服务器会把这两次请求当成**两个完全无关的请求**，它不知道“查看订单的用户就是刚才登录的用户”，你就需要每次操作都重新登录。

而**会话的核心价值**，就是让服务器能“记住”同一个用户的连续请求。

## 二、会话的核心定义与类比

你可以把会话类比成：**你和客服的一次通话过程**

- 通话接通 → 会话创建（用户第一次访问服务器，服务器生成唯一会话标识）；
- 通话中交流 → 会话活跃（用户多次请求服务器，服务器通过会话标识识别用户，保存用户状态，比如登录信息、购物车数据）；
- 通话挂断 → 会话销毁（用户关闭浏览器/长时间不操作，服务器删除会话数据）。

在Web开发中，**一次会话的范围**：从用户打开浏览器访问服务器开始，到用户关闭浏览器、会话超时，或主动退出登录为止。

## 三、会话的实现方式（Web开发核心）

Web会话的实现，核心依赖 **`Session`****（服务器端存储） +** **`Cookie`****（客户端存储）** 的配合，步骤如下：

1. **会话创建**
    1. 用户第一次访问服务器（比如访问登录页），服务器会自动生成一个**唯一的会话ID**（如 `JSESSIONID=6789abc`）；
    2. 服务器在内存/数据库中创建一个**Session对象**，用于存储该用户的状态（比如后续登录后的用户ID、用户名）；
    3. 服务器通过**响应头**，把会话ID以Cookie的形式发送给浏览器，浏览器会自动保存这个Cookie（默认是会话级Cookie，关闭浏览器就失效）。
2. **会话活跃**
    1. 用户后续的每次请求（比如查看订单、添加购物车），浏览器都会**自动携带这个Cookie**（包含会话ID）发送给服务器；
    2. 服务器拿到会话ID后，找到对应的Session对象，从而识别用户身份，获取用户的状态数据，无需重复登录。
3. **会话销毁**
    1. **超时销毁**：用户长时间不操作（默认30分钟，可配置），服务器自动删除Session对象；
    2. **主动销毁**：用户点击“退出登录”，服务器调用`session.invalidate()`方法删除Session；
    3. **浏览器关闭**：会话级Cookie失效，下次访问服务器会创建新的会话。

### 代码示例：Spring Boot中操作Session

```Java
@RestController
@RequestMapping("/user")
public class UserController {

    // 登录接口：成功后将用户信息存入Session
    @PostMapping("/login")
    public String login(String username, String password, HttpSession session) {
        // 假设验证账号密码通过
        if ("admin".equals(username) && "123456".equals(password)) {
            // 存入用户信息到Session
            session.setAttribute("loginUser", username);
            // 设置会话超时时间：3600秒（1小时）
            session.setMaxInactiveInterval(3600);
            return "登录成功";
        }
        return "账号密码错误";
    }

    // 查看个人信息：从Session获取用户信息
    @GetMapping("/info")
    public String getUserInfo(HttpSession session) {
        String loginUser = (String) session.getAttribute("loginUser");
        if (loginUser != null) {
            return "当前登录用户：" + loginUser;
        }
        return "请先登录";
    }

    // 退出登录：销毁Session
    @GetMapping("/logout")
    public String logout(HttpSession session) {
        // 销毁当前会话
        session.invalidate();
        return "退出成功";
    }
}
```

## 四、会话的两种常见形态（传统开发 vs 前后端分离）

### 1. 传统后端渲染（如JSP）：服务器端Session

- Session数据**存储在服务器内存**（或分布式缓存如Redis）；
- 会话ID通过**浏览器Cookie**传递；
- 缺点：分布式部署时，需要解决Session共享问题（比如用Redis统一存储Session）。

### 2. 前后端分离（Vue + Spring Boot）：Token替代Session

在前后端分离场景中，更常用 **Token** 替代传统Session，核心原因是：前端可能是多端（浏览器、APP、小程序），Cookie机制在非浏览器端（如APP）不适用。

- 流程：用户登录成功 → 服务器生成Token（一串加密字符串）→ 前端存储Token（LocalStorage/SessionStorage）→ 后续请求在请求头中携带Token → 服务器验证Token识别用户。
- 对比：Token是**客户端存储**，Session是**服务器端存储**；Token更适合跨端、分布式场景。

## 五、会话的核心特点

1. **唯一性**：每个用户的会话ID都是唯一的，服务器通过它区分不同用户；
2. **临时性**：会话数据是临时存储的，会话销毁后数据就会丢失；
3. **安全性**：会话ID是随机生成的长字符串，且Session数据存储在服务器端，比客户端存储更安全（比如用户密码不会存在客户端）。

### 总结

1. Web中的会话，是**客户端与服务器的一次连续交互过程**，核心是解决HTTP无状态的问题，保存用户临时状态；
2. 传统会话基于 **Session（服务器） + Cookie（客户端）** 实现，前后端分离场景常用**Token**替代；
3. 在Spring Boot开发中，你可以直接通过`HttpSession`对象操作会话数据，实现用户登录状态管理等功能。