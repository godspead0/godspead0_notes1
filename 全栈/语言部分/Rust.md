# Rust

## 一、基本语法和数据类型

### 1. 变量和可变性

- 声明变量

- ```rust
    let some_number = 1;
    //默认：变量是不可变的
    ```

- 声明可变变量

- ```rust
    let mut another_number = 1;
    ```

### 2. 常量

- 声明常量

- ```rust
    const SPECIAL_NUMBER:i32 = 3;
    const THREE_HOURS_IN_SCEONDS:u32 = 60*60*3;
    ```

- 使用const声明

- 不可以使用mut

- 必须标注类型

- 可在任意作用域声明

- 仅可以使用常量表达式赋值，即在编译时即可确定值

### 3. 变量遮蔽（Shadowing）

- ```rust
    let my_number =1;
    let my_number =2;
    ```

- 可以用与之前变量相同的名字声明一个新变量：第一个变量被第二个变量遮蔽了

- 就是创建了一个新变量，只是名字相同

- 作用：在不同的作用域可以有不同的值

### 4. 数据类型

- 标量类型（表示一个单一的值）

    - 整数（Integer）

        - 具体类型：

        - | Length长度               | Signed有符号 | Unsigned无符号 |
            | ------------------------ | ------------ | -------------- |
            | 8-bit                    | i8           | u8             |
            | 16-bit                   | i16          | u16            |
            | 32-bit                   | i32          | u32            |
            | 64-bit                   | i64          | u64            |
            | 128-bit                  | i128         | u128           |
            | arch（和计算机架构有关） | isize        | usize          |

        - 整型字面值：

        - | 数值字面值        | 示例        |
            | ----------------- | ----------- |
            | 十进制（Decimal） | 98_222      |
            | 十六进制（Hex）   | 0xff        |
            | 八进制（Octal）   | 0o77        |
            | 二进制（Binary）  | 0b1111_0000 |
            | 字节（Byte）      | b'A'        |

            

    - 浮点（Floating Point）

        - 具体类型：

        - | 长度            | 表示（都有符号） |
            | --------------- | ---------------- |
            | 4字节（32-bit） | f32              |
            | 8字节（64-bit） | f64（默认）      |

        - 可以表示小数

    - 布尔（Boolean）

        - 两个值：true，false
        - 1字节（8-bit）
        - 类型名：bool

    - 字符（Character）

        - 4字节（32-bit）
        - 类型名：char
        - 可以表示一个Unicode标量值
        - 字符使用单引号

- 复合类型（可以将多个值组合在一个类型）

    - 元组Tuple

        - ```rust
            let my_tuple = ('A', 1, 1.2);
            let tup:(i32, f64, u8) = (500, 16.4, 1);
            let five_hundred = tup.0;
            let (x, y ,z) = tup;//解构
            ```

        - 固定长度

        - 可含不同类型的元素

        - 索引从0开始

    - 数组Array

        - ```rust
            let my_arr = [1, 2, 3];
            let my_arr_typed:[i32; 3] = [1, 2, 3];
            let a = [3; 5];//let a = [3, 3, 3, 3, 3];
            let first = my_arr[0];
            ```

        - 固定长度

        - 元素类型相同

        - 索引从0开始

## 二、函数和控制流程

### 1. 函数

- 函数和变量的命名规范（snake case）
    - 所有字母都小写
    - 单词之间使用'_'连接
- 参数
    - 必须声明每个参数的类型：`参数名:类型`
    - 多个参数需使用','分开
- 语句和表达式
    - 语句：执行某些操作的指令，不能返回值
    - 表达式：计算并返回一个结果值
    - 函数体一般是由一系列语句组成，可由表达式结尾
    - 例如：x + 1是表达式，有返回值；x + 1;是语句，无返回值。即如果需要返回值则不能在后面加分号
- 返回值
    - 使用"->"声明函数的返回值的类型
    - 可以使用return返回值，也可以使用中最后一个表达式的值

### 2. 控制流

- if表达式

    - 和Java/c类似，区别是后面的条件表达式不需要小括号

    - if

    - else

    - else if

    - if语句的返回值就是满足条件的{}作用域的返回值，可以用于变量的赋值，但是用于赋值时，所有{}的类型必须一致

    - ```rust
        let condition = true;
        let number = if condition {5} else {6};
        ```

    - 

- 循环

    - loop
        - 无限循环
        - 只能通过break停止循环和continue跳过本次循环
        - break后面跟的值就是loop的返回值
        - 嵌套循环，使用loop标签
            - 在外层循环的loop前面加上`'标签名`，在内层循环的break后面加上`'标签名`，即可指定从内层循环跳到外层循环，而不是直接跳出所有循环
    - while
        - 和Java中的while类似，同样whlie后面的条件表达式不要加小括号
    - for
        - 循环遍历集合元素
        - `for 元素 in 集合 {}`
        - 类似Java中的增强for循环
        - Range：可用于让for循环执行特定次数`for 元素 in (1..4) {}`(大于等于1小于4)

## 三、所有权

> 所有权：确保Rust程序安全的一种机制
>
> 安全：程序中没用未定义行为
>
> 未定义行为：当执行一段代码时，结果不可预测且未被编程语言指定的情况
>
> Rust的一个基础目标：是确保你的程序永远不会有未定义的行为
>
> Rust的一个次要目标：是在编译时而不是运行时防止未定义的行为

### 1. 所有权作为内存安全的规范

- 变量存在于Stack中（栈内存）

    - Stack Frame是每个函数调用期间用来存储该函数的局部变量、参数、返回值的一个内存空间
    - Stack Frame按照调用顺序存到Stack中

- Box存活于Heap中（堆内存）

    - 理论上数据可以在Heap堆中无限期的存活

    - box可以将数据放在堆中

    - ```rust
        let a = box::new([0; 1_000_000]);
        let b = a;
        ```

    - 在Stack中的Stack Frame中的变量是指针存储的是数据的地址，指针则指向该地址空间对应的堆（Heap）内存

    - Rust会自动释放Box的堆内存

    - **Box内存释放原则**：如果一个变量拥有一个Box，当Rust释放变量的Frame时，Rust也会释放Box的堆内存

    - 变量在被移动后不能再使用（函数调用也会移动所用权）

    - 移动堆（heap）数据的原则：如果变量x将堆（heap）数据的所有权移动给另一个变量y，那么在移动后，x不能再使用

    - clone可以避免移动：避免数据移动的一种方法是使用.clone()方法进行克隆

- Rust不允许手动管理内存

- Stack Frame由Rust自动管理

    - 当调用一个函数时，Rust为被调用的函数分配一个Stack Frame。当调用结束时，Rust释放该Stack Frame

### 2. 修复所有权错误

- 函数返回了一个引用，该引用指向Stack
    - 可以不返回引用，直接返回所有权
    - 返回值类型设置为`'static str`，函数体改为需要返回的字面量
    - 使用RC指针（可对引用计数的指针）
    - 直接修改原数据的可变引用
- 没有足够的权限
    - 将不可变的引用改为可变引用以获得写的权限
    - 在函数里面将数据复制clone一份，但会有额外的内存占用
    - 使用join生成一份新的String
- 同时使用了别名和可变性
    - 使用clone
    - 新建一个集合来存储之前使用到的集合的元素，让之前的集合恢复权限，但是占内存
    - 将获取元素改为获取元素的属性，这样原集合就不会失去权限
- 从集合中拷贝一个元素，移出一个元素的所有权
    - 如果一个值不拥有堆数据，那么它可以在不移动的情况下被复制
    - i32、&String不拥有堆数据，String拥有堆数据；即String类型需要复制时是获取的数据的所有权
    - 当然String类型也可以通过clone方法进行克隆
- 修改Tuple中不同字段的问题
    - 对Tuple中的一个元素进行借用，在借用期间整个Tuple和被借用的元素失去写的权限，但是对该Tuple中其他未被借用的元素，仍然具有写的权限，前提是该借用是被确定的，显式的，如果程序不能事先知道借用的是哪个，则会默认全部都有可能被借用（对于函数只看函数签名）
- 修改数组的不同元素
    - 已经对数组的元素进行了可变的借用过程中，就不能对该数组的任何元素进行不可变的借用了

### 3. SLICE切片

- 创建：[起始索引..结束索引]，不包括结束索引位置的元素

- Slice是特殊的引用类型，因为他们是“fat”（宽/肥/胖）指针，带有元数据

- 只记录**起始地址和长度**，不进行拷贝，减少内存占用

- 切片是对原数据的不可变的引用

- Range语法

    - [0..2]：大于等于0小于2
    - [..2]：大于等于0小于2
    - [3..len]：大于等于3小于最大长度
    - [3..]：大于等于3小于最大长度
    - [0..len]：全部
    - [..]：全部

- 字符串切片范围索引必须出现在有效的 UTF-8 字符边界上。如果你尝试在多字节字符的中间创建字符串切片，程序将会出错并退出。

- 字符串字面值是切片，类型是&str

- ```rust
    let s = "hello world";
    //s 的类型是&str，它是指向二进制文件特定位置的切片
    //这也是为什么字符串字面量是不可变的
    //&str 是一个不可变的引用
    ```

- 数组的切片

- ```rust
    let slice: &[i32] = &a[1..3];
    ```

## 四、引用和借用

### 1. 引用

- 引用是没用所有权的指针

- 引用在变量前面加上&

- ```rust
    use std::fmt::format;
    
    fn main() {
        let m1 = String::from("hello");
        let m2 = String::from("world");
        //greet(m1, m2);
    
        // 移动所有权的方法
        //let (m1_again, m2_again) = greet(m1, m2);
        //let s = format!("{} {}", m1_again, m2_again);
    
        //引用的方法
        greet(&m1, &m2);
        let s = format!("{} {}", m1, m2);
    }
    
    /*fn greet(g1: String, g2: String) {
        println!("{} {}!", g1, g2);
    }*/
    
    //移动所有权的方法
    /*fn greet(g1: String, g2: String) -> (String, String) {
        println!("{} {}!", g1, g2);
        (g1, g2)
    }*/
    
    //引用的方法
    fn greet(g1: &String, g2: &String){
        println!("{} {}!", g1, g2);
    }
    
    ```

- 解引用

    - 解引用指针以访问数据
    - 解引用运算符是*
    - 显示解引用
        - 使用解引用运算符*
        - 多层的引用需要多个*
    - 隐式解引用
        - 需要解引用的指针`变量.方法`可以自动解引用
        - 隐式转换支持多层
        - 亦可反向

- 别名和可变性不可同时存在

    - 别名：通过不同变量访问同一数据

    - 别名数据：可以被多个变量访问的一块数据

    - Box（有所有权的指针）：不能别名

        - 将一个Box变量赋给了另一个变量：移动了所有权
        - 即不能让多个Box同时拥有一块数据
        - 被所有的数据只能通过其所有者来访问，不能通过别名访问

    - 引用（无所有权的指针）：旨在临时创建别名

    - 变量对其数据又三种权限：

        - 读（R）：数据可以被复制到另一个位置
        - 写（W）：数据可以被修改
        - 拥有（O）：数据可以被移动或释放
        - 变量对其数据的默认是RO，如果声明的时候加上mut则会有W权限
        - 这些权限在运行时并不存在，仅在编译器内部存在
        - 引用可以临时移除这些权限

        - 权限是定义在位置上的（不仅仅是变量）
            - 位置是任何可以放在赋值语句左侧的东西
            - 位置包括：变量、位置的解引用、位置的数组访问、位置的字段访问以及上述的任何组合
            - 当位置变成不使用的时候，它们会失去权限，因为有些权限是互斥的
            - 例如：一个变量被引用，该变量就会失去对应权限（WO），否则引用该变量的变量就可能出现未定义

- 创建对这个数据的引用（即借用）会导致该数据变成临时不可修改，只能等该引用不在使用时才会恢复W权限

- 借用检查器能发现权限违规

    - Rust在借用检查器里使用RWO等权限

    - 借用检查器查找涉及引用的潜在不安全操作

- 可变引用提供对数据“唯一的”且“非拥有的”访问

    - 不可变引用（共享引用）：只读的
    - 可变引用（独占引用）：在不移动数据的情况下，临时提供可变访问，即不移动数据和所有权
        - 创建可变引用：使用&mut

- 权限在引用生命周期结束时被返回

- 数据必须在其所有的引用存在的期间存活

    - 即将被引用的O权限关闭

- 流动权限F

    - 在表达式使用函数的输入引用或返回函数输出引用时需要
    - F权限在函数体内不会发生变化
    - 如果一个引用允许在特定的表达式中使用（即流动），那么它就具有F权限

## 五、结构体

> 自定义类型，其字段可包含多种数据类型，需要给每个字段命名

```rust
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    let mut user2 = Users {
        email: "someone@example.com",
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
	let mut user3 = User {
        email: String::from("123@example.com"),
        ..user1
    };
    user1.email = String::from("123@qq.com");
    fn build_user(email: String, username: String) -> User {
        User {
            active: true,
            //username: username,
            //email: email,
            username,
            email,
            sign_in_count: 1,
        }
    }
}

struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
struct Users {
    active: bool,
    username: String,
    email: &'static str,
    sign_in_count: u64,
}
```

- 字段和值一样时可以合并只写字段名
- 如果两个变量实现了同一个结构体，并且其中有多个字段的值相同，另外一个可以只写不同的字段和值，然后加上`..user1`即可user1是另一个变量的名称
- 使用引用类型作为字段类型，需要设置生命周期

### 1. Tuple Struct

> 字段没有名

- ```rust
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);
    
    fn main() {
        let black = Color(0, 0, 0);
        let origin = Point(0, 0, 0);
    }
    ```

### 2. 无字段的Struct

- ```rust
    struct AlwaysEqual;
    
    fn main() {
        let subject = AlwaysEqual;
    }
    ```

### 3. 借用结构体的字段

- ```rust
    struct Point { x: i32, y: i32 }
    
    fn print_point(p: &Point) {
        println!("{}, {}", p.x, p.y);
    }
    
    fn main() {
        let mut p = Point { x: 0, y: 0 };
    
        let x = &mut p.x;
    
        print_point(&p);
        *x += 1;
    }
    ```

### 4. Derived Trait

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

//为struct添加方法
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    fn can_hold(&self, other: &Rectangle) -> bool {
    	self.width > other.width && self.height > other.height
	}
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let rec = Rectangle::square(3);
    
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

- 在struct里面是方法，其他地方是函数
- 方法的第一个参数永远是self（自己）（可以是引用、所有权、可变引用等）
- 使用struct的方法时，只需传递第二个及其后面的参数即可，因为第一个参数是self
- 第一个参数不是self的函数叫做关联函数

## 六、枚举

> 定义了一组可能的值

- ```rust
    fn main() {
        let four = IpAddrKind::V4;
        let six = IpAddrKind::V6;
    }
    
    enum IpAddrKind {
        V4, // Variants
        V6
    }
    ```

- 这里的four和six的类型都是IpAddrKind

- ```rust
    fn main() {
        enum IpAddrKind {
            V4(u8, u8, u8, u8),
            V6(String),
        }
    
        let home = IpAddrKind::V4(127, 0, 0, 1);
        let loopback = IpAddrKind::V6(String::from("::1"));
    }
    ```

- 枚举成员可以携带类型

### 1. Option Enum

> 来自标准库，表示某个值可能存在或者不存在

- ```rust
    enum Option<T> {
        None,
        Some(T),
    }
    ```

- 类型是Option<T>

- 表示出某可能不存在

- `Option<T>` 与 `T` 是不同类型

- 强迫你得处理这种情况

### 2. Match表达式

> 控制流

- ```rust
    #[derive(Debug)]
    enum UsState {
        Alabama,
        Alaska,
        // --snip--
    }
    
    enum Coin {
        Penny,
        Nickel,
        Dime,
        Quarter(UsState),
    }
    
    fn value_in_cents(coin: Coin) -> u8 {
        match coin {
            Coin::Penny => {
                println!("Lucky penny!");
                1
            },
            Coin::Nickel => 5,
            Coin::Dime => 10,
            Coin::Quarter(state) => 25{
                println!("State quarter from {:?}!", state);
    			25
            },
        }
    }
    
    fn main() {}
    ```

- 和其他语言的switch类似

- Coin::Penny就是模式匹配

- `=>`后面接的是表达式，其返回的值就是match返回的值

- match必须把所有可能情况列出

### 3. if let

> 匹配一种情况

```rust
fn main() {
    let config_max = Some(3u8);

    if let Some(max) = config_max {
        println!("The maximum number is {}", max);
    } else {
        println!("None");
    }
    // match config_max {
    //     Some(max) => println!("The maximum is configured to be {max}"),
    //     _ => (),
    // }
}
```

## 七、项目代码组织

### 1. creat

> creat是组织和共享代码的基本构建块

- binary creat：可执行的，需要有main函数
- library creat：没有main函数，无法执行，定义一些功能，可共享使用

### 2. creat root

> 编译creat的入口点（源代码文件）

- binary creat：src/main.rs
- library creat：src/lib.rs

### 3. package

> 由一个或多个creats组成

- 包含Cargo.toml文件（描述了如何构建这些creats）
- package规则
    - 可有多个binary creats
    - 最多只能有一个library creat
    - 但至少得有一个creat

### 4. module（模块）

> 将代码组织成更小、更易管理的单元的方法

- 使用mod声明

- 可有子模块

- 路径（path）

- public vs private

    - 所有的东西 (functions, methods, structs, enums, modules, and constants) 默认对父模块是 private（私有的）
    - 父模块中的项不能使用子模块中的私有项
    - **但子模块中的项可以使用其祖先模块中的项**（底部字幕补充：*哪怕它是私有的*）
    - 使用 `pub` 关键字让其变为 public
    - 相对路径可使用 `super`、`self` 关键字

- 引用（use）

    - > 有点像Java的import导包

    - 引用其他文件的方法等可以在本文件直接使用，use+路径

    - **function**：引用到父模块

    - **struct、enum...**：引用完整路径

    - 但如果引用到项目同名则使用时需要带上上一级

    - 在lib.rs中使用`pub use`导入的话，就会被视为在该文件下创建的了

- 会在声明mod的文件里、libaray creat里面、同名的rs文件里面寻找

- ```rust
    mod models;
    
    //inline, mod xxx {}
    //models.rs
    //models/mos.rs
    ```

- 绝对路径

    - 例子`creat::m1::m2:methon`
    - 其中m2和methon需要加put

- 相对路径

    - super表示上一级目录
    - self表示自己

- 访问library creat需要从包名开始

- Struct vs Enum

    - **Struct**：需为 struct 本身和各字段单独设置 `pub`
    - **Enum**：**只要 enum 本身是 pub 的，那么所有变体都是 pub 的**

### 5. as关键字

> 给引用起别名

- 可以用来区分同名的引用

### 6. 使用第三方creat

- 在Cargo.toml文件中的[dependencies]下面添加需要的第三方库`名称 = "版本"`
- 通命令行添加`cargo add 名称`

## 八、集合

### 1. Vectors

- 在单一数据结构存储多个值

- 在内存中连续存储（相邻）

- 元素必须是同类型

- 数据放在堆上

- Vec<T>来自标准库

- ```rust
    let v: Vec<i32> = Vec::new();//创建
    let v: = vec![1,|2, 3];//赋值（使用宏）
    ```

- ```rust
    fn main() {
        let mut v = Vec::new();
        
        v.push(5);//在集合内添加元素
        v.push(6);
        v.push(7);
    }
    ```

- 引用集合元素

    - 直接引用：`&v[索引]`；返回类型是引用类型，例如：`&i32`
        - 索引越界会panic恐慌

    - get方法引用：`v.get(索引)`;返回Option枚举类型，例如：`Option[&i32]`；
        - 可以使用match来处理可能的情况
        - 索引越界会返回None，不会恐慌，反而可以使用match处理该情况

- 遍历集合元素

    - ```rust
        fn main() {
            let v: Vec<i32> = vec![100, 32, 57];
            for n_ref in &v {
                let n_plus_one: i32 = *n_ref + 1;
                println!("n_plus_one: {n_plus_one}");
            }
        }
        ```

    - ```rust
        fn main() {
            let mut v: Vec<i32> = vec![100, 32, 57];
            for n_ref in &mut v {
                *n_ref += 200;//修改数组内元素
            }
        }
        ```

- 迭代器（Iterator）

    - 迭代器负责为序列中的每个项提供访问接口，并决定序列何时结束。Rust 中的迭代器是**惰性的（Lazy）**，不调用消费方法就不会产生开销。

    - 三大遍历方式（核心对比）

    - | 方法               | 产出项类型           | 集合所有权变化                  | 等价语法糖        | 适用场景       |
        | :----------------- | :------------------- | :------------------------------ | :---------------- | :------------- |
        | **`.iter()`**      | `&T`（不可变借用）   | 无变化，原集合仍可用            | `for x in &v`     | 只读访问元素   |
        | **`.iter_mut()`**  | `&mut T`（可变借用） | 无变化，原集合仍可用            | `for x in &mut v` | 就地修改元素   |
        | **`.into_iter()`** | `T`（转移所有权）    | **集合被消费/移动**，后续不可用 | `for x in v`      | 获取元素所有权 |

    - 基础语法与解引用示例

    - ```rust
        let mut v = vec![100, 32, 57];
        
        // 1. 只读借用 (.iter() / &v)
        for n_ref in &v {
            let next: i32 = *n_ref + 1; // 需解引用获取值
            println!("{next}");
        }
        
        // 2. 可变借用 (.iter_mut() / &mut v)
        for n_ref in &mut v {
            *n_ref += 200; // 解引用修改原内存中的值
        }
        
        // 3. 转移所有权 (.into_iter() / v)
        for val in v {
            println!("{val}"); // val 拥有所有权
        }
        // println!("{:?}", v); // 错误！v 已被移走
        ```

- Range (区间) 遍历

    - Range 本身实现了 `Iterator` 特征，可直接在 `for` 循环中生成连续数字序列，具备惰性求值与零成本抽象特性。

    - 核心语法对比

        | **语法格式**  | **数学区间表示** | **是否包含终点** | **示例** | **产出序列** |
        | ------------- | ---------------- | ---------------- | -------- | ------------ |
        | `start..end`  | `[start, end)`   | 否（左闭右开）   | `0..3`   | `0, 1, 2`    |
        | `start..=end` | `[start, end]`   | 是（全闭区间）   | `1..=3`  | `1, 2, 3`    |

    - 基础遍历与常用方法示例

        ```rust
        // 1. 左闭右开区间
        for i in 0..3 {
            println!("{i}"); // 输出: 0, 1, 2
        }
        
        // 2. 全闭区间
        for i in 1..=3 {
            println!("{i}"); // 输出: 1, 2, 3
        }
        
        // 3. 倒序遍历（需加括号并调用 .rev()）
        for i in (0..3).rev() {
            println!("{i}"); // 输出: 2, 1, 0
        }
        
        // 4. 指定步长 (.step_by())
        for i in (0..10).step_by(2) {
            println!("{i}"); // 输出: 0, 2, 4, 6, 8
        }
        ```

    - 配合集合索引遍历与对比

        ```rust
        let v = vec![10, 20, 30];
        
        // 方式一：通过 Range 索引遍历（需注意越界风险）
        for i in 0..v.len() {
            println!("索引 {i} 的值: {}", v[i]);
        }
        
        // 方式二：惯用安全写法（结合 .enumerate()，天然杜绝越界）
        for (i, val) in v.iter().enumerate() {
            println!("索引 {i} 的值: {val}");
        }
        ```

    - 易错陷阱

        - 逆序陷阱：直接写 `3..0` 不会报错，但属于长度为 0 的空迭代器，不会执行任何循环；反向计数必须写为 `(0..3).rev()`。
        - 越界恐慌：使用类似 `0..til` 配合 `v[i]` 访问时，若 `til > v.len()` 会直接引发 index out of bounds 的 panic 恐慌。

    - next()

        - range.next()返回的是索引

- 存储不同元素的方法：通过在vec集合元素为euem枚举类型，然后在枚举内部声明各个类型的元素实现

### 2. String

> 字节的集合，外加一些方法

- Rust的核心语言：str（&str）

- String来自标准库

- 可变、可增长、拥有所有权、UTF-8编码的字符串类型

- 创建String

    - ```rust
        fn main() {
        	let mut s = Sring::new();//直接创建
            
            let data = "initial content";//有初始数据的创建
            let s = data.to_string();//实现了Display Trait的类型，具有to_string()方法
            
            let s = "initial content".to_string;//和上面两行表示含义一致
            
            let s = String::from("initial content");//使用from函数创建
        }
        ```

- 更新String

    - ```rust
        fn main() {
            //String后面附加字符串
            let mut s = String::from("foo");
            s.push_str("bar");//s:foobar
            
            //String后面附加字符
            let mut s = String::from("lo");
            s.push('l');
        }
        ```

- 连接String

    - ```rust
        fn main() {
        	let s1 = String::from("Hello");
            let s2 = String::from("World");
            let s2 = s1 + ',' + &s2;//此时s2发生强转，由String转为str;使用的是add(self, S:&str) -> String{}方法
            
            //使用format!宏
            let s = format!("{s1},{s2}");
        }
        ```

- 对String的索引访问

    - Rust不允许通过索引访问String的元素
    - 原因
        - String中元素的字节数无法确定
        - 索引操作常被期望为常数时间(O(1))，但是Rust必须从头开始遍历内容直到指定索引，以确定有多少个有效字符，因此无法实现恒定时间复杂度的访问操作

- 对String进行切片

    - 如果要使用索引来创建String切片，必须更加具体，可使用[]配合Range
    - 必须保证切的位置在字符的边界，因为可能字符不止一个字节

- 遍历String

    - 获得单个Unicode标量值，使用`.chars()`方法

    - 获得原始字节，使用`.bytes()`方法

    - ```rust
        fn main() {
            //输出
            //a	
            //b
            for c in "ab".chars(){
                println!({c});
            }
            //输出
            //97
            //98
            for b in "ab".bytes(){
                println!({b});
            }
        }
        ```

    - 

### 3. HashMap<K, V>

- 存储键值对（K，V）映射的集合

- 通过K来查找

- 数据放在堆上

- Key的类型必须相同；Value的类型也必须相同

- 创建HashMap

    - ```rust
        fn main() {
            let mut scores = HashMap::new():
            scores.insert(String("Blue"), 10);//插入元素
            scores.insert(String("Yellow"), 50);
            
            let vec = vec![("key1", "value1"), ("key2", "value2")];
            let map: HashMap<_, _> = vec.into_iter().collect();//into_iter()方法将vec变成迭代器
            
            //访问HashMap的值，使用get方法
            let team_name = String::from("Blue");
            let score = scores.get(&team_name).copied().unwarp_or(0);//copied()方法的作用是解引用，因为get方法的返回值是Option<&T>，解引用为Option<T>;unwrap_or(0) 的作用是安全解包并提供保底默认值，将Option<T>解包为T，并保证安全
            
            //遍历K-V
            for (key, value) in scores {
                println!("{}: {}", key, value);
            }
        }
        ```

- HashMap和所有权

    - 对于实现Copy Trait的值，直接复制到map里

    - 对于具有所有权的类型的值，移动到map里

    - ```rust
        fn main() {
            
            //所有权
            let field_name = String::from("Favorite color");
            let field_value = String::from("Blue");
            
            let mut map = HashMap:new();
            map.insert(firld_name, field_value);
            
            //println!("{field_name}-{field_value}")
            //field_name和field_value的所有权已经移动，不能再调用
            
            //更新HashMap
            //Key 存在：替换还是保留value？新旧合并？
            //替换
            let mut scores = HashMap:new();
            scores.insert(String::from("Blue"), 10);
            scores.insert(String::from("Blue"), 25);
            println!("{scores:?}");//25
            //保留
            let mut scores = HashMap:new();
            scores.insert(String::from("Blue"), 10);
            
            scores.entry(String::from("Yellow")).or_insert(50);//map.entry(key)检测key在map中是否存在
            scores.entry(String::from("Blue")).or_insert(50);//Entry.or_insert(value)不存在则插入,并且返回对这个值的一个可变引用&mut Value
            println!("{scores:?}");//{"Blue":10, "Yellow": 50}
            
            //基于key原有的值更新（新旧合并）
            let text = "hello world wonderful world";//统计各个单词出现的次数
            let mut map = HashMap:new();
            for word in text.split_whitespace() {
                let count = map.entry(word).or_insert(0);
                *count += 1;
            }
            println!("{map:#?}");//HashMap打印K-V对出现的顺序是不一定的
        }
        ```

- Hashing函数

    - 默认使用SipHash，相对安全，并不是最快的
    - 可通过指定的hasher（实现了BuildHasher这个Trait）来切换Hashing函数

## 九、错误处理

> Rust没有异常
>
> 只有可恢复错误和不可恢复错误

### 1. 不可恢复错误Panic

- 什么是 panic 

    * 当代码遇到严重逻辑错误或无法继续安全运行的非法状态时触发。
    * 程序会打印错误信息，展开（Unwind）调用栈释放资源，并终止当前线程。

- 常见触发场景 

    - **主动触发**： 
        - 显式调用宏：`panic!("something went wrong");`   
        - 断言失败：`assert!(a == b);`   
        - 强行解包失败：`None.unwrap()` 或 `Err(e).expect("msg")` 
    - **被动触发（底层运行时检测）**：   
        - 索引越界**（最常见）：访问了超出长度的数组/Vector 下标，例如 `v[3]`。   **
        - **非法算术**：整数除以零（`x / 0`）。 

- Panic后的响应

    - 展开Stack，并清理数据

    - 立即终止（abort），使用这个二进制文件会变小，需要在Cargo.toml中配置panic = "abort"

- 如何避免索引越界 panic（以 Vector 为例） 

    - 直接下标索引 `v[i]`**：如果 `i` 越界，**会直接 panic 导致程序崩溃**。 **
    - **安全获取 `.get(i)`**：返回 `Option<&T>`，越界时返回 `None` 而不是 panic，可以通过 `match` 或 `if let` 安全处理。 

- ```rust
    let v = vec![1, 2, 3];
    
    // 危险：越界会 panic!
    // let x = v[99]; 
    
    // 安全：返回 Option<&i32>
    match v.get(99) {
        Some(val) => println!("值: {val}"),
        None => println!("索引越界，安全处理！"),
    }
    ```

- Backtrace

    - 到达某个点之前所调用的所有函数的列表
    - 需要设置环境变量
        - `RUST_BACKTRACE="不为0的任何值，必须是Debug模式"`


### 2. 可恢复错误

- 使用Result处理可恢复的错误

    - ```rust
        enum Result<T,E>{
            Ok(T),
            Err(E),
        }
        ```

### 3. 错误处理方式

- 出现错误时使程序产生恐慌（panic）的常用快捷方式
    - unwrap（）
        - 是Ok，用于提取Option或Result类型内部的值
        - 如果是None或Err，程序将panic并终止
    - expect（）
        - 与unwarp（）类似，允许你提供一个自定义的panic消息
- 传播错误
    - 将错误返回，由调用该函数的代码来决定如何处理错误
- ?运算符
    - 使用?运算符时
        - 如果操作成功，它会解包Ok并继续执行下一行代码
        - 如果操作失败，它会立即返回Err，并将错误传播给调用者
    - 使用?运算符可以避免大量的match或if let语句，是代码更简洁
    - 可以进行链式调用
    - from函数
        - 把值从一个类型转化为另一个类型
        - 定义在std的From trail上
    - 什么时候可以使用?
        - 函数的返回类型与?所作用的值类型兼容
        - ?可用于返回类型为Result、Option或者实现了FromResidual的类型的函数内
- main函数的返回类型也可以是Result<T,E>
    - main函数可以返回任何实现了std::process::Termination这个Trail类型
        - Termination定义了report函数，它返回ExitCode
- 何时使用panic
    - 不可恢复场景
    - 程序进入不可预期的bad state
    - 安全问题或代码无法继续执行
    - 违反函数契约或关键假设
- 何时使用Result
    - 可恢复的错误场景
    - 提供恢复选项
    - 预期可能发生的错误
    - 希望调用者决定如何处理错误
- 推荐使用panic的情况
    - 原型代码和示例
    - 测试代码
    - 安全性关键的输入验证
    - 调用外部不可控代码时的异常状态
- 推荐使用Result的情况
    - 处理可预期的错误
    - HTTP请求失败
    - 解析错误
    - 用户输入验证

## 十、泛型

> Rust中用于消除重复的工具之一
>
> 可以使用泛型数据类型来定义函数或struct

### 1. 在函数中使用泛型

- ```rust
    fn largest<T>(list: &[T])->&T{
        
    }
    ```

- 函数使用泛型时需要在函数名后面声明<T>

### 2. 在Struct中使用泛型

- ```rust
    struct Point<T>{
        x:T,
        y:T,
    }
    //该形式x和y的类型必须一致
    struct Point<T,U>{
        x:T,
        y:U,
    }
    //此时x和y的类型可以不一样
    ```

- 泛型参数太多，可能意味着需要重构

### 3. 在Enum中使用泛型

- ```rust
    enum Option<T>{
        Some(T),
        None,
    }
    ```

### 4. 在方法定义中使用泛型

- ```rust
    impl<T> Point<T> {
        fn x(&self) -> &T {
            &self.x
        }
    }
    ```

- 方法使用泛型时必须在impl后面声明<T>

- 不能同时为泛型类型和具体类型实现同一个名称的方法

- Rust没有继承机制

### 5. 使用泛型代码的性能

- 在Rust中使用泛型类型不会比使用具体类型让程序运行得更慢
- Rust通过单态化在编译时实现这种效率，单态化是编译器将泛型代码转换为具体代码的过程

## 十一、 组织方式

### 1.箱(Crate)

- 箱是二进制程序文件或者库文件，存在于包中
- 箱是树状结构，它的树根是编译器开始运行时编译的源文件所编译的程序
- "二进制程序文件"不一定是"二进制可执行文件"，只能确定是是包含目标机器语言的文件，文件格式随编译环境的不同而不同

### 2. 包(Package)

- 当我们使用 Cargo 执行 new 命令创建 Rust 工程时，工程目录下会建立一个 Cargo.toml 文件。工程的实质就是一个包，包必须由一个 Cargo.toml 文件来管理，该文件描述了包的基本信息以及依赖项。
- 一个包最多包含一个库"箱"，可以包含任意数量的二进制"箱"，但是至少包含一个"箱"（不管是库还是二进制"箱"）。
- 当使用 cargo new 命令创建完包之后，src 目录下会生成一个 main.rs 源文件，Cargo 默认这个文件为二进制箱的根，编译之后的二进制箱将与包名相同。

### 3. 模块(Module)

- 对于一个软件工程来说，我们往往按照所使用的编程语言的组织规范来进行组织，组织模块的主要结构往往是树。Java 组织功能模块的主要单位是类，而 JavaScript 组织模块的主要方式是 function。
- 这些先进的语言的组织单位可以层层包含，就像文件系统的目录结构一样。Rust 中的组织单位是模块（Module）。

## 十二、Trait

> 一个Trait定义了特定类型所具有的功能
>
> 可以使用Trait以一种抽象的方式来定义共享的行为
>
> 可以使用Trait Bounds来指定哪些类型才是我们想要的泛型类型（实现了某些特定行为的类型）

### 1. 定义Traut

- 类型的行为：可以在该类型上调用的方法

- Trait定义：将不同的方法签名组成一个方法签名，由此定义一套共享的行为

- ```rust
    pub trait Summary {
        fn summarize(&self) -> String;//没有方法体
    }
    ```

- r如果实现了该trait，就必须分别实现trait卡密的方法或者方法签名

### 2. 实现Trait

- ```rust
    impl Summary fir NewsArtucle {
        fn summarize(&self) -> Stirng{
            
        }
    }
    ```

### 3. Trait的实现规则

- 只要trait或类型其中之一属于当前crate，就可以实现该trait
- 一致性和孤儿规则：孤儿规则要求trait或类型必须属于当前crate，防止冲突出现；避免多个crate为同一类型实现相同的trait导致的歧义，保证代码稳定性
- 合法示例
    - 在本地类型Tweet上实现标准库的Display（trait）
    - 在标准库的Vec<T>上实现本地的Summary（trait）
- 非法示例
    - 不能在Vec<T>上实现Display（trait）；因为两者都来自标准库

### 4.  默认实现

- ```rust
    pub trait Summary{
        fn summarize(&self) -> String{
            String::from("(Read more...)")
        }
    }
    ```

- 可以保留也可以覆盖，类似Java的重写

### 5. Trait作为参数

- ```rust
    pub fn notify(item: &impl Summary) {
        println!("Breaking news! {}, item.summarize()");
    }
    ```

### 6. Trait Bound

- impl Trait语法适用于简单的情况，但它实际上是更长形式的语法糖，称为Trait Bound

- ```rust
    pub fn notify<T: Summary>(item: &T) {
        println!("Breaking news! {}", item.summarize())
    }
    ```

- Trait Bound适用于复杂情况

- ```rust
    pub fn notify(item1: &impl Summary, item2: &impl Summary){}
    
    pub fn notify<T: Summary>(item1: &T, item2: &T){}
    ```

- 使用+来指定多个Trait Bound

- ```rust
    pub fn notify(item1: &(impl Summary + Display)){}
    
    pub fn notify<T: Summary + Display>(item: &T){}
    ```

- 使用Where让Trait Bound更清晰

- 使用过多的Trait Bound会使函数签名难以阅读

- ```rust
    fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32{}
    
    fn some_dunction<T, U>(t: &T, u: &U) -> i32
    where
    	T: Display + Clone,
    	U: Clone + Debug,
    {}
    ```

### 7. 返回实现Trait的类型

- ```rust
    fn returns_sumarizeable() -> impl Summary {
        Tweet {
            username: String::from("hprse_ebooks"),
            content: String::from(
            	"of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
    ```

- impl Trait只能在返回单一类型时使用

### 8. 使用Trait Bound 来有条件地实现方法

- 所有的pair<T>都会实现new方法

- ```rust
    impl<T> pair<T> {
        fn new(x: T, y: T) -> Self {
            Self { x, y}
        }
    }
    ```

- blanket实现：可以为实现某个trait的类型有条件地实现另一个trait

- ```rust
    impl<T: Display> ToString for T {
        
    }
    
    let s = 3.to_stirng();
    ```

## 十三、生命周期

> 生命周期：确保引用在所需的时间内有效；引用的有效范围（作用域Scope）
>
> 每个引用都有生命周期
>
> 大多数时，生命周期都是隐式的，且可被推断出来
>
> 当引用的生命周期可能以几种不同的方式相关联时：
>
> ​	就必须标注生命周期了
>
> ​	泛型生命周期参数
>
> 是哦那个生命周期防止悬垂引用
>
> 悬垂引用会导致程序引用到它不应该引用的数据

### 1.  借用检查器

- 确保数据存活的时间长于其引用
- 比较作用域，以确定所有的借用是否有效

### 2. 函数中的泛型生命周期

- 为了确实参数和返回类型等引用之间的关系

### 3. 生命周期语法

- 生命周期注解不会改变引用存活的时间，而是描述多个引用之间的生命周期关系
- 生命周期参数：
    - 必须以`'`开头
    - 通常是小写
    - 并且非常短
- 位置：紧跟在引用&的后面，用空格与引用类型分开

### 4. 函数签名中的生命周期注解

- ```rust
    fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
        if x.len() > y.len() {
            x
        } else {
            y
        }
    }
    ```

- &'a str的生命周期是x、y中间生命周期短的那个

- **![image-20260910185327363](C:\Users\ZhuanZ\AppData\Roaming\Typora\typora-user-images\image-20260910185327363.png)**

- 报错原因是result的生命周期应该为string1和string2之间短的那个，而string1和string2都有可能是的，但是string2的作用域限制

- 函数返回类型的生命周期必须要与其中一个参数的生命周期匹配

- 如果函数的返回的引用不指向某个参数，就只能是指向在函数内部创建的本地变量，但这样会导致悬垂引用，因为函数走完之后本地变量释放

### 5. Struct定义中的生命周期注解

- 

### 6. 生命周期省略规则

- 无需程序员遵守

- 是一组编译器需要考虑的特殊情况，满足这些情况后，就不需要显示的写出生命周期了

- 如果Rust应用这套规则后，仍存在歧义，编译器就会报错

- 输入生命周期：在函数、方法的参数上的生命周期

- 输出生命周期：在返回值上的生命周期

- 三条规则：

    - 编译器为每个输入类型中的每个生命周期分配不同的生命周期参数

    - 如果只有一个输入生命周期参数，该生命周期将被分配给所有的输出生命周期参数
    - 如果有多个输入生命周期参数，但其中一个是&self或者&mut self（因为这是一个方法），那么self的生命周期会被分配给所有的输出生命周期参数

### 7. 方法定义中的生命周期注解

- ```rust
    impl<'a> ImportantExcerpt<'a> {
        fn level(&self) -> i32 {
            3 
        }
    }
    ```

- ![image-20260910195216637](C:\Users\ZhuanZ\AppData\Roaming\Typora\typora-user-images\image-20260910195216637.png)

### 8. 静态生命周期

- `'static`：它表示受影响的引用可以在整个程序的持续时间内存货，所有的字符串字面量都具有`'static`生命周期

- ```rust
    let s: &'static str = "I have a static lifetime."
    ```

### 泛型类型参数、Trait Bound和生命周期一起使用的例子

<img src="C:\Users\ZhuanZ\AppData\Roaming\Typora\typora-user-images\image-20260910195629746.png" alt="image-20260910195629746" style="zoom:33%;" />

## 十四、测试

> Tests(测试)就是函数
>
> ​	设置所需的数据或状态
>
> ​	运行需要被测试的代码
>
> ​	断言其结果是你想要的

### 1. 编写测试

- 最简单情况下：Tests就是带有`#[test]`的函数

- ```rust
    pub fn add(left: u64, right: u64) -> u64{ left + right }
    
    #[cfg(test)]
    mod tests{
        use super::*;//使用super调用函数之外的东西
        
        #[test]
        //#[should_panic]//发生panic测试就会通过，不发生panic测试不通过
        //#[should_panic(expected = "匹配的内容")]//panic里面的内容和expected内容匹配则通过，反正失败
        fn it_work() {
            let result = add(2, 2);
            assert_eq!(result, 4);
        }
    }
    ```

- `assert!`宏

    - `assert!`确保某个条件在测试中为true
    - 它接收一个Boolean作为参数
    - 如果值为true，测试通过，不会发生任何事情
    - 如果值为false，则触发panic!，导致测试失败
    - 使用`assert!`宏可以帮助我们检查代码是否按预期运行
    - 第二个参数就是断言失败显示的信息（可选）

- 测试相等性

    - `assert_eq!`相等
    - `assert_ne!`不等
    - 比`assert!`方便，断言失败时会打印出比较的两个值
    - 实际上，他们就是使用==和!=
    - 被比较的值需实现PartialEq和Debug

### 2. 执行测试

- 测试命令的参数
    - 先列出cargo test的参数	cargo test -- help
    - 然后跟着--                             cargo test -- --help
    - 再列出Test binary（测试二进制文件）的参数
- 测试的默认运行模式
    - 使用线程并行运行
    - 必须确保
        - 各个测试不互相依赖
        - 不依赖于共享的状态（包括环境、目录、环境变量等）
    - 单线程运行
        - `$ cargo test -- --test-threads=1`
- 测试成功时打印出具体内容
    - `cargo test -- --show-output`
- 根据函数名进行测试
    - `cargo test <测试的函数名(支持模糊匹配但不需要*和_)>`
- 忽略函数测试
    - 在需要忽略的函数上加`#[igonre]`
    - `cargo test -- --igonred`只测试被忽略的函数
    - `cargo test -- --include_igonred`忽略的和没有忽略的都会进行测试

### 3. 组织测试

- 单元测试：小而集中的测试，每次只测试一个模块，并且可以测试私有接口
    - 用于隔离测试代码单元，快速定位问题
    - 测试代码通常于被测代码放在同一个文件中，并放在`#[cfg(test)]`注解的tests模块中
    - `#[cfg(test)]`确保测试代码仅在运行cargo test时编译和运行，而不会影响普通的构建（cargo build）流程
    - 单元测试于代码在同一文件中，需要使用`#[cfg(test)]`注解避免它们被编译进最终结果
- 集成测试：完全外部的测试，它们以其他外部代码的方式使用你的代码，仅使用公共接口，并且可能在一个测试中覆盖多个模块
    - 集成测试在不同目录中，不需要`#[cfg(test)]`
    - 集成测试独立于库
    - 只能调用公共API
    - 用于验证库各部分的协作性
    - 测试覆盖很重要
    - 需在项目中创建tests目录来编写集成测试
    - `cargo test --test <集成测试文件名（不需要后缀）>`测试某个集成测试
    - 辅助文件（里面没有测试函数），可以放在tests/common/下，就不会测试该文件，例如tests/common/mod.rs