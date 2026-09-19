# Web 开发中 Token 的奥秘：Vue + Spring Boot 场景解析

你想知道Web开发中Token的具体含义，尤其是它在Vue+Spring Boot前后端分离场景中替代传统Session的核心作用，我会用通俗的语言拆解Token的定义、实现流程和实际用法，结合代码示例让你快速理解。

## 一、Token的核心定义

Token（中文：令牌）是**服务器生成的一串加密字符串**，核心作用是：**作为客户端访问服务器的“临时通行证”，让服务器识别用户身份**，替代传统Session解决前后端分离、多端（浏览器/APP/小程序）场景下的身份认证问题。

你可以把Token类比成：

- 传统Session：你去游乐园，工作人员给你发一张**实体门票（会话ID Cookie）**，每次玩项目都要出示门票，门票信息存在游乐园（服务器）的系统里；
- Token：你去游乐园，工作人员给你发一张**电子二维码（Token）**，二维码里直接包含你的身份信息（加密），每次玩项目只需扫码，游乐园不用存你的门票信息，只需验证二维码是否有效。

## 二、为什么需要Token（解决Session的痛点）

传统Session依赖Cookie，在以下场景会失效：

1. **前后端分离+多端适配**：APP、小程序不支持Cookie机制，无法传递会话ID；
2. **分布式部署**：多台服务器时，Session需要共享（如Redis），配置复杂；
3. **跨域请求**：浏览器跨域时，Cookie默认不携带，需要额外配置。

而Token是**无状态、跨端、跨域友好**的，完美解决这些问题。

## 三、Token的核心类型：JWT（最常用）

实际开发中，Token几乎都是基于**JWT（JSON Web Token）** 实现的，JWT是Token的一种标准化格式，结构如下：

```Plain
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsInVzZXJuYW1lIjoiYWRtaW4iLCJleHAiOjE3MTYxMjM0NTB9.8h9Z8w0Z7e5k4b3a2s1d0f9g8h7j6k5l4m3n2b1v0c9x8s7a6d5f4g3h2j1k0l9m8n7b6v5c4x3s2a1d0f
```

JWT由3部分组成（用`.`分隔）：

1. **Header（头部）**：声明加密算法（如HS256）和Token类型；
2. **Payload（载荷）**：存储用户核心信息（如用户ID、用户名），**不要存密码等敏感信息**；
3. **Signature（签名）**：用服务器密钥对Header+Payload加密，防止Token被篡改。

## 四、Token（JWT）的实现流程（Vue + Spring Boot）

暂时无法在豆包文档外展示此内容

### 代码示例：Spring Boot实现JWT Token

#### 1. 先引入依赖（pom.xml）

```XML
<!-- JWT依赖 -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

#### 2. JWT工具类（生成/验证Token）

```Java
package com.example.springbootdemo.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import javax.crypto.SecretKey;
import java.util.Date;

public class JwtUtil {
    // 服务器密钥（务必保密，可配置在配置文件）
    private static final String SECRET_KEY = "my-secret-key-12345678901234567890";
    // Token过期时间：1小时（3600秒）
    private static final long EXPIRATION_TIME = 3600 * 1000;

    // 生成Token
    public static String generateToken(Integer userId, String username) {
        // 生成加密密钥
        SecretKey key = Keys.hmacShaKeyFor(SECRET_KEY.getBytes());
        // 构建JWT Token
        return Jwts.builder()
                .claim("userId", userId) // 载荷：用户ID
                .claim("username", username) // 载荷：用户名
                .setIssuedAt(new Date()) // 签发时间
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME)) // 过期时间
                .signWith(key) // 签名
                .compact();
    }

    // 验证Token并解析载荷
    public static Claims parseToken(String token) {
        SecretKey key = Keys.hmacShaKeyFor(SECRET_KEY.getBytes());
        // 验证Token有效性（过期/篡改都会抛异常）
        return Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();
    }
}
```

#### 3. 登录接口（生成Token）

```Java
@RestController
@RequestMapping("/user")
public class UserController {

    // 登录接口：生成Token返回给前端
    @PostMapping("/login")
    public Map<String, Object> login(String username, String password) {
        Map<String, Object> result = new HashMap<>();
        // 模拟验证账号密码
        if ("admin".equals(username) && "123456".equals(password)) {
            // 生成Token
            String token = JwtUtil.generateToken(1, username);
            result.put("code", 200);
            result.put("msg", "登录成功");
            result.put("token", token); // 返回Token
            return result;
        }
        result.put("code", 400);
        result.put("msg", "账号密码错误");
        return result;
    }

    // 需认证的接口：验证Token并获取用户信息
    @GetMapping("/info")
    public Map<String, Object> getUserInfo(@RequestHeader("Authorization") String token) {
        Map<String, Object> result = new HashMap<>();
        try {
            // 解析Token（去掉前缀Bearer，若前端传的是Bearer Token）
            if (token.startsWith("Bearer ")) {
                token = token.substring(7);
            }
            Claims claims = JwtUtil.parseToken(token);
            // 获取载荷中的用户信息
            Integer userId = claims.get("userId", Integer.class);
            String username = claims.get("username", String.class);
            
            result.put("code", 200);
            result.put("data", Map.of("userId", userId, "username", username));
        } catch (Exception e) {
            // Token过期/篡改，返回未授权
            result.put("code", 401);
            result.put("msg", "Token无效或已过期");
        }
        return result;
    }
}
```

#### 4. 前端Vue示例（存储/携带Token）

```JavaScript
// 登录请求：存储Token
async login() {
  const res = await axios.post('/user/login', {
    username: 'admin',
    password: '123456'
  });
  if (res.data.code === 200) {
    // 存储Token到LocalStorage（持久化，关闭浏览器不丢失）
    localStorage.setItem('token', res.data.token);
  }
}

// 请求拦截器：每次请求携带Token
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    // 在请求头中添加Token（规范写法：Bearer + 空格 + Token）
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 调用需认证的接口
async getUserInfo() {
  const res = await axios.get('/user/info');
  console.log(res.data);
}
```

## 五、Token vs Session 核心对比

| 维度       | Token（JWT）                  | Session                     |
| ---------- | ----------------------------- | --------------------------- |
| 存储位置   | 客户端（LocalStorage/Cookie） | 服务器端（内存/Redis）      |
| 状态性     | 无状态（服务器不存Token）     | 有状态（服务器存Session）   |
| 跨端适配   | 支持APP/小程序/浏览器         | 仅支持浏览器（依赖Cookie）  |
| 分布式部署 | 无需额外配置（无状态）        | 需Session共享（如Redis）    |
| 过期处理   | Token内自带过期时间           | 服务器配置超时时间          |
| 安全性     | 签名防篡改，不存敏感信息      | Session数据存服务器，更安全 |

### 总结

1. Token是服务器生成的加密字符串，核心是作为客户端的“临时通行证”，解决前后端分离/多端的身份认证问题；
2. 实际开发中常用**JWT**实现Token，由Header、Payload、Signature三部分组成，无状态且跨端友好；
3. Token和Session的核心区别是**存储位置和状态性**：Token存在客户端（无状态），Session存在服务器（有状态）；
4. 在Spring Boot开发中，可通过JWT工具类生成/验证Token，前端存储Token并在请求头中携带，实现身份认证。

如果需要，我可以帮你完善这个示例，添加**全局Token拦截器**（自动验证所有接口的Token）和**Token刷新机制**（避免Token过期后用户重新登录），让代码更贴近生产环境。