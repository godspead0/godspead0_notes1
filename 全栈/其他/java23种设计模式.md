# 

# Java 23种设计模式详解与对比

## 一、设计模式概述

设计模式是软件开发中**可复用的解决方案**，用于解决代码中反复出现的设计问题。它不是现成的代码，而是一套经过验证的、标准化的设计思路，核心价值在于：

- 提高代码可复用性、可维护性和扩展性
- 降低模块间耦合，使系统更灵活
- 便于团队协作（统一设计语言）

Java 23种设计模式按功能分为三大类：

- **创建型模式（5种）**：关注对象的创建机制，解耦对象创建与使用
- **结构型模式（7种）**：关注类/对象的组合与结构，优化类间关系
- **行为型模式（11种）**：关注对象间的交互与职责分配，优化流程逻辑

## 二、23种设计模式详细解析

### （一）创建型模式：对象创建的“工程化方案”

核心目标：**控制对象创建过程**，避免直接使用 `new` 关键字导致的耦合，支持灵活的对象实例化。	

#### 1. 单例模式（Singleton）

- **定义**：确保一个类在整个系统中**仅有一个实例**，并提供全局唯一的访问入口。
- **核心作用**：节省内存资源（避免重复创建重量级对象）、保证对象状态一致性。
- **应用场景**：Spring Bean默认单例、数据库连接池、日志工具类、配置类。
- **源码级实现思路**：
    - 饿汉式：类加载时直接初始化实例（线程安全，浪费内存）
    - 懒汉式：首次调用时创建实例（需加锁保证线程安全）
    - 双重校验锁（DCL）：`volatile + 双重if判断`（兼顾线程安全与性能）
    - 静态内部类：利用类加载机制实现懒加载+线程安全（推荐）
- **关键代码示例（DCL）**：

```Java
public class Singleton {
    // volatile 防止指令重排
    private static volatile Singleton instance;
    // 私有构造器禁止外部实例化
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) { // 第一次判断（避免频繁加锁）
            synchronized (Singleton.class) {
                if (instance == null) { // 第二次判断（防止多线程并发创建）
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

- **关键要点**：禁止反射破坏单例、序列化/反序列化时需重写 `readResolve()` 保持单例。

#### 2. 工厂方法模式（Factory Method）

- **定义**：定义一个创建对象的接口（工厂接口），由**子类决定具体实例化哪个类**，将对象创建延迟到子类。
- **核心作用**：解耦对象创建与使用，支持“开闭原则”（新增产品无需修改工厂接口）。
- **应用场景**：Spring的 `BeanFactory`（定义创建Bean的接口，子类如 `DefaultListableBeanFactory` 实现具体创建逻辑）、日志框架（不同日志实现的工厂）。
- **核心组件**：
    - 产品接口（Product）：定义产品规范
    - 具体产品（ConcreteProduct）：实现产品接口
    - 工厂接口（Factory）：定义创建产品的方法
    - 具体工厂（ConcreteFactory）：实现工厂接口，创建具体产品
- **关键要点**：一个工厂对应一个产品，适合产品种类较少、扩展单一的场景。

#### 3. 抽象工厂模式（Abstract Factory）

- **定义**：提供一个创建**产品族**（一组相关/相互依赖的产品）的接口，不指定具体产品类，支持多产品族的创建。
- **核心作用**：解决“产品族”的统一创建问题，保证产品间的兼容性。
- **应用场景**：Spring的 `ApplicationContext`（管理Bean工厂族）、数据库连接（不同数据库的Connection、Statement属于同一产品族）。
- **核心组件**：
    - 抽象产品族（AbstractProductA/AbstractProductB）：定义产品族中不同类型的产品
    - 具体产品（ConcreteProductA1/ConcreteProductB1）：某一产品族的具体实现
    - 抽象工厂（AbstractFactory）：定义创建产品族的所有方法
    - 具体工厂（ConcreteFactory1）：实现产品族的创建逻辑
- **与工厂方法的区别**：工厂方法关注“单个产品”，抽象工厂关注“产品族”（多个相关产品）。

#### 4. 建造者模式（Builder）

- **定义**：将复杂对象的**构建过程与表示分离**，通过分步构建（链式调用）生成复杂对象。
- **核心作用**：简化复杂对象的创建（避免多参数构造器），支持灵活配置对象属性。
- **应用场景**：MyBatis的 `SqlSessionFactoryBuilder`、Lombok的 `@Builder` 注解、复杂POJO的创建（如订单对象）。
- **核心组件**：
    - 产品（Product）：复杂对象本身
    - 建造者（Builder）：定义分步构建方法
    - 指挥者（Director）：控制构建流程（可选，简化版可省略）
- **关键代码示例**：

```Java
// 产品类
public class Order {
    private String id;
    private String productName;
    private int quantity;
    // 私有构造器，仅允许Builder创建
    private Order(Builder builder) {
        this.id = builder.id;
        this.productName = builder.productName;
        this.quantity = builder.quantity;
    }
    // 建造者内部类
    public static class Builder {
        private String id;
        private String productName;
        private int quantity;
        public Builder id(String id) {
            this.id = id;
            return this;
        }
        public Builder productName(String productName) {
            this.productName = productName;
            return this;
        }
        public Builder quantity(int quantity) {
            this.quantity = quantity;
            return this;
        }
        public Order build() {
            return new Order(this);
        }
    }
}
// 使用：Order order = new Order.Builder().id("1001").productName("手机").quantity(2).build();
```

- **关键要点**：适合属性多、配置灵活的复杂对象，链式调用提升代码可读性。

#### 5. 原型模式（Prototype）

- **定义**：通过**复制现有对象**（原型）创建新对象，无需重新初始化，减少实例化开销。
- **核心作用**：优化重量级对象的创建（如数据库连接、大尺寸对象），支持动态获取对象状态。
- **应用场景**：Spring Bean的 `prototype` 作用域、克隆对象（如ArrayList的 `clone()` 方法）。
- **实现方式**：
    - 浅克隆：实现 `Cloneable` 接口，重写 `clone()` 方法（仅复制基本类型，引用类型共享）
    - 深克隆：通过序列化（`Serializable`）实现（复制所有属性，包括引用类型）
- **关键要点**：深克隆需处理所有引用类型的序列化，避免循环引用问题。

### （二）结构型模式：类/对象的“组合优化方案”

核心目标：**优化类与对象的组合关系**，在不改变原有结构的前提下，扩展功能或适配接口。

#### 6. 适配器模式（Adapter）

- **定义**：将一个类的接口**转换成客户端期望的另一个接口**，使原本不兼容的类可以一起工作。
- **核心作用**：解决接口不兼容问题，复用现有代码。
- **应用场景**：Spring MVC的 `HandlerAdapter`（适配不同类型的处理器Handler）、JDBC的 `DriverAdapter`（适配不同数据库驱动）。
- **实现方式**：
    - 类适配器：继承被适配类 + 实现目标接口（Java单继承限制，灵活性低）
    - 对象适配器：持有被适配类实例 + 实现目标接口（推荐，解耦）
- **关键代码示例（对象适配器）**：

```Java
// 被适配类（旧接口）
public class OldCalculator {
    public int add(int a, int b) {
        return a + b;
    }
}
// 目标接口（新接口）
public interface NewCalculator {
    int compute(int a, int b);
}
// 适配器类
public class CalculatorAdapter implements NewCalculator {
    private OldCalculator oldCalculator;
    public CalculatorAdapter(OldCalculator oldCalculator) {
        this.oldCalculator = oldCalculator;
    }
    @Override
    public int compute(int a, int b) {
        // 适配旧接口到新接口
        return oldCalculator.add(a, b);
    }
}
```

- **与装饰器的区别**：适配器改变接口，装饰器不改变接口，仅增强功能。

#### 7. 桥接模式（Bridge）

- **定义**：将**抽象部分与实现部分分离**，使两者可以独立扩展（抽象类依赖实现接口，而非直接继承）。
- **核心作用**：解决“多维度变化”问题（如产品类型+品牌、形状+颜色），避免类爆炸。
- **应用场景**：JDBC的 `Driver` 接口（抽象：Connection，实现：MySQLDriver/OracleDriver）、GUI组件（窗口+主题）。
- **核心组件**：
    - 抽象角色（Abstraction）：定义抽象接口，持有实现角色引用
    - 扩展抽象角色（RefinedAbstraction）：扩展抽象角色
    - 实现角色（Implementor）：定义实现接口
    - 具体实现角色（ConcreteImplementor）：实现接口
- **关键要点**：抽象与实现通过“组合”而非“继承”关联，支持独立扩展。

#### 8. 组合模式（Composite）

- **定义**：将对象组合成**树形结构**，统一处理“单个对象”和“组合对象”（叶子节点+容器节点）。
- **核心作用**：简化客户端代码，无需区分单个对象和组合对象的处理逻辑。
- **应用场景**：Spring的 `ApplicationContext`（BeanDefinition的树形结构）、文件系统（文件+文件夹）、菜单系统（菜单项+子菜单）。
- **核心组件**：
    - 组件（Component）：定义叶子节点和容器节点的统一接口
    - 叶子节点（Leaf）：无子节点的具体组件
    - 容器节点（Composite）：包含子节点，实现组件接口的管理方法（add/remove）
- **关键要点**：树形结构+统一接口，客户端遍历树形结构时无需判断节点类型。

#### 9. 装饰器模式（Decorator）

- **定义**：动态地给对象**添加额外功能**，不改变原类结构和接口，支持多层装饰。
- **核心作用**：替代继承的扩展方式，灵活组合功能（遵循“开闭原则”）。
- **应用场景**：Java IO流（`BufferedReader` 装饰 `Reader`、`DataInputStream` 装饰 `InputStream`）、Spring的 `TransactionAwareCacheDecorator`。
- **核心组件**：
    - 抽象组件（Component）：定义被装饰对象的接口
    - 具体组件（ConcreteComponent）：被装饰的原始对象
    - 装饰器（Decorator）：持有抽象组件引用，实现组件接口
    - 具体装饰器（ConcreteDecorator）：实现额外功能
- **关键代码示例**：

```Java
// 抽象组件
public interface Coffee {
    String getName();
    double getPrice();
}
// 具体组件（原始对象）
public class SimpleCoffee implements Coffee {
    @Override
    public String getName() {
        return "纯咖啡";
    }
    @Override
    public double getPrice() {
        return 10.0;
    }
}
// 装饰器
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}
// 具体装饰器（加牛奶）
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    @Override
    public String getName() {
        return coffee.getName() + "+牛奶";
    }
    @Override
    public double getPrice() {
        return coffee.getPrice() + 3.0;
    }
}
// 使用：Coffee coffee = new MilkDecorator(new SimpleCoffee());
```

- **与适配器的区别**：装饰器增强功能不改变接口，适配器改变接口适配需求。

#### 10. 外观模式（Facade）

- **定义**：为复杂子系统提供一个**统一的访问接口**，隐藏子系统内部的复杂逻辑，简化客户端调用。
- **核心作用**：降低客户端与子系统的耦合，简化调用流程。
- **应用场景**：Spring的 `JdbcTemplate`（封装JDBC的复杂操作：加载驱动、创建连接、执行SQL、关闭资源）、MyBatis的 `SqlSession`。
- **核心组件**：
    - 外观角色（Facade）：提供统一接口，内部调用子系统组件
    - 子系统角色（Subsystem）：复杂逻辑的具体实现者
- **关键要点**：外观类不新增功能，仅封装子系统的调用逻辑，客户端无需关注子系统细节。

#### 11. 享元模式（Flyweight）

- **定义**：共享细粒度对象（享元对象），减少内存消耗，适用于大量重复对象的场景。
- **核心作用**：复用对象，降低内存占用（如缓存池）。
- **应用场景**：Integer的 `valueOf()` 方法（缓存-128~127的整数）、String常量池、数据库连接池、线程池。
- **核心组件**：
    - 享元工厂（FlyweightFactory）：创建并管理享元对象（缓存）
    - 享元对象（Flyweight）：可共享的对象（内部状态不变，外部状态通过参数传入）
- **关键要点**：区分内部状态（共享）和外部状态（不共享），外部状态通过方法参数传递。

#### 12. 代理模式（Proxy）

- **定义**：为其他对象提供一个**代理对象**，控制对原对象的访问，可在访问前后添加额外逻辑（拦截/增强）。
- **核心作用**：解耦业务逻辑与横切逻辑（如日志、事务、权限），保护原对象。
- **应用场景**：Spring AOP（动态代理实现）、MyBatis Mapper代理、RPC框架的服务代理。
- **实现方式**：
    - JDK动态代理：基于**接口**生成代理对象（`Proxy.newProxyInstance()` + `InvocationHandler`），无需依赖第三方库。
    - CGLIB动态代理：基于**类**生成代理对象（继承原类），支持无接口类的代理，需依赖CGLIB库。
    - JDK里面有一个操作类叫做Proxy，里面有一个newProxyInstance方法可以生成代理对象，其中三个参数是classloader类加载器，class<?>[] interfaces接口集合(JDK动态代理只能基于接口进行代理)，InvocationHandler代理拦截器(其中调用的是invoke方法)。这个过程要使用一个对象对这个动态代理对象进行接受(可以强转成传入接口的某个具体形式)，这样就可以对这个接口的方法进行调用(此时接受的是一个“Proxy@雪花算法生成数”类型的)。调用之后的数据传到InvocationHandler代理拦截器中的invoke方法里面(包括对象本身，参数等等)，整个过程就是先生成一个代理对象，这个对象会基于类加载器实现接口数组里的所有接口方法，然后传一个拦截器，拦截器中的方法是可以随意写的
- **关键代码示例（JDK动态代理）**：

```Java
// 目标接口
public interface UserService {
    void addUser();
}
// 目标实现类
public class UserServiceImpl implements UserService {
    @Override
    public void addUser() {
        System.out.println("新增用户");
    }
}
// 代理拦截器
public class MyInvocationHandler implements InvocationHandler {
    private Object target; // 目标对象
    public MyInvocationHandler(Object target) {
        this.target = target;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 前置增强（如日志、权限校验）
        System.out.println("方法执行前：记录日志");
        // 调用目标方法
        Object result = method.invoke(target, args);
        // 后置增强（如事务提交）
        System.out.println("方法执行后：事务提交");
        return result;
    }
}
// 测试
public class ProxyTest {
    public static void main(String[] args) {
        UserService target = new UserServiceImpl();
        // 生成代理对象
        UserService proxy = (UserService) Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            new MyInvocationHandler(target)
        );
        proxy.addUser(); // 调用代理对象方法，触发拦截
    }
}
```

- **关键要点**：JDK代理依赖接口，CGLIB依赖继承；Spring AOP默认优先使用JDK动态代理，无接口时使用CGLIB。

### （三）行为型模式：对象交互的“流程优化方案”

核心目标：**优化对象间的交互方式**，明确对象职责，简化流程逻辑，支持灵活扩展。

#### 13. 策略模式（Strategy）

- **定义**：定义一组**算法族**，将每个算法封装成独立类，支持动态切换算法（客户端选择算法）。
- **核心作用**：替代多重 `if-else` 或 `switch`，遵循“开闭原则”（新增算法无需修改原有代码）。
- **应用场景**：Spring的 `Resource` 接口（不同资源加载策略：文件、ClassPath、URL）、排序算法切换、支付方式选择（微信/支付宝/银行卡）。
- **核心组件**：
    - 策略接口（Strategy）：定义算法规范
    - 具体策略（ConcreteStrategy）：实现算法
    - 上下文（Context）：持有策略接口引用，提供切换策略的方法
- **关键要点**：客户端需了解所有策略，负责选择合适的策略。

#### 14. 模板方法模式（Template Method）

- **定义**：定义一个算法的**骨架**（抽象类），将算法的可变步骤延迟到子类实现，固定算法流程。
- **核心作用**：复用算法骨架，子类仅需实现差异化步骤（遵循“开闭原则”）。
- **应用场景**：Spring的 `JdbcTemplate`（`execute()` 方法定义SQL执行骨架，子类实现 `RowMapper`）、JUnit的 `TestCase`（`setUp()`/`tearDown()` 固定测试流程）。
- **核心组件**：
    - 抽象模板（AbstractClass）：定义算法骨架（模板方法）和抽象步骤
    - 具体模板（ConcreteClass）：实现抽象步骤
- **关键代码示例**：

```Java
// 抽象模板
public abstract class AbstractOrderService {
    // 模板方法（算法骨架，final禁止子类重写）
    public final void processOrder() {
        validateOrder(); // 固定步骤1：校验订单
        calculatePrice(); // 可变步骤：计算价格（子类实现）
        saveOrder(); // 固定步骤2：保存订单
        notifyUser(); // 可变步骤：通知用户（子类实现）
    }
    // 固定步骤（父类实现）
    private void validateOrder() {
        System.out.println("校验订单合法性");
    }
    private void saveOrder() {
        System.out.println("保存订单到数据库");
    }
    // 可变步骤（抽象方法，子类实现）
    protected abstract void calculatePrice();
    protected abstract void notifyUser();
}
// 具体模板（普通订单）
public class NormalOrderService extends AbstractOrderService {
    @Override
    protected void calculatePrice() {
        System.out.println("普通订单：按原价计算");
    }
    @Override
    protected void notifyUser() {
        System.out.println("普通订单：短信通知用户");
    }
}
```

- **关键要点**：模板方法用 `final` 修饰，避免子类修改算法流程；抽象方法定义可变步骤。

#### 15. 观察者模式（Observer）

- **定义**：定义对象间的**一对多依赖关系**，当一个对象（主题）状态变化时，所有依赖它的对象（观察者）会自动收到通知并更新。
- **核心作用**：解耦主题与观察者，支持动态添加/移除观察者。
- **应用场景**：Spring的 `ApplicationEvent` 事件机制（`ApplicationEventPublisher` 发布事件，`ApplicationListener` 监听事件）、GUI组件的事件响应（按钮点击、文本框输入）。
- **核心组件**：
    - 主题（Subject）：持有观察者列表，提供注册/移除/通知方法
    - 观察者（Observer）：定义接收通知的更新方法
- **JDK内置支持**：`java.util.Observable`（主题）和 `java.util.Observer`（观察者）接口。
- **关键要点**：主题仅依赖观察者接口，不依赖具体实现；观察者可动态注册/注销。

#### 16. 迭代器模式（Iterator）

- **定义**：提供一种方法**遍历集合对象**，而不暴露集合的内部结构（如数组、链表）。
- **核心作用**：统一集合遍历方式，解耦集合与遍历逻辑。
- **应用场景**：Java的 `Iterator` 接口（ArrayList、HashMap等集合的遍历）、Spring的 `Iterable` 接口实现。
- **核心组件**：
    - 迭代器（Iterator）：定义遍历方法（`hasNext()`、`next()`）
    - 集合（Aggregate）：提供创建迭代器的方法
- **关键要点**：客户端通过迭代器遍历集合，无需关注集合的底层存储结构。

#### 17. 责任链模式（Chain of Responsibility）

- **定义**：将请求的处理者组成一条**责任链**，请求沿链传递，直到被某个处理者处理（或链结束）。
- **核心作用**：解耦请求发送者与接收者，支持动态调整责任链顺序。
- **应用场景**：Spring MVC的 `Interceptor` 拦截器链、Servlet的 `Filter` 链、异常处理链、审批流程（部门审批→公司审批→总部审批）。
- **核心组件**：
    - 抽象处理者（Handler）：定义处理请求的方法和设置下一个处理者的方法
    - 具体处理者（ConcreteHandler）：实现处理逻辑，若无法处理则传递给下一个处理者
- **关键代码示例**：

```Java
// 抽象处理者
public abstract class Handler {
    protected Handler nextHandler;
    // 设置下一个处理者
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }
    // 处理请求的抽象方法
    public abstract void handleRequest(String request);
}
// 具体处理者1（部门审批）
public class DepartmentHandler extends Handler {
    @Override
    public void handleRequest(String request) {
        if ("部门级审批".equals(request)) {
            System.out.println("部门审批通过");
        } else {
            // 无法处理，传递给下一个处理者
            if (nextHandler != null) {
                nextHandler.handleRequest(request);
            } else {
                System.out.println("无对应审批流程");
            }
        }
    }
}
// 具体处理者2（公司审批）
public class CompanyHandler extends Handler {
    @Override
    public void handleRequest(String request) {
        if ("公司级审批".equals(request)) {
            System.out.println("公司审批通过");
        } else {
            if (nextHandler != null) {
                nextHandler.handleRequest(request);
            } else {
                System.out.println("无对应审批流程");
            }
        }
    }
}
// 使用：Handler chain = new DepartmentHandler(); chain.setNextHandler(new CompanyHandler()); chain.handleRequest("公司级审批");
```

- **关键要点**：处理者仅需关注自身职责，无需知道整个链的结构；支持动态增减处理者。

#### 18. 命令模式（Command）

- **定义**：将请求封装成**命令对象**，使请求的发送者与接收者解耦，支持请求的撤销/重做、队列化执行。
- **核心作用**：解耦请求发送者（如按钮）与接收者（如业务逻辑），支持请求的灵活管理。
- **应用场景**：Spring的 `JmsTemplate`（消息发送命令）、GUI按钮点击（按钮是发送者，业务逻辑是接收者）、事务管理（命令的提交/回滚）。
- **核心组件**：
    - 命令（Command）：定义执行请求的接口（`execute()`）
    - 具体命令（ConcreteCommand）：绑定发送者与接收者，实现 `execute()`
    - 发送者（Invoker）：调用命令对象执行请求
    - 接收者（Receiver）：执行具体业务逻辑
- **关键要点**：命令对象封装了请求的所有信息（接收者、参数），支持请求的延迟执行、批量执行。

#### 19. 备忘录模式（Memento）

- **定义**：在不破坏封装的前提下，保存对象的**内部状态**，以便后续恢复到该状态（快照模式）。
- **核心作用**：实现对象状态的“回滚”，如撤销操作、事务回滚。
- **应用场景**：Spring的事务管理（保存事务执行前的状态，失败时回滚）、文本编辑器的撤销功能、游戏存档。
- **核心组件**：
    - 原发器（Originator）：创建备忘录并恢复状态
    - 备忘录（Memento）：存储原发器的状态（私有构造器，仅允许原发器访问）
    - 管理者（Caretaker）：管理备忘录（保存/获取，不操作状态）
- **关键要点**：备忘录封装状态，避免外部访问；管理者仅负责存储，不参与状态逻辑。

#### 20. 状态模式（State）

- **定义**：当对象的**内部状态变化时**，改变其行为（对象看起来像改变了类），将状态逻辑封装到独立的状态类中。
- **核心作用**：替代多重 `if-else` 判断状态，将状态逻辑解耦到独立类。
- **应用场景**：Spring的 `StateMachine`（状态机框架）、订单状态流转（待支付→已支付→已发货→已完成）、电梯状态（开门→关门→运行→停止）。
- **核心组件**：
    - 上下文（Context）：持有当前状态，委托状态类处理行为
    - 状态（State）：定义状态对应的行为接口
    - 具体状态（ConcreteState）：实现特定状态的行为
- **关键要点**：状态类独立，上下文无需判断状态，仅需切换状态对象；支持动态切换状态。

#### 21. 访问者模式（Visitor）

- **定义**：分离对象的**数据结构**和**操作逻辑**，新增操作时无需修改数据结构类，仅需新增访问者类。
- **核心作用**：解决“数据结构稳定但操作频繁变化”的问题，遵循“开闭原则”。
- **应用场景**：Java的 `AnnotationProcessor`（注解处理）、XML/JSON解析（不同解析规则作为访问者）、报表生成（不同报表类型作为访问者）。
- **核心组件**：
    - 访问者（Visitor）：定义对数据结构的操作接口
    - 具体访问者（ConcreteVisitor）：实现操作逻辑
    - 元素（Element）：定义接收访问者的方法（`accept()`）
    - 具体元素（ConcreteElement）：实现 `accept()`，调用访问者的操作
- **关键要点**：数据结构与操作分离，新增操作仅需新增访问者；但数据结构变化时，所有访问者需修改（适合数据结构稳定的场景）。

#### 22. 中介者模式（Mediator）

- **定义**：用一个**中介者对象**封装对象间的交互，使对象间无需直接通信，降低耦合。
- **核心作用**：减少对象间的直接依赖，将多对多关系转化为一对多关系。
- **应用场景**：Spring的 `ApplicationContext`（Bean间的依赖通过容器中介）、MVC框架（Controller作为View和Model的中介）、聊天室（服务器作为用户间的中介）。
- **核心组件**：
    - 中介者（Mediator）：定义对象间交互的接口
    - 具体中介者（ConcreteMediator）：实现交互逻辑，持有所有同事对象引用
    - 同事（Colleague）：定义与中介者交互的方法
- **关键要点**：中介者集中管理对象交互，同事对象仅与中介者通信，不直接交互。

#### 23. 解释器模式（Interpreter）

- **定义**：定义一种语言的**文法规则**，并构建一个解释器来解释该语言的语句（如表达式、配置文件）。
- **核心作用**：解析自定义语言或表达式，支持灵活扩展文法规则。
- **应用场景**：Spring的EL表达式解析、SQL解析（MyBatis的动态SQL）、正则表达式解析、数学表达式计算。
- **核心组件**：
    - 抽象表达式（AbstractExpression）：定义解释方法
    - 终结符表达式（TerminalExpression）：解析文法中的终结符（如变量、常量）
    - 非终结符表达式（NonTerminalExpression）：解析文法中的非终结符（如运算符）
    - 上下文（Context）：存储解释器的全局状态
- **关键要点**：适合简单文法规则，复杂文法会导致类爆炸（如SQL解析更适合用ANTLR等专业工具）。

## 三、23种设计模式对比总结

| 模式分类   | 模式名称   | 核心意图                         | 适用场景                      | 典型框架应用                        | 关键特点                               | 注意事项                         |
| ---------- | ---------- | -------------------------------- | ----------------------------- | ----------------------------------- | -------------------------------------- | -------------------------------- |
| **创建型** | 单例模式   | 确保唯一实例                     | 重量级对象、全局配置          | Spring Bean（默认单例）             | 全局唯一访问入口                       | 防止反射/序列化破坏单例          |
|            | 工厂方法   | 延迟对象创建到子类               | 单一产品扩展                  | Spring BeanFactory                  | 一个工厂对应一个产品                   | 产品过多时工厂类膨胀             |
|            | 抽象工厂   | 创建产品族                       | 多相关产品的统一创建          | Spring ApplicationContext           | 一个工厂对应多个相关产品               | 产品族扩展复杂                   |
|            | 建造者     | 分步构建复杂对象                 | 多属性、灵活配置的对象        | MyBatis SqlSessionFactoryBuilder    | 链式调用，分离构建与表示               | 简单对象无需使用                 |
|            | 原型模式   | 复制对象创建新实例               | 重量级对象、重复对象          | Spring Bean（prototype作用域）      | 减少实例化开销                         | 深克隆需处理序列化/循环引用      |
| **结构型** | 适配器     | 接口适配（不兼容→兼容）          | 旧接口复用、跨系统集成        | Spring MVC HandlerAdapter           | 改变接口，不改变功能                   | 避免过度使用导致系统复杂         |
|            | 桥接模式   | 抽象与实现分离，独立扩展         | 多维度变化（如类型+品牌）     | JDBC Driver接口                     | 组合替代继承，解耦抽象与实现           | 抽象层与实现层需独立设计         |
|            | 组合模式   | 树形结构统一处理单个/组合对象    | 层级结构（文件系统、菜单）    | Spring ApplicationContext（Bean树） | 统一接口，无需区分节点类型             | 叶子节点与容器节点的方法一致性   |
|            | 装饰器     | 动态增强功能，不改变接口         | 功能组合、多层增强            | Java IO流、Spring AOP装饰器         | 多层装饰，灵活组合功能                 | 避免装饰链过长影响可读性         |
|            | 外观模式   | 简化子系统访问接口               | 复杂子系统、客户端简化调用    | Spring JdbcTemplate                 | 隐藏子系统细节，统一入口               | 不新增功能，仅封装调用逻辑       |
|            | 享元模式   | 共享细粒度对象，减少内存         | 大量重复对象、缓存场景        | Integer缓存、String常量池           | 区分内部/外部状态，共享内部状态        | 外部状态需通过参数传递           |
|            | 代理模式   | 控制对象访问，添加拦截逻辑       | AOP、权限控制、RPC代理        | Spring AOP、MyBatis Mapper代理      | JDK（接口）/CGLIB（类）两种实现        | 避免代理链过长影响性能           |
| **行为型** | 策略模式   | 动态切换算法族                   | 替代多重if-else、算法灵活选择 | Spring Resource加载策略             | 算法封装，客户端选择策略               | 客户端需了解所有策略             |
|            | 模板方法   | 固定算法骨架，延迟可变步骤到子类 | 算法流程固定、步骤差异化      | Spring JdbcTemplate                 | 模板方法final修饰，子类实现抽象步骤    | 避免骨架逻辑过于复杂             |
|            | 观察者模式 | 一对多依赖，状态变化通知         | 事件通知、订阅-发布           | Spring ApplicationEvent             | 动态注册/移除观察者，解耦主题与观察者  | 避免循环依赖导致通知死循环       |
|            | 迭代器模式 | 统一集合遍历，隐藏内部结构       | 集合遍历、数据结构无关遍历    | Java Iterator接口                   | 遍历逻辑与集合解耦                     | 只读遍历，避免遍历中修改集合     |
|            | 责任链模式 | 请求沿链传递，直到被处理         | 拦截器、审批流程、异常处理    | Spring MVC Interceptor              | 动态调整链顺序，解耦请求发送者与接收者 | 避免链过长导致性能问题           |
|            | 命令模式   | 封装请求，支持撤销/队列化        | 事务管理、消息队列、GUI操作   | Spring JmsTemplate                  | 解耦发送者与接收者，支持请求管理       | 简单请求无需使用，增加类复杂度   |
|            | 备忘录模式 | 保存对象状态，支持恢复           | 撤销操作、事务回滚、存档      | Spring事务管理                      | 封装状态，不破坏对象封装               | 状态过大时占用内存               |
|            | 状态模式   | 状态变化改变对象行为             | 状态流转（订单、电梯）        | Spring StateMachine                 | 状态逻辑封装到独立类，无需if-else      | 状态过多时类膨胀                 |
|            | 访问者模式 | 分离数据结构与操作逻辑           | 稳定数据结构、频繁变化操作    | Spring EL表达式解析                 | 新增操作无需修改数据结构               | 数据结构变化时需修改所有访问者   |
|            | 中介者模式 | 封装对象交互，减少直接依赖       | 多对象耦合、分布式通信        | Spring ApplicationContext           | 多对多→一对多，集中管理交互            | 中介者可能成为性能瓶颈/紧耦合点  |
|            | 解释器模式 | 解析自定义文法规则               | 表达式解析、配置解析          | Spring EL、MyBatis动态SQL           | 支持文法扩展                           | 复杂文法导致类爆炸，适合简单场景 |

## 四、设计模式选择原则

1. **优先考虑问题本质**：设计模式是解决方案，需先明确问题（如对象创建→创建型，接口不兼容→适配器）。
2. **遵循设计原则**：优先满足“开闭原则”（扩展开放、修改关闭）、“单一职责”（一个类只做一件事）。
3. **拒绝过度设计**：简单问题无需使用复杂模式（如简单对象无需建造者模式）。
4. **结合框架特性**：Spring、MyBatis等框架已内置大量设计模式，优先复用框架能力（如Spring AOP无需手动实现代理模式）。
5. **团队共识优先**：选择团队熟悉的模式，避免为了用模式而用模式。