在 C++ 中，实现赋值运算符（`operator=`）最经典、最健壮的设计模式就是 **Copy-and-Swap 惯用法（Idiom）**。



它通过**复用已有的拷贝构造函数、移动构造函数与析构函数**，既消除了重复代码，又天然提供了**强异常安全保证（Strong Exception Safety Guarantee）**。



## 1. 传统 `operator=` 的缺陷

回顾传统的手写深拷贝赋值方式：



C++

```
DynamicArray& operator=(const DynamicArray& other) {
    if (this != &other) {
        delete[] data_;             // 1. 释放原有内存
        size_ = other.size_;
        data_ = new int[other.size_]; // 2. 申请新内存（可能抛出 std::bad_alloc 异常！）
        std::memcpy(data_, other.data_, size_ * sizeof(int));
    }
    return *this;
}
```

这段代码存在两个严重设计问题：



1. **异常安全性崩溃（Basic Exception Safety 都不满足）**：

    如果 `new int[other.size_]` 在堆内存耗尽时抛出 `std::bad_alloc` 异常，此时旧的 `data_` **已经被释放**。当前对象内部的指针变成野指针，状态遭到破坏，无法回滚到操作前的状态。

2. **代码重复（DRY 原则破坏）**：

    释放内存的逻辑在**析构函数**和 `operator=` 中各写了一遍；分配内存和复制数据的逻辑在**拷贝构造函数**和 `operator=` 中又各写了一遍。

## 2. Copy-and-Swap 的实现步骤

Copy-and-Swap 的核心思路是：



1. **Copy**：在操作前，先通过传值或拷贝构造复制出一份临时副本。若此时内存不足抛异常，原对象**完全不受任何影响**。
2. **Swap**：用不抛异常的 `swap` 操作，把当前对象的资源与临时副本交换。
3. **Destruction**：离开作用域时，临时副本销毁，顺带把当前对象的旧资源安全释放。

### 第一步：提供一个友元（Friend）非抛异常的 `swap` 函数

在类内部提供一个基于 ADL（参数依赖查找）的 `swap` 函数，只交换基本成员变量和指针：



C++

```
#include <utility>
#include <algorithm>

class DynamicArray {
private:
    int* data_ = nullptr;
    size_t size_ = 0;

public:
    // ... 构造函数、拷贝构造、析构函数已实现 ...

    // 友元 swap 函数：仅交换指针和标量，绝不抛出异常 (noexcept)
    friend void swap(DynamicArray& first, DynamicArray& second) noexcept {
        using std::swap;
        swap(first.data_, second.data_);
        swap(first.size_, second.size_);
    }
};
```

### 第二步：编写统一的按值传参 `operator=`

借助 **按值传参（Pass-by-Value）**，我们可以将**拷贝赋值**与**移动赋值**合二为一：



C++

```
class DynamicArray {
    // ... 前置成员与 swap ...

    // 单一 operator=，形参 other 是按值传递（按副本传入）
    DynamicArray& operator=(DynamicArray other) noexcept {
        // 交换当前对象与临时副本 other 的内容
        swap(*this, other);
        return *this;
    }
};
```

## 3. 运行过程推演

### 场景 A：作为拷贝赋值（接收左值）

C++

```
DynamicArray a(100);
DynamicArray b(200);

a = b; // b 是左值
```

1. **构造形参**：`b` 是左值，调用 **拷贝构造函数**，在栈上生成形参 `other`（深拷贝）。
    - *如果此时 `new` 抛出异常，赋值操作立即终止，`a` 和 `b` 毫发无损。*
2. **执行交换**：`swap(*this, other)` 将 `a` 原先的内存指针与 `other` 的指针互换。
3. **离开函数**：局部变量 `other`（此时持有 `a` 的旧内存）超出作用域，自动调用析构函数，干净地释放掉 `a` 原有的旧内存。

### 场景 B：作为移动赋值（接收右值）

C++

```
DynamicArray a(100);

a = DynamicArray(500); // 传入临时对象（右值）
// 或者 a = std::move(b);
```

1. **构造形参**：实参是右值，调用 **移动构造函数** 构造形参 `other`（$O(1)$ 指针窃取）。
2. **执行交换**：`swap(*this, other)`，`a` 拿到了新资源。
3. **离开函数**：`other` 析构，清理掉 `a` 原有的旧资源。
4. **效果**：一并完成了移动赋值，无需单独重载 `operator=(DynamicArray&&)`。

## 4. Copy-and-Swap 的核心优势

| **维度**         | **传统手写 operator=**                            | **Copy-and-Swap 惯用法**                                     |
| ---------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| **自我赋值检测** | 必须显式写 `if (this != &other)` 防止自身破坏     | **天然免疫**自我赋值（交换自身内容毫无副作用）               |
| **异常安全性**   | 需手写繁琐的 try-catch 或暂存指针，极易写出野指针 | **强异常安全保证**（All-or-Nothing，要么全成功，要么状态回滚） |
| **代码复用度**   | 与拷贝构造、析构函数充斥重复代码                  | 100% 复用已有构造函数和析构函数，代码极其紧凑                |
| **统一维护性**   | 需分别编写拷贝赋值与移动赋值两套函数              | 仅需一个按值传递的 `operator=` 即可同时满足两者              |

## 5. 什么时候不该使用 Copy-and-Swap？

虽然 Copy-and-Swap 是工业界编写资源管理类的首选惯用法，但在**极端追求绝对吞吐量的超高性能场景**下，存在一个细微的代价：



- **无法复用已有容量（Capacity Reuse）**：

    以 `std::vector` 为例：若 `vector A` 的容量是 1000，当前只存了 10 个元素，此时把一个包含 20 个元素的 `vector B` 赋值给 `A`。

    - 传统的 `vector` 赋值会直接复用 `A` 已经开辟的堆空间，**不发生任何内存分配**。
    - Copy-and-Swap 会强制先深拷贝构造出一个新的临时对象（发生一次堆分配），再销毁旧空间，多出了一次分配与释放开销。

> **工程准则**：
>
> 凡是资源对象（如 `std::unique_ptr` 包装器、文件句柄、Socket 管理类、没有 capacity 概念的动态数组），**Copy-and-Swap 是最佳实践**；仅在需要重用已分配内存池缓冲区的特定容器场景下，才考虑分离手写优化。