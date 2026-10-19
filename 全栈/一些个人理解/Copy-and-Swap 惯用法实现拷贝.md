以下是一个工业级标准的完整可编译示例。代码实现了一个动态数组类 `DynamicArray`，严格遵循 **RAII**、**Rule of Five**（五法则）、**智能指针语义** 以及 **Copy-and-Swap** 惯用法，同时包含测试用例验证每个阶段的调用过程：



C++

```
#include <iostream>
#include <utility>    // for std::swap
#include <algorithm>  // for std::copy
#include <cstddef>    // for std::size_t

class DynamicArray {
private:
    int* data_ = nullptr;
    std::size_t size_ = 0;

public:
    // -------------------------------------------------------------
    // 1. 基础构造函数 (RAII：获取资源即初始化)
    // -------------------------------------------------------------
    explicit DynamicArray(std::size_t size = 0)
        : data_(size > 0 ? new int[size]{} : nullptr), size_(size) {
        std::cout << "[Constructor] Allocated array of size " << size_ << "\n";
    }

    // 带初始值的构造函数
    DynamicArray(std::size_t size, int init_value)
        : data_(size > 0 ? new int[size] : nullptr), size_(size) {
        if (data_) {
            std::fill(data_, data_ + size_, init_value);
        }
        std::cout << "[Constructor] Allocated array of size " << size_ 
                  << " filled with " << init_value << "\n";
    }

    // -------------------------------------------------------------
    // 2. 析构函数 (RAII：生命周期结束自动释放资源)
    // -------------------------------------------------------------
    ~DynamicArray() noexcept {
        std::cout << "[Destructor] Deleting array of size " << size_ << "\n";
        delete[] data_; // delete[] nullptr 在 C++ 中是良定义且绝对安全的
    }

    // -------------------------------------------------------------
    // 3. 拷贝构造函数 (深拷贝：O(N))
    // -------------------------------------------------------------
    DynamicArray(const DynamicArray& other)
        : data_(other.size_ > 0 ? new int[other.size_] : nullptr), 
          size_(other.size_) {
        std::cout << "[Copy Constructor] Deep copying array of size " << size_ << "\n";
        if (data_) {
            std::copy(other.data_, other.data_ + size_, data_);
        }
    }

    // -------------------------------------------------------------
    // 4. 移动构造函数 (资源窃取：O(1)，绝不抛出异常)
    // -------------------------------------------------------------
    DynamicArray(DynamicArray&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        std::cout << "[Move Constructor] Moving array of size " << size_ << "\n";
        // 关键：剥夺原对象所有权，将其置于安全的可析构状态
        other.data_ = nullptr;
        other.size_ = 0;
    }

    // -------------------------------------------------------------
    // 友元 swap 函数 (ADL 友好，Copy-and-Swap 的核心基石)
    // -------------------------------------------------------------
    friend void swap(DynamicArray& first, DynamicArray& second) noexcept {
        using std::swap;
        swap(first.data_, second.data_);
        swap(first.size_, second.size_);
    }

    // -------------------------------------------------------------
    // 5. 赋值运算符 (Copy-and-Swap 惯用法：统一拷贝与移动赋值)
    // -------------------------------------------------------------
    // 注意：形参 other 是【按值传递】(Pass-by-Value)
    // - 传入左值时：调用拷贝构造函数生成 other（深拷贝）
    // - 传入右值时：调用移动构造函数生成 other（指针窃取）
    DynamicArray& operator=(DynamicArray other) noexcept {
        std::cout << "[operator=] Swapping with temporary/passed-by-value object\n";
        swap(*this, other);
        // 离开函数时，other 带着当前对象的旧内存析构，自动完成释放
        return *this;
    }

    // -------------------------------------------------------------
    // 容器基本接口与元素访问
    // -------------------------------------------------------------
    [[nodiscard]] std::size_t size() const noexcept { return size_; }
    [[nodiscard]] int* data() noexcept { return data_; }
    [[nodiscard]] const int* data() const noexcept { return data_; }

    int& operator[](std::size_t index) noexcept { return data_[index]; }
    const int& operator[](std::size_t index) const noexcept { return data_[index]; }
};

// -------------------------------------------------------------
// 测试验证代码
// -------------------------------------------------------------
int main() {
    std::cout << "--- 1. 测试构造与 RAII ---\n";
    {
        DynamicArray a(3, 42); // 构造 a
    } // 离开作用域，a 自动析构

    std::cout << "\n--- 2. 测试拷贝赋值 (深拷贝 + Swap) ---\n";
    {
        DynamicArray a(2, 10);
        DynamicArray b(4, 20);
        std::cout << ">>> 开始执行 a = b (左值赋值) <<<\n";
        a = b; 
        std::cout << ">>> a = b 执行完毕，当前 a 的大小: " << a.size() << " <<<\n";
    }

    std::cout << "\n--- 3. 测试移动赋值 (移动构造 + Swap) ---\n";
    {
        DynamicArray a(2, 10);
        std::cout << ">>> 开始执行 a = DynamicArray(5, 99) (右值赋值) <<<\n";
        a = DynamicArray(5, 99); 
        std::cout << ">>> 移动赋值执行完毕，当前 a 的大小: " << a.size() << " <<<\n";
    }

    return 0;
}
```

### 设计亮点剖析

1. **Rule of Five 精简为四**：

    依靠 Copy-and-Swap 惯用法，拷贝赋值（Copy Assignment）与移动赋值（Move Assignment）被整合进了同一个按值传参的 `operator=(DynamicArray other)` 中，代码减少了近一半，且杜绝了重复造轮子。

2. **强异常安全保证（Strong Exception Safety）**：

    所有可能因内存不足抛出 `std::bad_alloc` 的步骤，均发生在形参 `other` 构造期间。如果抛出异常，赋值动作根本不会发生，目标对象保持原样。而后续的 `swap` 与析构均为 `noexcept`。

3. **彻底杜绝自我赋值 Bug**：

    即使写出 `a = a;`，形参 `other` 也是 `a` 的独立深拷贝，交换后内存状态依然完整且正确，不需要任何 `if (this != &other)` 分支判断。