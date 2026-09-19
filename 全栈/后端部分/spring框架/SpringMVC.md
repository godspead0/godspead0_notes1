# Spring MVC 全面指南

## 1. Spring MVC 概述

### 1.1 什么是 Spring MVC

Spring MVC 是基于 Spring 框架的 Web 框架，用于替代传统的 Servlet 开发，提供了更简洁、更强大的 Web 开发能力。

**与传统 Servlet 对比：**

| 特性       | Servlet              | Spring MVC             |
| ---------- | -------------------- | ---------------------- |
| 配置复杂度 | 高，需要 web.xml     | 低，注解驱动           |
| 开发效率   | 低，手动处理请求参数 | 高，自动绑定           |
| 测试性     | 困难                 | 容易，支持单元测试     |
| 集成性     | 需要手动集成         | 与 Spring 生态无缝集成 |

## 2. Spring MVC 使用过程

### 2.1 基础配置

```java
// 控制器类
@Controller
@RequestMapping("/user")
public class UserController {
    
    @RequestMapping("/info")
    @ResponseBody
    public String getUserInfo() {
        return "用户信息";
    }
}
```

### 2.2 配置类设置

```java
// Spring 配置类
@Configuration
@ComponentScan(basePackages = "com.example.service,com.example.dao")
public class SpringConfig {
    // 业务层和数据访问层配置
}

// Spring MVC 配置类
@Configuration
@ComponentScan(basePackages = "com.example.controller")
@EnableWebMvc
public class SpringMvcConfig implements WebMvcConfigurer {
    // MVC 相关配置
}
```

### 2.3 启动类配置

```java
// 继承 AbstractAnnotationConfigDispatcherServletInitializer
public class ServletContainerInitConfig extends 
        AbstractAnnotationConfigDispatcherServletInitializer {
    
    // 加载 Spring 配置类
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }
    
    // 加载 Spring MVC 配置类
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }
    
    // 配置 DispatcherServlet 的映射路径
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

## 3. Spring MVC 加载过程

### 3.1 启动流程

1. **服务器启动** → 加载 web.xml
2. **创建 Spring 容器** → 初始化 SpringConfig
3. **创建 Spring MVC 容器** → 初始化 SpringMvcConfig
4. **DispatcherServlet 初始化** → 建立请求映射关系
5. **接收请求** → 根据 URL 匹配控制器方法
6. **执行方法** → 调用对应的 @RequestMapping 方法
7. **返回响应** → 渲染视图或返回数据

### 3.2 组件关系图

```
客户端请求 → DispatcherServlet → HandlerMapping 
    ↓
HandlerAdapter → 控制器方法 → 返回 ModelAndView
    ↓
ViewResolver → 视图渲染 → 响应客户端
```

## 4. Spring MVC Bean 加载控制

### 4.1 精确扫描方式

```java
@Configuration
@ComponentScan(basePackages = {
    "com.example.service",
    "com.example.dao",
    "com.example.utils"
})
public class SpringConfig {
    // 精确指定扫描包，排除 controller
}
```

### 4.2 排除过滤方式

```java
@Configuration
@ComponentScan(
    value = "com.example",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = "com\\.example\\.controller\\..*"
    )
)
public class SpringConfig {
    // 扫描整个包，但排除 controller 包
}

// Spring MVC 配置类单独扫描 controller
@Configuration
@ComponentScan(
    value = "com.example.controller",
    useDefaultFilters = false,
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = Controller.class
    )
)
public class SpringMvcConfig {
    // 只扫描带有 @Controller 注解的类
}
```

### 4.3 基于注解的排除

```java
// 使用自定义注解进行更精细的控制
@Configuration
@ComponentScan(
    value = "com.example",
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class),
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = RestController.class)
    }
)
public class SpringConfig {
    // 排除所有 Web 层相关的 Bean
}
```

## 5. 请求与响应处理

### 5.1 请求映射

```java
@Controller
@RequestMapping("/api/users") // 类级别路径
public class UserController {
    
    // 完整路径: /api/users/list
    @RequestMapping("/list")
    @ResponseBody
    public String listUsers() {
        return "用户列表";
    }
    
    // 路径冲突避免 - 使用不同的 HTTP 方法
    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseBody
    public String getUser(@PathVariable Long id) {
        return "用户ID: " + id;
    }
    
    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    @ResponseBody
    public String updateUser(@PathVariable Long id) {
        return "更新用户: " + id;
    }
}
```

### 5.2 参数接收

#### 5.2.1 基本类型参数

```java
@Controller
public class UserController {
    
    // GET /user/info?name=john&age=25
    @RequestMapping("/user/info")
    @ResponseBody
    public String getUserInfo(String name, Integer age) {
        return "姓名: " + name + ", 年龄: " + age;
    }
    
    // 使用 @RequestParam 指定参数名和默认值
    @RequestMapping("/user/search")
    @ResponseBody
    public String searchUsers(
            @RequestParam("keyword") String keyword,
            @RequestParam(value = "page", defaultValue = "1") Integer page,
            @RequestParam(value = "size", defaultValue = "10") Integer size) {
        return "搜索: " + keyword + ", 页码: " + page + ", 大小: " + size;
    }
}
```

#### 5.2.2 对象参数绑定

```java
// 用户实体类
public class User {
    private String name;
    private Integer age;
    private String email;
    
    // getter 和 setter 方法
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

@Controller
public class UserController {
    
    // 自动绑定对象属性
    // GET /user/create?name=john&age=25&email=john@example.com
    @RequestMapping("/user/create")
    @ResponseBody
    public String createUser(User user) {
        return "创建用户: " + user.getName() + ", 年龄: " + user.getAge();
    }
}
```

#### 5.2.3 数组和集合参数

```java
@Controller
public class UserController {
    
    // 数组参数: /user/delete?ids=1,2,3 或 /user/delete?ids=1&ids=2&ids=3
    @RequestMapping("/user/delete")
    @ResponseBody
    public String deleteUsers(Long[] ids) {
        return "删除用户ID: " + Arrays.toString(ids);
    }
    
    // 集合参数需要 @RequestParam
    @RequestMapping("/user/batchUpdate")
    @ResponseBody
    public String batchUpdateUsers(@RequestParam List<Long> userIds) {
        return "批量更新用户: " + userIds;
    }
    
    // Map 参数接收
    @RequestMapping("/user/filter")
    @ResponseBody
    public String filterUsers(@RequestParam Map<String, String> filters) {
        return "过滤条件: " + filters;
    }
}
```

### 5.3 JSON 数据处理

```java
@Controller
public class UserController {
    
    // 启用 JSON 支持
    @Configuration
    @EnableWebMvc
    public static class JsonConfig implements WebMvcConfigurer {
        @Override
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            converters.add(new MappingJackson2HttpMessageConverter());
        }
    }
    
    // 接收 JSON 请求体
    @RequestMapping(value = "/user/create", method = RequestMethod.POST)
    @ResponseBody
    public User createUser(@RequestBody User user) {
        // 处理用户创建逻辑
        user.setId(System.currentTimeMillis()); // 模拟生成ID
        return user; // 返回创建的用户对象（自动转为JSON）
    }
    
    // 接收 JSON 数组
    @RequestMapping(value = "/user/batchCreate", method = RequestMethod.POST)
    @ResponseBody
    public List<User> batchCreateUsers(@RequestBody List<User> users) {
        // 批量创建用户
        return users.stream()
                   .peek(user -> user.setId(System.currentTimeMillis()))
                   .collect(Collectors.toList());
    }
}
```

### 5.4 日期类型处理

```java
@Controller
public class UserController {
    
    // 日期参数格式化
    @RequestMapping("/user/byDate")
    @ResponseBody
    public String getUsersByDate(
            @DateTimeFormat(pattern = "yyyy-MM-dd") Date startDate,
            @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") Date endDate) {
        return "查询日期范围: " + startDate + " - " + endDate;
    }
    
    // 实体类中的日期字段
    public class UserProfile {
        private String name;
        
        @DateTimeFormat(pattern = "yyyy-MM-dd")
        private Date birthday;
        
        @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
        private Date createTime;
        
        // getter 和 setter
    }
    
    @RequestMapping(value = "/user/profile", method = RequestMethod.POST)
    @ResponseBody
    public UserProfile updateProfile(@RequestBody UserProfile profile) {
        return profile;
    }
}
```

### 5.5 响应处理

```java
@Controller
public class UserController {
    
    // 返回 JSON 响应
    @RequestMapping("/user/{id}")
    @ResponseBody
    public User getUser(@PathVariable Long id) {
        User user = new User();
        user.setId(id);
        user.setName("用户" + id);
        user.setAge(25);
        return user; // 自动转为 JSON
    }
    
    // 返回统一响应格式
    @RequestMapping("/api/user/{id}")
    @ResponseBody
    public Result<User> getApiUser(@PathVariable Long id) {
        try {
            User user = userService.findById(id);
            return Result.success(user);
        } catch (Exception e) {
            return Result.error("用户不存在");
        }
    }
    
    // 响应状态码控制
    @RequestMapping(value = "/user/{id}", method = RequestMethod.DELETE)
    @ResponseBody
    public ResponseEntity<String> deleteUser(@PathVariable Long id) {
        try {
            userService.deleteById(id);
            return ResponseEntity.ok("删除成功");
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND)
                               .body("用户不存在");
        }
    }
    
    // 文件下载
    @RequestMapping("/user/export")
    public ResponseEntity<byte[]> exportUsers() throws IOException {
        byte[] data = userService.exportUserData();
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        headers.setContentDispositionFormData("attachment", "users.xlsx");
        
        return new ResponseEntity<>(data, headers, HttpStatus.OK);
    }
}

// 统一响应结果类
class Result<T> {
    private boolean success;
    private String message;
    private T data;
    
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.success = true;
        result.message = "成功";
        result.data = data;
        return result;
    }
    
    public static <T> Result<T> error(String message) {
        Result<T> result = new Result<>();
        result.success = false;
        result.message = message;
        return result;
    }
    
    // getter 和 setter
}
```

## 6. RESTful 风格 API

### 6.1 REST 原则

- **统一接口**：一致的资源操作方式
- **无状态**：每次请求包含所有必要信息
- **可缓存**：响应可被缓存
- **分层系统**：客户端不需要了解底层实现
- **按需代码**：可下载并执行代码

### 6.2 RESTful 注解

```java
@RestController // @Controller + @ResponseBody
@RequestMapping("/api/v1/users")
public class UserRestController {
    
    // GET /api/v1/users - 获取用户列表
    @GetMapping
    public List<User> getUsers() {
        return userService.findAll();
    }
    
    // GET /api/v1/users/{id} - 获取指定用户
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // POST /api/v1/users - 创建用户
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
    
    // PUT /api/v1/users/{id} - 更新用户（完整更新）
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        user.setId(id);
        return userService.update(user);
    }
    
    // PATCH /api/v1/users/{id} - 部分更新用户
    @PatchMapping("/{id}")
    public User patchUser(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        return userService.patch(id, updates);
    }
    
    // DELETE /api/v1/users/{id} - 删除用户
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 6.3 路径变量高级用法

```java
@RestController
@RequestMapping("/api")
public class AdvancedRestController {
    
    // 多个路径变量
    @GetMapping("/users/{userId}/orders/{orderId}")
    public String getOrder(
            @PathVariable Long userId,
            @PathVariable Long orderId) {
        return "用户" + userId + "的订单" + orderId;
    }
    
    // 正则表达式约束路径变量
    @GetMapping("/users/{id:\\d+}")
    public User getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // 可选路径变量
    @GetMapping({"/posts/{id}", "/posts"})
    public Object getPosts(@PathVariable(required = false) Long id) {
        if (id != null) {
            return postService.findById(id);
        } else {
            return postService.findAll();
        }
    }
    
    // 矩阵变量
    @GetMapping("/products/{id}")
    public Product getProduct(
            @PathVariable Long id,
            @MatrixVariable String color,
            @MatrixVariable String size) {
        return productService.findWithAttributes(id, color, size);
    }
}
```

## 7. SSM 整合（Spring + Spring MVC + MyBatis）

### 7.1 项目结构

```
src/main/java/
├── com/example/
│   ├── config/
│   │   ├── SpringConfig.java
│   │   ├── SpringMvcConfig.java
│   │   └── MyBatisConfig.java
│   ├── controller/
│   ├── service/
│   ├── dao/
│   └── entity/
src/main/resources/
├── application.properties
└── mapper/
```

### 7.2 完整配置类

```java
// Spring 配置类
@Configuration
@ComponentScan(value = "com.example",
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = {Controller.class, RestController.class}
    )
)
@PropertySource("classpath:application.properties")
public class SpringConfig {
    
    @Bean
    public DataSource dataSource(
            @Value("${jdbc.driver}") String driver,
            @Value("${jdbc.url}") String url,
            @Value("${jdbc.username}") String username,
            @Value("${jdbc.password}") String password) {
        
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
    
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}

// MyBatis 配置类
@Configuration
public class MyBatisConfig {
    
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) throws IOException {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        sessionFactory.setTypeAliasesPackage("com.example.entity");
        
        // 配置 MyBatis 设置
        org.apache.ibatis.session.Configuration configuration = 
            new org.apache.ibatis.session.Configuration();
        configuration.setMapUnderscoreToCamelCase(true);
        configuration.setLogImpl(StdOutImpl.class); // 使用标准输出日志
        
        sessionFactory.setConfiguration(configuration);
        return sessionFactory;
    }
    
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer scanner = new MapperScannerConfigurer();
        scanner.setBasePackage("com.example.dao");
        return scanner;
    }
}

// Spring MVC 配置类
@Configuration
@ComponentScan(value = "com.example.controller")
@EnableWebMvc
public class SpringMvcConfig implements WebMvcConfigurer {
    
    // 静态资源处理
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/");
    }
    
    // 视图解析器
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
    
    // JSON 消息转换器
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter converter = 
            new MappingJackson2HttpMessageConverter();
        converters.add(converter);
    }
}
```

### 7.3 启动配置

```java
public class WebAppInitializer extends 
        AbstractAnnotationConfigDispatcherServletInitializer {
    
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class, MyBatisConfig.class};
    }
    
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }
    
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    
    // 配置字符编码过滤器
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceEncoding(true);
        return new Filter[]{encodingFilter};
    }
}
```

## 8. 拦截器（Interceptor）

### 8.1 拦截器与过滤器区别

| 特性      | 过滤器（Filter）         | 拦截器（Interceptor） |
| --------- | ------------------------ | --------------------- |
| 技术规范  | Servlet 规范             | Spring MVC 框架       |
| 作用范围  | 所有 Web 资源            | 仅 Controller 方法    |
| 依赖      | 依赖 Servlet 容器        | 依赖 Spring 容器      |
| 执行时机  | 在拦截器之前             | 在过滤器之后          |
| 获取 Bean | 不能直接获取 Spring Bean | 可以获取 Spring Bean  |

### 8.2 拦截器实现

```java
// 自定义拦截器
@Component
public class AuthInterceptor implements HandlerInterceptor {
    
    @Autowired
    private UserService userService;
    
    // 预处理方法 - 在 Controller 方法执行前调用
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        
        // 获取 token
        String token = request.getHeader("Authorization");
        if (token == null || !userService.validateToken(token)) {
            response.setStatus(HttpStatus.UNAUTHORIZED.value());
            response.getWriter().write("未授权访问");
            return false; // 阻止继续执行
        }
        
        // 设置用户信息到请求属性
        User user = userService.getUserByToken(token);
        request.setAttribute("currentUser", user);
        
        return true; // 继续执行
    }
    
    // 后处理方法 - 在 Controller 方法执行后，视图渲染前调用
    @Override
    public void postHandle(HttpServletRequest request, 
                         HttpServletResponse response, 
                         Object handler, 
                         ModelAndView modelAndView) throws Exception {
        // 可以修改 ModelAndView
        if (modelAndView != null) {
            modelAndView.addObject("timestamp", System.currentTimeMillis());
        }
    }
    
    // 完成方法 - 在整个请求完成后调用
    @Override
    public void afterCompletion(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler, 
                              Exception ex) throws Exception {
        // 资源清理、日志记录等
        if (ex != null) {
            log.error("请求处理异常", ex);
        }
    }
}

// 日志拦截器
@Component
public class LoggingInterceptor implements HandlerInterceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingInterceptor.class);
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        
        long startTime = System.currentTimeMillis();
        request.setAttribute("startTime", startTime);
        
        logger.info("请求开始: {} {} from {}", 
                   request.getMethod(), 
                   request.getRequestURI(),
                   request.getRemoteAddr());
        
        return true;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler, 
                              Exception ex) throws Exception {
        
        long startTime = (Long) request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        long duration = endTime - startTime;
        
        logger.info("请求完成: {} {} - 状态: {} - 耗时: {}ms", 
                   request.getMethod(), 
                   request.getRequestURI(),
                   response.getStatus(),
                   duration);
    }
}
```

### 8.3 拦截器配置

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    
    @Autowired
    private AuthInterceptor authInterceptor;
    
    @Autowired
    private LoggingInterceptor loggingInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 日志拦截器 - 拦截所有请求
        registry.addInterceptor(loggingInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/static/**", "/error");
        
        // 认证拦截器 - 拦截 API 请求
        registry.addInterceptor(authInterceptor)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/login", "/api/register");
        
        // 添加更多拦截器...
    }
    
    // 静态资源配置
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/", "file:./uploads/");
        
        registry.addResourceHandler("/uploads/**")
                .addResourceLocations("file:./uploads/");
    }
}
```

### 8.4 拦截器参数详解

```java
@Component
public class ParameterInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        
        // handler 参数分析
        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            
            // 获取控制器方法信息
            Method method = handlerMethod.getMethod();
            Class<?> beanType = handlerMethod.getBeanType();
            
            System.out.println("执行方法: " + method.getName());
            System.out.println("控制器类: " + beanType.getSimpleName());
            
            // 获取方法上的注解
            RequestMapping requestMapping = method.getAnnotation(RequestMapping.class);
            if (requestMapping != null) {
                System.out.println("请求路径: " + Arrays.toString(requestMapping.value()));
            }
        }
        
        // 请求参数分析
        System.out.println("请求方法: " + request.getMethod());
        System.out.println("请求URL: " + request.getRequestURL());
        System.out.println("查询参数: " + request.getQueryString());
        
        // 请求头分析
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String headerName = headerNames.nextElement();
            System.out.println(headerName + ": " + request.getHeader(headerName));
        }
        
        return true;
    }
}
```

## 9. 高级特性

### 9.1 全局异常处理

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Result<String> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.error("系统繁忙，请稍后重试");
    }
    
    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    public Result<String> handleBusinessException(BusinessException e) {
        return Result.error(e.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public Result<String> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult()
                         .getFieldErrors()
                         .stream()
                         .map(FieldError::getDefaultMessage)
                         .collect(Collectors.joining(", "));
        return Result.error("参数校验失败: " + message);
    }
}
```

### 9.2 跨域配置

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000", "https://example.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

## 总结

Spring MVC 提供了强大的 Web 开发能力，通过合理的配置和使用，可以构建出高性能、易维护的 Web 应用程序。关键要点包括：

1. **清晰的层次分离**：控制器、服务、数据访问层职责分明
2. **灵活的配置方式**：支持 XML 和注解两种配置方式
3. **强大的参数绑定**：自动处理各种类型的请求参数
4. **完善的拦截机制**：过滤器 + 拦截器提供完整的请求处理链路
5. **RESTful 支持**：轻松构建 REST API
6. **良好的扩展性**：支持自定义组件和全局配置

通过掌握这些核心概念和最佳实践，可以充分发挥 Spring MVC 框架的优势，提高开发效率和代码质量。