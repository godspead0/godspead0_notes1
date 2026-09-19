# **主题**：C 语言基础语法全面解析及经典实战案例展示，包括变量与数据类型、输入输出函数、运算符与表达式、控制语句、数组、字符串、指针、函数、预处理指令等内容，以及素数判断、二分查找、冒泡排序等基础算法，猜数字游戏、闰年判断、汉诺塔等实用程序，还有字符串拷贝、大小写转换等字符串处理和菱形打印等图形打印案例

# C语言基础语法与实战案例大全

## 一、基础语法核心知识点

### 1. 变量与数据类型

#### 基本数据类型

| 类型           | 描述                  | 打印格式符 | 占用空间（32位系统） |
| -------------- | --------------------- | ---------- | -------------------- |
| `char`         | 字符型（ASCII码存储） | `%c`       | 1字节                |
| `short`        | 短整型                | `%d`       | 2字节                |
| `int`          | 整型                  | `%d`       | 4字节                |
| `long`         | 长整型                | `%ld`      | 4字节                |
| `long long`    | 超长整型              | `%lld`     | 8字节                |
| `float`        | 单精度浮点型          | `%f`       | 4字节                |
| `double`       | 双精度浮点型          | `%lf`      | 8字节                |
| `unsigned int` | 无符号整型            | `%u`       | 4字节                |

#### 关键规则

- 变量必须初始化（如 `int num = 0;`），避免垃圾值
- `const` 修饰变量变为常量，不可修改（`const int MAX = 100;`）
- 自动类型转换：低精度→高精度（`char/short→int→long→float→double`）
- 浮点型默认是 `double`，`float` 需加后缀 `f`（`float pi = 3.14f;`）

### 2. 输入输出函数

#### printf 格式化输出

| 格式符 | 功能                      | 示例                    | 输出结果   |
| ------ | ------------------------- | ----------------------- | ---------- |
| `%d`   | 输出整型                  | `printf("%d", 10);`     | `10`       |
| `%f`   | 输出浮点型（默认6位小数） | `printf("%.2f", 3.14);` | `3.14`     |
| `%c`   | 输出字符                  | `printf("%c", 97);`     | `a`        |
| `%s`   | 输出字符串                | `printf("%s", "abc");`  | `abc`      |
| `%u`   | 输出无符号整型            | `printf("%u", 100);`    | `100`      |
| `%08d` | 不足8位补0                | `printf("%08d", 123);`  | `00000123` |

#### scanf 输入函数

- 基本用法：`scanf("%d %c", &num, &ch);`（变量需加 `&` 取地址）
- 忽略分隔符：`scanf("%d%*c%d", &a, &b);`（`%*c` 忽略中间字符如 `-`）
- 返回值：成功读取的变量个数（多组输入：`while(scanf("%d", &n) == 1)`）
- 字符串输入：`scanf("%s", str);`（自动忽略空格，遇空格终止）

### 3. 运算符与表达式

#### 算术运算符

- 基础运算：`+` `-` `*` `/` `%`（`%` 仅用于整数，结果符号与被除数一致）
- 自增自减：
    - `i++`：先使用后自增
    - `++i`：先自增后使用
- 复合赋值：`+=` `-=` `*=` `/=`（如 `a += 3` 等价于 `a = a + 3`）

#### 逻辑运算符

- `&&`（与）、`||`（或）、`!`（非）
- 短路求值：`a && b` 中若 `a` 为假，`b` 不再计算；`a || b` 中若 `a` 为真，`b` 不再计算

#### 位运算符

- `&`（按位与）、`|`（按位或）、`^`（按位异或）、`~`（按位取反）
- 移位运算：`<<`（左移）、`>>`（右移）（左移等价于乘2，右移等价于除2）

### 4. 控制语句

#### 分支语句

- if-else：
    - ```C
        if (condition1) {
            // 代码块1
        } else if (condition2) {
            // 代码块2
        } else {
            // 代码块3
        }
        ```
- switch-case：
    - ```C
        switch (expression) { // expression 必须是整型
            case 1:
                // 代码
                break; // 跳出switch，否则穿透
            case 2:
                // 代码
                break;
            default: // 所有case不匹配时执行
                // 代码
        }
        ```

#### 循环语句

- while：先判断后执行
    - ```C
        while (condition) {
            // 循环体
        }
        ```
- do-while：先执行后判断（至少执行一次）
    - ```C
        do {
            // 循环体
        } while (condition);
        ```
- for：结构紧凑，适合已知循环次数
    - ```C
        for (初始化; 条件; 更新) {
            // 循环体
        }
        ```
- 循环控制：
    - `break`：跳出当前循环
    - `continue`：跳过本次循环剩余部分，进入下一次循环

### 5. 数组

#### 一维数组

- 定义：`int arr[10];`（大小可省略，如 `int arr[] = {1,2,3};`）
- 初始化：未初始化元素默认为0（`int arr[5] = {1,2};` 其余为0）
- 访问：`arr[i]`（索引从0开始）
- 数组长度计算：`int sz = sizeof(arr) / sizeof(arr[0]);`

#### 二维数组

- 定义：`int arr[3][5];`（行可省略，列不可省略）
- 初始化方式：
    - ```C
        int arr1[3][5] = {1,2,3,4,5,6}; // 按行填充，剩余为0
        int arr2[3][5] = {{1,2}, {3,4}}; // 按行初始化，剩余为0
        ```
- 存储方式：行优先连续存储

### 6. 字符串

#### 字符串表示

- 字符数组：`char str[] = "abc";`（自动添加 `\0` 作为结束标志）
- 字符指针：`char *str = "abc";`（指向字符串常量）

#### 核心函数（需包含 `<string.h>`）

| 函数名   | 功能                      | 示例                        |
| -------- | ------------------------- | --------------------------- |
| `strlen` | 求字符串长度（不含 `\0`） | `strlen("abc")` → 3         |
| `strcpy` | 字符串复制                | `strcpy(dst, src)`          |
| `strcmp` | 字符串比较（按ASCII码）   | `strcmp(a,b)`：0相等，>0a大 |
| `strcat` | 字符串拼接                | `strcat(dst, src)`          |

#### 关键注意

- `strlen` 返回无符号整型，避免直接参与减法运算（如 `strlen("a") - strlen("aa")` 结果为正数）
- 字符串复制必须确保目标数组足够大，且手动添加 `\0`

### 7. 指针

#### 基本概念

- 指针变量存储变量的地址：`int *p = #`（`p` 存储 `num` 的地址，`*p` 访问 `num` 的值）
- 数组名是数组首元素地址：`arr == &arr[0]`
- 指针运算：
    - `p++`：指针向后移动一个元素大小
    - `*p`：解引用，访问指针指向的值

#### 指针应用

- 函数传参：通过指针修改实参的值
    - ```C
        void swap(int *a, int *b) {
            int temp = *a;
            *a = *b;
            *b = temp;
        }
        ```
- 字符串操作：通过指针遍历字符串
    - ```C
        char *str = "abc";
        while (*str != '\0') {
            printf("%c", *str++);
        }
        ```
- 数组访问：`arr[i] == *(arr + i)`

#### const 与指针

- `const int *p`：指针指向的值不可修改（`*p = 10` 错误）
- `int *const p`：指针本身不可修改（`p = &num` 错误）

### 8. 函数

#### 函数定义与调用

- 定义格式：
    - ```C
        返回值类型 函数名(参数列表) {
            // 函数体
            return 返回值;
        }
        ```
- 声明：若函数定义在调用之后，需提前声明（`int add(int a, int b);`）
- 递归：函数调用自身（如斐波那契数列、汉诺塔）

#### 函数传参

- 值传递：形参是实参的拷贝，修改形参不影响实参
- 地址传递：通过指针/数组名传递，修改形参影响实参

### 9. 预处理指令

- 宏定义：`#define MAX 100`（无分号，直接替换）
- 条件编译：`#ifdef DEBUG ... #endif`
- 头文件包含：
    - 系统头文件：`#include <stdio.h>`
    - 自定义头文件：`#include "myfile.h"`

## 二、经典实战案例

### 1. 基础算法

#### 素数判断（优化版）

```C
#include <stdio.h>
#include <math.h>

int is_prime(int n) {
    if (n <= 1) return 0;
    for (int i = 2; i <= sqrt(n); i++) {
        if (n % i == 0) return 0;
    }
    return 1;
}

int main() {
    for (int i = 100; i <= 200; i++) {
        if (is_prime(i)) printf("%d\n", i);
    }
    return 0;
}
```

#### 二分查找（有序数组）

```C
#include <stdio.h>

int binary_search(int arr[], int sz, int target) {
    int left = 0, right = sz - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2; // 避免溢出
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1; // 未找到
}

int main() {
    int arr[] = {1,2,3,4,5,6,7,8,9};
    int sz = sizeof(arr)/sizeof(arr[0]);
    int idx = binary_search(arr, sz, 5);
    printf("索引：%d\n", idx); // 输出4
    return 0;
}
```

#### 冒泡排序

```C
#include <stdio.h>

void bubble_sort(int arr[], int sz) {
    for (int i = 0; i < sz-1; i++) {
        int flag = 1; // 优化：无交换则提前退出
        for (int j = 0; j < sz-1-i; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
                flag = 0;
            }
        }
        if (flag) break;
    }
}

int main() {
    int arr[] = {3,1,4,2,5};
    int sz = sizeof(arr)/sizeof(arr[0]);
    bubble_sort(arr, sz);
    for (int i = 0; i < sz; i++) {
        printf("%d ", arr[i]); // 输出1 2 3 4 5
    }
    return 0;
}
```

### 2. 实用程序

#### 猜数字游戏

```C
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <windows.h>

void menu() {
    printf("-----------------\n");
    printf("-----0. 开始游戏-----\n");
    printf("-----1. 退出游戏-----\n");
    printf("-----------------\n");
}

void game() {
    int secret = rand() % 100 + 1; // 1-100随机数
    int guess, count = 5;
    printf("猜数字游戏开始！剩余5次机会\n");
    while (count > 0) {
        printf("请猜数字：");
        scanf("%d", &guess);
        if (guess < secret) printf("猜小了\n");
        else if (guess > secret) printf("猜大了\n");
        else {
            printf("恭喜猜对！答案是%d\n", secret);
            return;
        }
        count--;
        printf("剩余%d次机会\n", count);
    }
    printf("游戏结束！答案是%d\n", secret);
}

int main() {
    srand((unsigned int)time(NULL)); // 随机数种子
    int choice;
    do {
        menu();
        printf("请选择：");
        scanf("%d", &choice);
        switch (choice) {
            case 0: game(); break;
            case 1: printf("退出游戏\n"); break;
            default: printf("输入错误\n");
        }
    } while (choice != 1);
    return 0;
}
```

#### 闰年判断

```C
#include <stdio.h>

int is_leap_year(int year) {
    // 能被4整除且不能被100整除，或能被400整除
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
}

int main() {
    int year;
    printf("请输入年份：");
    scanf("%d", &year);
    if (is_leap_year(year)) printf("%d是闰年\n", year);
    else printf("%d不是闰年\n", year);
    return 0;
}
```

#### 汉诺塔（递归）

```C
#include <stdio.h>

// 将n个盘子从a借助b移到c
void hanoi(int n, char a, char b, char c) {
    if (n == 1) {
        printf("%c->%c\n", a, c);
        return;
    }
    hanoi(n-1, a, c, b); // n-1个从a借助c移到b
    printf("%c->%c\n", a, c); // 第n个从a移到c
    hanoi(n-1, b, a, c); // n-1个从b借助a移到c
}

int main() {
    int n;
    printf("请输入盘子数量：");
    scanf("%d", &n);
    hanoi(n, 'A', 'B', 'C');
    return 0;
}
```

### 3. 字符串处理

#### 字符串拷贝（手动实现strcpy）

```C
#include <stdio.h>

void my_strcpy(char *dst, const char *src) {
    while (*src != '\0') {
        *dst++ = *src++;
    }
    *dst = '\0'; // 手动添加结束标志
}

int main() {
    char src[] = "hello world";
    char dst[20];
    my_strcpy(dst, src);
    printf("%s\n", dst); // 输出hello world
    return 0;
}
```

#### 字符串大小写转换

```C
#include <stdio.h>
#include <string.h>

void to_upper(char *str) {
    int len = strlen(str);
    for (int i = 0; i < len; i++) {
        if (str[i] >= 'a' && str[i] <= 'z') {
            str[i] -= 32; // ASCII码差值
        }
    }
}

int main() {
    char str[] = "Hello World!";
    to_upper(str);
    printf("%s\n", str); // 输出HELLO WORLD!
    return 0;
}
```

### 4. 图形打印

#### 菱形打印

```C
#include <stdio.h>

int main() {
    int n;
    printf("请输入菱形边长：");
    scanf("%d", &n);
    // 上半部分
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n-i; j++) printf(" ");
        for (int j = 1; j <= 2*i-1; j++) printf("*");
        printf("\n");
    }
    // 下半部分
    for (int i = n-1; i >= 1; i--) {
        for (int j = 1; j <= n-i; j++) printf(" ");
        for (int j = 1; j <= 2*i-1; j++) printf("*");
        printf("\n");
    }
    return 0;
}
```

#### 矩形边框

```C
#include <stdio.h>

int main() {
    int row, col;
    char ch;
    printf("请输入行数、列数、字符：");
    scanf("%d %d %c", &row, &col, &ch);
    for (int i = 0; i < row; i++) {
        for (int j = 0; j < col; j++) {
            // 边框打印字符，内部打印空格
            if (i == 0 || i == row-1 || j == 0 || j == col-1) {
                printf("%c", ch);
            } else {
                printf(" ");
            }
        }
        printf("\n");
    }
    return 0;
}
```

## 三、常见问题与注意事项

### 1. 编译警告与错误

- `_CRT_SECURE_NO_WARNINGS`：解决 `scanf` 安全警告
- `C4996`：忽略函数返回值警告（如 `scanf`）
- `C4244`：类型转换精度丢失（如 `double` 转 `int`）

### 2. 内存相关问题

- 数组越界：访问 `arr[sz]` 会导致内存溢出
- 野指针：未初始化的指针（`int *p; *p = 10;` 错误）
- 字符串未加 `\0`：导致 `strlen` 计算错误

### 3. 调试技巧

- `Ctrl+F5`：直接运行程序
- `F11`：逐语句调试
- `assert` 断言：调试阶段检查条件（`assert(p != NULL);`）

### 4. 代码规范

- 变量命名：见名知意（`int student_count` 而非 `int a`）
- 缩进：统一使用4个空格
- 注释：关键逻辑添加注释，函数添加功能说明
- 空格：运算符前后、逗号后加空格（`a = b + c;` 而非 `a=b+c;`）

## 四、拓展知识

### 1. 递归与迭代

- 递归优点：代码简洁（如汉诺塔、斐波那契）
- 递归缺点：栈溢出风险、效率低
- 迭代替代：将递归转换为循环（如斐波那契迭代版）

### 2. 多文件编程

- 头文件（`.h`）：函数声明、宏定义、结构体定义
- 源文件（`.c`）：函数实现
- 编译：多个 `.c` 文件一起编译（`gcc main.c func.c -o program`）

### 3. 静态库与动态库

- 静态库（`.lib`/`.a`）：编译时链接，体积大，无需依赖
- 动态库（`.dll`/`.so`）：运行时链接，体积小，需依赖库文件

### 4. 位运算应用

- 二进制转换：`n & 1` 判断奇偶，`n >> 1` 右移等价于除2
- 标志位操作：用一个整数的不同位表示不同状态（如 `flag |= 1 << 3` 设置第3位）

通过以上内容，可系统掌握C语言基础语法与核心技能，结合实战案例巩固应用，适用于入门学习与面试备考。如需深入学习指针进阶、结构体、文件操作等内容，可在此基础上进一步拓展。