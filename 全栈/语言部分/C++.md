# C++ 知识体系总结

## 引言

```cpp
/* 对于 C++ 的一部分知识储备，在算法竞赛中可见一斑，但是算法竞赛与项目实践是有分歧的
对于项目而言接下来的知识就是类和对象，而算法竞赛的要求是对于 STL 的学习以及题量的丰富性和思路的灵活性 */
```

## 基础语法与特性

### 头文件与命名空间

```cpp
// C++ 内一般不要用 C 的头文件，否则可能会造成命名冲突
// 隐式类型转换发生在编译阶段

// 命名空间：namespace
namespace bit {
    // 可以放变量、结构体、函数等
    int a;
}

// 访问语法：域作用限定符 ::
cout << ::a;        // 全局域
cout << bit::a;     // 命名空间域
cout << a;          // 当前域
```

**域的分类：**
- 局部域
- 全局域  
- 命名空间域
- 类域

**注意：**
- namespace 只能定义在全局，不允许定义在局部
- 多文件编程中的同名 namespace 会自动合并
- C++ 的所有操作都放在 std 命名空间内

### 引用与别名

```cpp
int a = 10;
int &b = a;  // 对 a 取别名，操作空间相同且地址相同

// 特性：
// - 一旦初始化就不能修改
// - 必须初始化
// - 传引用可直接修改原值
```

**常量引用：**
```cpp
// int &ref = 10;  // 错误
const int &ref = 10;  // 合法，创建临时变量
```

## 函数进阶

### 默认参数与占位参数

```cpp
int test(int a, int b = 10, int c = 20) {
    return a + b + c;
}

// 使用：
test(20);      // 输出 50
test(20, 30);  // 输出 70

// 占位参数：
void test(int) {
    cout << "12345677890";
}
// 使用：test(10);
```

### 函数重载

**条件：**
- 同一作用域
- 函数形参表不同（个数、顺序）
- 函数名相同

```cpp
void func(int &a) { }
void func(const int &a) { }

int a = 10;
func(10);    // 调用 func(const int &a)
func(a);     // 调用 func(int &a)
```

## 面向对象编程

### 类与封装

```cpp
class Student {
private:     // 私有权限：类内可访问，类外不可访问
    string m_name;
    int m_num;

public:      // 公共权限：类内类外均可访问
    void setName(string name) {
        m_name = name;
    }
    
    void showName() {
        cout << m_name << endl;
    }
};

// 使用：
Student s1;
s1.setName("zhangsan");
s1.showName();
```

**权限分类：**
- `public`：类内类外均可访问
- `protected`：类内可访问，类外不可访问，子类可访问父类
- `private`：类内可访问，类外不可访问，子类不可访问父类

### 构造函数与析构函数

```cpp
class Person {
public:
    // 构造函数
    Person() {
        cout << "构造函数";
    }
    
    // 拷贝构造函数
    Person(const Person& p) {
        cout << "拷贝构造函数";
    }
    
    // 析构函数
    ~Person() {
        cout << "析构函数";
    }
};
```

**构造函数调用方式：**

1. **括号法：**
```cpp
Person p1(10);    // 有参构造
Person p2();      // 无参构造（注意：不要加空括号）
Person p3(p1);    // 拷贝构造
```

2. **显式法：**
```cpp
Person p1;
Person p2 = Person(10);  // 有参构造
Person p3 = Person(p1);  // 拷贝构造
```

3. **隐式转换：**
```cpp
Person p1 = 10;  // 等价于 Person p1 = Person(10)
```

### 深拷贝与浅拷贝

```cpp
class Person {
public:
    int m_age;
    int* height;
    
    // 深拷贝构造函数
    Person(const Person& p) {
        m_age = p.m_age;
        height = new int(*p.height);  // 深拷贝：开辟新空间
    }
    
    ~Person() {
        if (height != NULL) {
            delete height;
            height = NULL;
        }
    }
};
```

### 初始化列表

```cpp
class Person {
private:
    int m_a, m_b, m_c;

public:
    // 传统初始化
    Person(int a, int b, int c) {
        m_a = a;
        m_b = b;
        m_c = c;
    }
    
    // 初始化列表
    Person(int a, int b, int c) : m_a(a), m_b(b), m_c(c) {
    }
};
```

### 静态成员

```cpp
class Person {
public:
    static int m_a;  // 类内声明
    
    static void func() {
        m_a = 100;   // 只能访问静态变量
        cout << "调用了静态函数";
    }
};

// 类外初始化
int Person::m_a = 100;

// 使用：
Person p;
cout << p.m_a;           // 通过对象访问
cout << Person::m_a;     // 通过类名访问
Person::func();          // 通过类名调用静态函数
```

### this 指针

```cpp
class Person {
public:
    int age;
    
    Person(int age) {
        this->age = age;  // 区分成员变量和形参
    }
    
    Person& addAge(Person& p) {
        this->age += p.age;
        return *this;     // 返回对象本身，支持链式调用
    }
};

// 链式编程
p2.addAge(p1).addAge(p1).addAge(p1);
```

### 常函数与常对象

```cpp
class Person {
public:
    mutable int m_a;  // 可在常函数中修改
    
    void showPerson() const {  // 常函数
        // m_a = 100;  // 错误：常函数不能修改成员变量
        m_a = 100;     // 正确：mutable 修饰的变量可以修改
    }
};

const Person p;        // 常对象
p.m_a = 100;          // 正确：mutable 变量可修改
```

### 友元

```cpp
class Home {
    friend void goodGay(Home* home);  // 友元声明
    
private:
    string m_bedroom;

public:
    string m_livingroom;
};

void goodGay(Home* home) {
    cout << home->m_livingroom;
    cout << home->m_bedroom;  // 可以访问私有成员
}
```

## 运算符重载

### 加号运算符重载

```cpp
class Person {
public:
    int m_a, m_b;
    
    // 成员函数重载
    Person operator+(Person& p) {
        Person temp;
        temp.m_a = this->m_a + p.m_a;
        temp.m_b = this->m_b + p.m_b;
        return temp;
    }
};

// 全局函数重载
Person operator+(Person& p1, Person& p2) {
    Person temp;
    temp.m_a = p1.m_a + p2.m_a;
    temp.m_b = p1.m_b + p2.m_b;
    return temp;
}

// 使用：
Person p3 = p1 + p2;
```

## 继承

```cpp
class Base {
public:
    void header() { cout << "标题(公共头)" << endl; }
    void footer() { cout << "页尾(公共底)" << endl; }
    void left()   { cout << "左侧(公共左)" << endl; }
};

class Java : public Base {  // 公有继承
public:
    void content() { cout << "Java 内容" << endl; }
};
```

**继承方式：**
- 公有继承：原来是什么就是什么
- 保护继承：原来的一切变为保护  
- 私有继承：原来的一切变为子类私有

**构造顺序：** 先构造父类，再构造子类  
**析构顺序：** 先析构子类，后析构父类

### 多继承与菱形继承

```cpp
class Animal { 
public: 
    int m_age; 
};

class Sheep : virtual public Animal {};  // 虚继承
class Tuo : virtual public Animal {};    // 虚继承  
class SheepTuo : public Sheep, public Tuo {};  // 多继承

// 使用：
SheepTuo st;
st.m_age = 18;  // 虚继承避免了二义性
```

## 多态

```cpp
class Animal {
public:
    virtual void speak() {  // 虚函数
        cout << "动物在说话" << endl;
    }
    
    virtual void eat() = 0;  // 纯虚函数，抽象类
};

class Cat : public Animal {
public:
    void speak() {  // 重写虚函数
        cout << "小猫在说话" << endl;
    }
    
    void eat() {    // 必须重写纯虚函数
        cout << "小猫在吃鱼" << endl;
    }
};

// 使用：
Animal* animal = new Cat();
animal->speak();  // 动态多态：调用子类函数
```

**虚析构：**
```cpp
class Animal {
public:
    virtual ~Animal() {  // 虚析构
        cout << "Animal 析构" << endl;
    }
};
```

## 模板编程

### 函数模板

```cpp
template<typename T>
void mySwap(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}

// 使用：
int a = 9, b = 0;
mySwap(a, b);           // 自动类型推导
mySwap<int>(a, b);      // 显示指定类型
```

### 类模板

```cpp
template<class T1, class T2>
class Person {
public:
    Person(T1 name, T2 age) {
        this->m_name = name;
        this->m_age = age;
    }
    
    void showPerson() {
        cout << "姓名：" << m_name << " 年龄：" << m_age << endl;
    }

private:
    T1 m_name;
    T2 m_age;
};

// 使用：
Person<string, int> p("孙悟空", 100);
```

**类模板与继承：**
```cpp
template<class T>
class Base {
    T m;
};

// 子类指定父类类型
class Son : public Base<int> {};

// 灵活指定父类类型
template<class T1, class T2>
class Son2 : public Base<T2> {
    T1 obj;
};
```

## STL 与函数对象

### 仿函数

```cpp
class MyAdd {
public:
    int operator()(int v1, int v2) {
        return v1 + v2;
    }
};

// 使用：
MyAdd myadd;
cout << myadd(10, 10) << endl;  // 仿函数调用
```

### 谓词

```cpp
class GreaterFive {
public:
    bool operator()(int val) {  // 一元谓词
        return val > 5;
    }
};

// 使用：
vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9};
auto it = find_if(v.begin(), v.end(), GreaterFive());
```

## 名称修饰与混合编程

### C++ 调用 C

**C 语言文件 (c_lib.c):**
```c
#include <stdio.h>
void c_function() {
    printf("This is a C function");
}
```

**C 头文件 (c_lib.h):**
```c
#ifndef C_LIB_H
#define C_LIB_H

#ifdef __cplusplus
extern "C" {
#endif

void c_function();

#ifdef __cplusplus
}
#endif

#endif
```

**C++ 代码:**
```cpp
#include "c_lib.h"
int main() {
    c_function();  // 正常调用 C 函数
    return 0;
}
```

## 现代 C++ 特性

### Lambda 表达式

```cpp
auto isLessThanLimit = [limit](auto val) -> bool {
    return val < limit;
};

// 语法：[捕获列表](参数列表) mutable(可选) -> 返回类型 { 函数体 }
```

**捕获方式：**
- `[]`：不捕获任何变量
- `[=]`：值捕获所有变量  
- `[&]`：引用捕获所有变量
- `[x]`：值捕获 x
- `[&x]`：引用捕获 x
- `[=, &x]`：值捕获所有，但 x 引用捕获

### 结构化绑定

```cpp
std::map<int, std::string> myMap = {
    {1, "one"}, {2, "two"}, {3, "three"}
};

for (const auto& [key, value] : myMap) {
    std::cout << "Key: " << key << ", Value: " << value << std::endl;
}
```

## STL 算法概览

```cpp
// 常用算法：
for_each        // 遍历容器
transform       // 搬运容器
find / find_if  // 查找元素
sort            // 排序
count           // 统计元素
reverse         // 反转
copy            // 拷贝
replace         // 替换
accumulate      // 累加
set_intersection // 交集
set_union       // 并集
set_difference  // 差集
```

## 编程范式与设计原则

### 接口隔离原则
客户端不应该依赖不需要的接口，应该始终依赖最小的接口。

### 依赖倒置原则
高层模块不应该依赖低层模块，二者都应该依赖抽象。

## 总结

C++ 语言特性丰富，从基础语法到面向对象，从模板元编程到现代 C++ 特性，形成了一个完整的编程体系。算法竞赛更注重 STL 的使用和算法思维，而项目开发更注重面向对象设计、模块化和工程化实践。