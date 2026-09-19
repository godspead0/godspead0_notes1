# 主题：JavaScript 中 JavaScript 基础架构、书写位置、基本语法规则、数据类型与字面量、变量声明与作用域、类型转换、运算符、数组、对象等核心知识点全解析

# JavaScript 与 TypeScript 核心知识点全解析

## 一、JavaScript 基础架构

JavaScript 由 **ECMAScript**（核心语法）和 **Web API**（浏览器提供的扩展功能）两部分组成：

- **ECMAScript**：定义变量、数据类型、语法规则、函数等核心特性
- **Web API**：分为两类
    - DOM（文档对象模型）：操作 HTML 页面元素
    - BOM（浏览器对象模型）：操作浏览器窗口、历史记录、本地存储等

## 二、JavaScript 书写位置

与 CSS 类似，JS 有三种书写位置，优先级：行内 > 内部 > 外部（后加载覆盖先加载）

### 1. 行内式

直接写在 HTML 标签的事件属性中，仅适用于简单事件触发，不推荐大面积使用：

```HTML
<p onclick="alert('Hello World')">点击触发</p>
<button onmouseover="console.log('鼠标经过')">悬浮触发</button>
```

### 2. 内部式

写在 HTML 文件的 `<body>` 标签底部（避免阻塞页面渲染），用 `<script>` 标签包裹：

```HTML
<body>
  <!-- 页面内容 -->
  <script>
    // 内部 JS 代码
    alert('内部脚本执行');
    console.log('调试信息');
  </script>
</body>
```

### 3. 外部式

将代码写在独立的 `.js` 文件中，通过 `<script src="路径">` 引入：

```HTML
<!-- HTML 文件中引入 -->
<body>
  <script src="index.js"></script> <!-- 引入外部 JS 文件 -->
  <script>
    // 注意：外部引用模式下，此标签内的代码会被忽略
  </script>
</body>
```

- 引入路径支持相对路径（`./js/index.js`）和绝对路径（`https://xxx.com/index.js`）
- 多个外部脚本按引入顺序执行

## 三、JS 基本语法规则

### 1. 结束符

- 分号（`;`）可加可不加，建议统一风格（推荐添加，避免自动插入分号导致的 Bug）
- 换行符会被解析为隐式结束符，但复杂语句（如对象、数组多行定义）建议保留分号

### 2. 输出与输入语法

| 功能       | 语法                           | 说明                                             |
| ---------- | ------------------------------ | ------------------------------------------------ |
| 页面输出   | `document.write('内容')`       | 支持 HTML 标签解析（如 `<strong>文本</strong>`） |
| 警示框输出 | `alert('提示内容')`            | 阻塞页面渲染，优先执行                           |
| 控制台输出 | `console.log('调试信息')`      | 开发调试专用，不影响页面                         |
| 输入框     | `prompt('提示文字', '默认值')` | 返回用户输入的字符串，取消返回 `null`            |

### 3. 执行顺序原则

- 浏览器按 HTML 文档流顺序执行代码
- `alert()` 和 `prompt()` 会跳过页面渲染，优先弹出（阻塞后续代码执行）
- 外部脚本加载和执行会阻塞后续 HTML 解析（可通过 `defer`/`async` 属性优化）

## 四、数据类型与字面量

### 1. 字面量（Literal）

直接在代码中定义的值，是数据的原始形态：

- 数字字面量：`123`、`3.14`、`-45`、`NaN`、`Infinity`
- 字符串字面量：`'abc'`、`"123"`、``模板字符串``
- 数组字面量：`[1, 2, 3]`
- 对象字面量：`{ name: '张三', age: 18 }`
- 布尔字面量：`true`、`false`

### 2. 数据类型分类

JavaScript 是弱类型语言，变量类型可动态改变，核心数据类型如下：

| 类型      | 说明                               | 示例                                    |
| --------- | ---------------------------------- | --------------------------------------- |
| Number    | 数字（整数、浮点数、特殊值）       | `10`、`3.14`、`NaN`、`Infinity`         |
| String    | 字符串（单引号/双引号/反引号包裹） | `'hello'`、`"world"`、``name: ${name}`` |
| Boolean   | 布尔值（逻辑判断）                 | `true`、`false`                         |
| Undefined | 变量声明未赋值（默认值）           | `let a; console.log(a); // undefined`   |
| Null      | 空值（主动表示"无"）               | `let b = null;`                         |
| Object    | 复杂类型（对象、数组、函数等）     | `{}`、`[]`、`function() {}`             |
| Function  | 函数类型（可执行代码块）           | `function add(a, b) { return a + b }`   |

#### 特殊说明：

- `NaN`（Not a Number）：非数字类型，特点：`NaN !== NaN`（需用 `isNaN()` 判断）
- `null` 与 `undefined` 区别：`null` 是主动赋值的空，`undefined` 是未赋值的默认空
- 函数本质是特殊的对象（可调用的对象）

### 3. 类型判断（typeof）

用于检测变量的数据类型，支持两种语法：

```JavaScript
// 关键字形式
typeof 123; // "number"
typeof 'abc'; // "string"
typeof true; // "boolean"
typeof undefined; // "undefined"
typeof null; // "object"（历史 Bug，需特殊判断）
typeof {}; // "object"
typeof []; // "object"（数组本质是对象）
typeof function() {}; // "function"

// 函数形式
typeof(123); // 效果同上
```

## 五、变量声明与作用域

### 1. 变量声明方式

| 声明方式 | 作用域          | 重复声明 | 变量提升 | 示例               |
| -------- | --------------- | -------- | -------- | ------------------ |
| let      | 块级作用域      | 不允许   | 不提升   | `let a = 10;`      |
| const    | 块级作用域      | 不允许   | 不提升   | `const PI = 3.14;` |
| var      | 函数/全局作用域 | 允许     | 提升     | `var b = 20;`      |

#### 关键说明：

- `const` 声明的变量不可重新赋值，但复杂类型（对象、数组）的属性可修改（地址未变）：
    - ```JavaScript
        const arr = [1, 2, 3];
        arr.push(4); // 允许（地址不变）
        arr = [1, 2, 3, 4]; // 报错（重新赋值改变地址）
        ```
- 变量提升：`var` 声明的变量会在代码执行前被提升到当前作用域顶部，但值为 `undefined`：
    - ```JavaScript
        console.log(x); // undefined（变量提升）
        var x = 10;
        ```

### 2. 作用域

作用域决定变量的可访问范围，JS 有三种作用域：

- **全局作用域**：整个程序可访问（`var` 声明的变量、未声明直接赋值的变量）
- **函数作用域**：仅函数内部可访问（`var` 声明的变量）
- **块级作用域**：`{}` 内部（`let`/`const` 声明的变量，如 `if`、`for` 循环块）

#### 作用域链：

变量查找遵循"就近原则"：先在当前作用域查找，找不到则向上级作用域查找，直到全局作用域，形成作用域链。

#### 注意点：

- 函数内部未声明直接赋值的变量，自动成为全局变量（不推荐）：
    - ```JavaScript
        function fn() {
          num = 10; // 全局变量（无 let/const/var 声明）
        }
        fn();
        console.log(num); // 10
        ```

## 六、类型转换

JS 中类型转换分为**隐式转换**（自动触发）和**显式转换**（手动触发）

### 1. 隐式转换

由运算符或上下文自动触发，规则如下：

- **加号（+）**：一方为字符串则全部转为字符串（拼接）：
    - ```JavaScript
        1 + '2'; // "12"（数字转字符串）
        true + 'abc'; // "trueabc"（布尔转字符串）
        ```
- **其他运算符（-、\*、/、>、< 等）**：自动转为数字类型：
    - ```JavaScript
        1 - '2'; // -1（字符串转数字）
        '3' * '4'; // 12（字符串转数字）
        'abc' - 1; // NaN（无法转数字）
        ```
- **快速转数字技巧**：
    - ```JavaScript
        +'123'; // 123（字符串转数字）
        '456' - 0; // 456（字符串转数字）
        ```

### 2. 显式转换

手动调用转换函数，常用方法：

| 转换目标 | 方法                                           | 示例                                                |
| -------- | ---------------------------------------------- | --------------------------------------------------- |
| 数字     | `Number(值)`、`parseInt(值)`、`parseFloat(值)` | `Number('123') // 123`、`parseInt('123abc') // 123` |
| 字符串   | `String(值)`、`值.toString()`                  | `String(123) // "123"`、`true.toString() // "true"` |
| 布尔     | `Boolean(值)`                                  | `Boolean(0) // false`、`Boolean('abc') // true`     |

#### 关键说明：

- `parseInt()`：从字符串开头提取整数，遇到非数字停止（支持进制转换）
- `parseFloat()`：支持小数提取
- `Boolean()` 转换规则：以下值转为 `false`，其余为 `true`：

```
0`、`undefined`、`null`、`false`、`NaN`、`''`（空字符串）、`-0`、`0n
```

## 七、运算符

JS 支持多种运算符，重点补充常用特殊运算符：

### 1. 比较运算符

- `==`：松散相等（自动转换类型后比较）
- `===`：严格相等（不转换类型，值和类型都相同才成立）
- `!=`/`!==`：对应不相等判断

```JavaScript
2 == '2'; // true（类型转换后相等）
2 === '2'; // false（类型不同）
null == undefined; // true（特殊规则）
null === undefined; // false（类型不同）
```

### 2. 逻辑运算符

- `&&`：逻辑与（短路特性：左边为 `false` 则不执行右边）
- `||`：逻辑或（短路特性：左边为 `true` 则不执行右边）
- `!`：逻辑非（转为布尔值后取反）

#### 逻辑中断应用：

```JavaScript
// 给变量设置默认值（左边为 falsy 则用右边值）
let name = '' || '未知用户'; // "未知用户"
let age = 0 || 18; // 18（0 是 falsy）

// 逻辑与实现条件执行
let isLogin = true;
isLogin && console.log('已登录'); // "已登录"
```

### 3. 其他常用运算符

- **展开运算符（...）**：展开数组/对象：
    - ```JavaScript
        const arr1 = [1, 2, 3];
        const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]
        
        const obj1 = { name: '张三' };
        const obj2 = { ...obj1, age: 18 }; // { name: '张三', age: 18 }
        ```
- **可选链运算符（?.）**：避免对象属性不存在报错：
    - ```JavaScript
        const obj = { name: '张三' };
        console.log(obj.address?.city); // undefined（不报错）
        ```
- **空值合并运算符（??）**：仅当左边为 `null`/`undefined` 时用右边值：
    - ```JavaScript
        let num = 0 ?? 10; // 0（0 不是 null/undefined）
        let name = null ?? '未知'; // "未知"
        ```

## 八、数组（Array）

数组是有序的集合，支持增删改查等操作，核心用法如下：

### 1. 数组定义

```JavaScript
// 字面量方式（推荐）
const arr = [1, 2, 3, 'abc', true];

// 构造函数方式
const arr2 = new Array(1, 2, 3); // [1, 2, 3]
const arr3 = new Array(5); // 长度为 5 的空数组（注意：单参数是长度）
```

### 2. 核心属性与方法

| 功能         | 语法                                       | 示例                                                     |
| ------------ | ------------------------------------------ | -------------------------------------------------------- |
| 长度         | `arr.length`                               | `[1,2,3].length // 3`                                    |
| 末尾添加     | `arr.push(元素)`                           | `arr.push(4) // 返回新长度 4`                            |
| 末尾删除     | `arr.pop()`                                | `arr.pop() // 返回删除的元素`                            |
| 开头添加     | `arr.unshift(元素)`                        | `arr.unshift(0) // 返回新长度 4`                         |
| 开头删除     | `arr.shift()`                              | `arr.shift() // 返回删除的元素`                          |
| 中间操作     | `arr.splice(起始索引, 删除个数, 新增元素)` | `arr.splice(1, 2, 'a') // 删除索引1-2，插入'a'`          |
| 复制元素     | `arr.slice(起始索引, 结束索引)`            | `[1,2,3].slice(1,3) // [2,3]`（左闭右开）                |
| 合并数组     | `arr.concat(数组2, 数组3)`                 | `[1,2].concat([3,4]) // [1,2,3,4]`                       |
| 反转数组     | `arr.reverse()`                            | `[1,2,3].reverse() // [3,2,1]`                           |
| 排序         | `arr.sort(比较函数)`                       | `[3,1,2].sort((a,b) => a - b) // [1,2,3]`（升序）        |
| 数组转字符串 | `arr.join(分隔符)`                         | `[1,2,3].join('-') // "1-2-3"`                           |
| 查找索引     | `arr.indexOf(元素)`                        | `[1,2,3].indexOf(2) // 1`（未找到返回-1）                |
| 遍历映射     | `arr.map(回调函数)`                        | `[1,2,3].map(x => x*2) // [2,4,6]`                       |
| 筛选         | `arr.filter(回调函数)`                     | `[1,2,3].filter(x => x>1) // [2,3]`                      |
| 遍历         | `arr.forEach(回调函数)`                    | `arr.forEach((item, index) => console.log(item, index))` |

### 3. 数组遍历方式

```JavaScript
const arr = [1, 2, 3];

// 1. for 循环
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// 2. forEach（无返回值）
arr.forEach((item, index) => {
  console.log(item, index);
});

// 3. for...of（ES6+，支持 break）
for (let item of arr) {
  console.log(item);
  if (item === 2) break;
}

// 4. map（有返回值，生成新数组）
const newArr = arr.map(item => item * 2);
```

## 九、对象（Object）

对象是键值对的集合，用于存储复杂数据，核心用法如下：

### 1. 对象定义

```JavaScript
// 字面量方式（推荐）
const person = {
  name: '张三', // 属性（键值对）
  age: 18,
  gender: '男',
  // 方法（函数属性）
  sayHello: function() {
    console.log(`你好，我是${this.name}`);
  },
  // 简写方法（ES6+）
  eat() {
    console.log('吃饭了');
  }
};

// 构造函数方式
const obj = new Object();
obj.name = '李四';
obj.age = 20;
```

### 2. 对象属性访问

```JavaScript
const person = { name: '张三', age: 18 };

// 1. 点语法（推荐，简洁）
console.log(person.name); // "张三"
person.age = 19; // 修改属性

// 2. 方括号语法（支持变量或特殊字符键名）
console.log(person['name']); // "张三"
const key = 'age';
console.log(person[key]); // 19（变量键名）

// 3. 新增属性
person.gender = '男';

// 4. 删除属性（很少用）
delete person.gender;
```

### 3. 对象遍历

```JavaScript
const person = { name: '张三', age: 18, gender: '男' };

// for...in 循环（遍历属性名）
for (let key in person) {
  console.log(key); // 属性名：name, age, gender
  console.log(person[key]); // 属性值：张三, 18, 男
}

// Object 静态方法（ES6+）
Object.keys(person); // ["name", "age", "gender"]（获取所有键）
Object.values(person); // ["张三", 18, "男"]（获取所有值）
Object.entries(person); // [["name","张三"], ["age",18], ["gender","男"]]（获取键值对）
```

### 4. 自定义属性（data-*）

HTML 标签中可自定义 `data-*` 属性，通过 JS 读取：

```HTML
<div class="box" data-name="李四" data-age="20"></div>

<script>
  const box = document.querySelector('.box');
  console.log(box.dataset.name); // "李四"（读取 data-name）
  console.log(box.dataset.age); // "20"（读取 data-age）
</script>
```

## 十、函数（Function）

函数是可重复执行的代码块，JS 中函数是一等公民（可作为参数、返回值）

### 1. 函数定义方式

```JavaScript
// 1. 具名函数声明
function add(a = 0, b = 0) { // 默认参数
  return a + b; // 返回值
}

// 2. 匿名函数表达式
const subtract = function(a, b) {
  return a - b;
};

// 3. 箭头函数（ES6+）
const multiply = (a, b) => a * b; // 单句返回可省略 {} 和 return
const divide = (a, b) => { // 多句需加 {} 和 return
  if (b === 0) throw new Error('除数不能为0');
  return a / b;
};

// 4. 立即执行函数（IIFE，防止变量污染）
(function() {
  console.log('立即执行');
})();

(function(a, b) {
  console.log(a + b);
})(1, 2); // 传递参数
```

### 2. 函数参数

- **动态参数（arguments）**：伪数组，存储所有传入参数（仅普通函数支持）：
    - ```JavaScript
        function sum() {
          let total = 0;
          for (let i = 0; i < arguments.length; i++) {
            total += arguments[i];
          }
          return total;
        }
        sum(1, 2, 3); // 6
        ```
- **剩余参数（...args）**：ES6+ 特性，将剩余参数转为真数组（推荐）：
    - ```JavaScript
        function sum(...args) {
          return args.reduce((a, b) => a + b, 0);
        }
        sum(1, 2, 3, 4); // 10
        ```

### 3. 函数返回值

- 无 `return` 时返回 `undefined`
- 可返回任意类型，包括函数（闭包）、对象、数组等
- 返回多个值可通过数组或对象：
    - ```JavaScript
        function getInfo() {
          return [18, '张三']; // 数组返回
          // return { age: 18, name: '张三' }; // 对象返回（推荐，语义更清晰）
        }
        const [age, name] = getInfo(); // 解构赋值
        ```

### 4. 箭头函数与普通函数区别

| 特性              | 普通函数               | 箭头函数                          |
| ----------------- | ---------------------- | --------------------------------- |
| `this` 指向       | 指向调用者（动态绑定） | 指向外层最近的 `this`（静态绑定） |
| `arguments` 对象  | 支持                   | 不支持（需用剩余参数 `...args`）  |
| 构造函数用法      | 支持（可 `new` 调用）  | 不支持（不能 `new`）              |
| `prototype` 属性  | 有                     | 无                                |
| 函数体内 `return` | 多句需显式写 `return`  | 单句可省略 `return` 和 `{}`       |

## 十一、DOM 操作（文档对象模型）

DOM 是浏览器将 HTML 解析为的树形结构，JS 通过 DOM API 操作页面元素

### 1. 获取 DOM 元素

根据 CSS 选择器获取，核心方法：

| 方法                             | 说明                                 | 示例                                      |
| -------------------------------- | ------------------------------------ | ----------------------------------------- |
| `querySelector('选择器')`        | 获取第一个匹配的元素（返回单个元素） | `document.querySelector('.box')`          |
| `querySelectorAll('选择器')`     | 获取所有匹配的元素（返回伪数组）     | `document.querySelectorAll('ul li')`      |
| `getElementById('id名')`         | 根据 ID 获取元素（高效）             | `document.getElementById('title')`        |
| `getElementsByClassName('类名')` | 根据类名获取元素（返回伪数组）       | `document.getElementsByClassName('item')` |
| `getElementsByTagName('标签名')` | 根据标签名获取元素（返回伪数组）     | `document.getElementsByTagName('div')`    |

#### 关键说明：

- `querySelectorAll` 返回的是**伪数组**（NodeList），不能直接调用数组方法（需转数组：`Array.from(伪数组)`）
- 伪数组可通过 `for` 循环或 `forEach`（部分浏览器支持）遍历

### 2. 操作元素内容

| 方法/属性   | 说明                                 | 示例                                                      |
| ----------- | ------------------------------------ | --------------------------------------------------------- |
| `innerText` | 操作文本内容（不解析 HTML 标签）     | `box.innerText = '<strong>文本</strong>' // 显示原始标签` |
| `innerHTML` | 操作 HTML 内容（解析标签）           | `box.innerHTML = '<strong>文本</strong>' // 显示加粗文本` |
| `value`     | 操作表单元素值（input、textarea 等） | `input.value = '默认值'`                                  |

### 3. 操作元素样式

#### （1）行内样式操作（`style` 属性）

```JavaScript
const box = document.querySelector('.box');
box.style.width = '200px'; // 注意：值需加单位，驼峰命名（backgroundColor）
box.style.backgroundColor = 'red';
box.style.fontSize = '16px';
```

#### （2）类名操作（`classList`）

推荐方式（不覆盖原有类名）：

```JavaScript
const box = document.querySelector('.box');
box.classList.add('active'); // 新增类名
box.classList.remove('active'); // 删除类名
box.classList.toggle('active'); // 切换类名（有则删，无则加）
box.classList.contains('active'); // 判断是否有该类名（返回布尔值）
box.classList.replace('old', 'new'); // 替换类名
```

#### （3）直接修改 `className`（覆盖原有类名）

```JavaScript
box.className = 'box active'; // 保留原有类名需手动拼接
```

### 4. 操作元素属性

#### （1）普通属性（src、href、alt 等）

```JavaScript
const img = document.querySelector('img');
img.src = 'new.jpg'; // 修改图片路径
img.alt = '示例图片'; // 修改alt属性
console.log(img.src); // 获取属性值
```

#### （2）表单属性（disabled、checked、selected 等）

```JavaScript
const btn = document.querySelector('button');
const checkbox = document.querySelector('input[type="checkbox"]');

btn.disabled = true; // 禁用按钮
checkbox.checked = true; // 勾选复选框
```

### 5. 节点操作（增删改查）

#### （1）创建节点

```JavaScript
// 创建元素节点
const div = document.createElement('div');
div.className = 'new-box';
div.innerText = '新创建的元素';

// 创建文本节点（较少用）
const text = document.createTextNode('文本内容');
```

#### （2）添加节点

```JavaScript
const parent = document.querySelector('.parent');
const child = document.createElement('div');

// 追加到父节点末尾
parent.appendChild(child);

// 插入到指定节点之前
const referenceNode = document.querySelector('.reference');
parent.insertBefore(child, referenceNode);
```

#### （3）删除节点

```JavaScript
const parent = document.querySelector('.parent');
const child = document.querySelector('.child');
parent.removeChild(child); // 父节点删除子节点
```

#### （4）复制节点

```JavaScript
const box = document.querySelector('.box');
const cloneBox1 = box.cloneNode(false); // 浅拷贝（仅复制自身，不复制子节点）
const cloneBox2 = box.cloneNode(true); // 深拷贝（复制自身及所有子节点）
```

#### （5）查找节点关系

```JavaScript
const child = document.querySelector('.child');

// 父节点
child.parentNode; // 父节点（包括非元素节点，如文本节点）
child.parentElement; // 父元素节点（仅元素节点）

// 子节点
parent.children; // 所有子元素节点（伪数组）
parent.firstElementChild; // 第一个子元素节点
parent.lastElementChild; // 最后一个子元素节点

// 兄弟节点
child.previousElementSibling; // 上一个兄弟元素节点
child.nextElementSibling; // 下一个兄弟元素节点
```

## 十二、事件监听

事件是用户或浏览器的操作（如点击、滚动），JS 通过事件监听响应这些操作

### 1. 事件三要素

- **事件源**：触发事件的元素（如按钮、输入框）
- **事件类型**：事件的类型（如点击 `click`、鼠标经过 `mouseenter`）
- **事件处理函数**：事件触发后执行的代码（回调函数）

### 2. 事件监听语法

```JavaScript
// 1. 基础语法
const btn = document.querySelector('button');
btn.addEventListener('click', function() {
  console.log('按钮被点击了');
});

// 2. 箭头函数（注意 this 指向）
btn.addEventListener('click', () => {
  console.log(this); // window（箭头函数 this 指向外层）
});

// 3. 具名函数（方便解绑）
function handleClick() {
  console.log('点击事件');
}
btn.addEventListener('click', handleClick);
```

### 3. 常见事件类型

| 事件分类 | 常用事件                                         | 说明                                     |
| -------- | ------------------------------------------------ | ---------------------------------------- |
| 鼠标事件 | `click`、`mouseenter`、`mouseleave`、`mousemove` | 点击、鼠标进入、鼠标离开、鼠标移动       |
| 键盘事件 | `keydown`、`keyup`、`keypress`                   | 键盘按下、键盘松开、字符键按下           |
| 表单事件 | `focus`、`blur`、`change`、`submit`、`input`     | 获得焦点、失去焦点、内容改变、提交、输入 |
| 窗口事件 | `load`、`resize`、`scroll`                       | 页面加载完成、窗口尺寸改变、页面滚动     |

### 4. 事件对象（event）

事件触发时，回调函数会接收一个**事件对象**，包含事件相关信息：

```JavaScript
btn.addEventListener('click', function(e) {
  console.log(e.type); // 事件类型："click"
  console.log(e.target); // 事件源（触发事件的元素）
  console.log(e.clientX, e.clientY); // 鼠标相对于窗口的坐标
  console.log(e.offsetX, e.offsetY); // 鼠标相对于事件源的坐标
});

// 键盘事件示例
input.addEventListener('keyup', function(e) {
  if (e.key === 'Enter') { // 检测是否按下回车键
    console.log('按下了回车，输入内容：', this.value);
  }
});
```

### 5. 事件流与阻止冒泡

#### （1）事件流

事件触发后会经历两个阶段：

- **捕获阶段**：从最外层元素向内传播（`addEventListener` 第三个参数为 `true` 时触发）
- **冒泡阶段**：从事件源向外传播（默认触发，`addEventListener` 第三个参数为 `false`）

#### （2）阻止冒泡

默认情况下，子元素事件会冒泡到父元素，可通过 `e.stopPropagation()` 阻止：

```JavaScript
const child = document.querySelector('.child');
const parent = document.querySelector('.parent');

child.addEventListener('click', function(e) {
  console.log('子元素点击');
  e.stopPropagation(); // 阻止冒泡到父元素
});

parent.addEventListener('click', function() {
  console.log('父元素点击'); // 不会触发
});
```

### 6. 阻止默认行为

部分元素有默认行为（如链接跳转、表单提交），可通过 `e.preventDefault()` 阻止：

```JavaScript
// 阻止链接跳转
const link = document.querySelector('a');
link.addEventListener('click', function(e) {
  e.preventDefault(); // 阻止默认跳转行为
  console.log('链接被点击，未跳转');
});

// 阻止表单提交
const form = document.querySelector('form');
form.addEventListener('submit', function(e) {
  e.preventDefault(); // 阻止默认提交
  // 手动处理表单数据
});
```

### 7. 事件委托

利用事件冒泡，将子元素的事件委托给父元素处理（优化性能，适用于动态生成的元素）：

```JavaScript
const ul = document.querySelector('ul');
ul.addEventListener('click', function(e) {
  // 仅处理 li 元素的点击
  if (e.target.tagName === 'LI') {
    console.log('点击了 li 元素：', e.target.innerText);
  }
});
```

## 十三、BOM 操作（浏览器对象模型）

BOM 用于操作浏览器窗口，核心对象包括 `window`、`location`、`history`、`navigator` 等

### 1. window 对象

`window` 是 BOM 的核心对象，代表浏览器窗口，所有全局变量和函数都是 `window` 的属性：

```JavaScript
// 全局变量本质是 window 的属性
let a = 10;
console.log(window.a); // 10

// 全局函数本质是 window 的方法
function fn() {}
console.log(window.fn); // 函数本身
```

#### 常用 window 方法

- **定时器**：
    - ```JavaScript
        // 延时函数（仅执行一次）
        const timer1 = setTimeout(function() {
          console.log('1秒后执行');
        }, 1000);
        clearTimeout(timer1); // 清除延时函数
        
        // 间隙函数（重复执行）
        const timer2 = setInterval(function() {
          console.log('每隔1秒执行一次');
        }, 1000);
        clearInterval(timer2); // 清除间隙函数
        ```
- **窗口操作**：
    - ```JavaScript
        window.open('https://www.baidu.com'); // 打开新窗口
        window.close(); // 关闭当前窗口
        window.alert('提示'); // 警示框
        window.confirm('确定吗？'); // 确认框（返回布尔值）
        ```

### 2. location 对象

用于操作浏览器地址栏，核心属性和方法：

```JavaScript
// 属性
console.log(location.href); // 完整 URL（如 "https://www.baidu.com/s?wd=js"）
console.log(location.protocol); // 协议（如 "https:"）
console.log(location.host); // 主机名+端口（如 "www.baidu.com"）
console.log(location.pathname); // 路径（如 "/s"）
console.log(location.search); // 查询参数（如 "?wd=js"）
console.log(location.hash); // 锚点（如 "#top"）

// 方法
location.href = 'https://www.baidu.com'; // 跳转页面（推荐）
location.assign('https://www.baidu.com'); // 跳转页面（可后退）
location.replace('https://www.baidu.com'); // 跳转页面（不可后退，替换历史记录）
location.reload(); // 刷新页面（默认缓存刷新）
location.reload(true); // 强制刷新（忽略缓存）
```

### 3. history 对象

用于操作浏览器历史记录（与地址栏前进/后退按钮对应）：

```JavaScript
history.back(); // 后退一页（等同于点击后退按钮）
history.forward(); // 前进一页（等同于点击前进按钮）
history.go(1); // 前进1页（参数为负数则后退）
history.go(-1); // 后退1页
history.go(0); // 刷新页面
console.log(history.length); // 历史记录长度
```

### 4. navigator 对象

用于获取浏览器相关信息（如判断设备类型）：

```JavaScript
console.log(navigator.userAgent); // 浏览器 User-Agent 字符串（判断浏览器/设备）
console.log(navigator.appName); // 浏览器名称
console.log(navigator.platform); // 操作系统平台
```

### 5. 页面滚动与尺寸

#### （1）页面滚动

```JavaScript
// 获取页面滚动距离（兼容写法）
const scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
const scrollLeft = document.documentElement.scrollLeft || document.body.scrollLeft;

// 设置页面滚动到指定位置
window.scrollTo(0, 1000); // 滚动到顶部1000px处（x, y）
window.scrollTo({
  top: 1000,
  behavior: 'smooth' // 平滑滚动
});
```

#### （2）页面尺寸

```JavaScript
// 窗口可视区域尺寸（不含滚动条）
const clientWidth = document.documentElement.clientWidth;
const clientHeight = document.documentElement.clientHeight;

// 页面总尺寸（含滚动区域）
const scrollWidth = document.documentElement.scrollWidth;
const scrollHeight = document.documentElement.scrollHeight;

// 元素尺寸与位置
const box = document.querySelector('.box');
console.log(box.offsetWidth); // 元素宽度（含边框、内边距）
console.log(box.offsetHeight); // 元素高度（含边框、内边距）
console.log(box.offsetLeft); // 元素相对于定位父级的左偏移
console.log(box.offsetTop); // 元素相对于定位父级的上偏移
```

## 十四、本地存储

浏览器提供的本地数据存储方案，核心有 `localStorage` 和 `sessionStorage`（仅在客户端存储，不发送到服务器）

### 1. 核心特性对比

| 特性     | localStorage               | sessionStorage                               |
| -------- | -------------------------- | -------------------------------------------- |
| 存储时长 | 永久存储（除非手动删除）   | 会话级存储（关闭标签页/浏览器后清除）        |
| 作用域   | 同一域名下所有页面共享     | 仅当前标签页共享（同一域名不同标签页不共享） |
| 存储大小 | 约 5MB                     | 约 5MB                                       |
| 数据类型 | 仅支持字符串（需手动转换） | 仅支持字符串（需手动转换）                   |

### 2. 核心方法（两者用法一致）

```JavaScript
// 1. 存储数据（键值对，值必须是字符串）
localStorage.setItem('name', '张三');
localStorage.setItem('age', '18'); // 数字也会转为字符串

// 2. 获取数据
const name = localStorage.getItem('name');
const age = localStorage.getItem('age');

// 3. 删除数据
localStorage.removeItem('age');

// 4. 清空所有数据
localStorage.clear();

// 5. 获取键名（根据索引）
const key = localStorage.key(0); // 获取第一个键名

// 6. 获取存储数量
const length = localStorage.length;
```

### 3. 存储复杂数据类型（对象/数组）

本地存储仅支持字符串，存储对象/数组需先序列化（`JSON.stringify`），读取时反序列化（`JSON.parse`）：

```JavaScript
// 存储对象
const user = { name: '张三', age: 18 };
localStorage.setItem('user', JSON.stringify(user));

// 读取对象
const storedUser = JSON.parse(localStorage.getItem('user'));
console.log(storedUser.name); // "张三"

// 存储数组
const arr = [1, 2, 3];
localStorage.setItem('arr', JSON.stringify(arr));

// 读取数组
const storedArr = JSON.parse(localStorage.getItem('arr'));
```

## 十五、JavaScript 执行机制

JS 是**单线程**语言（同一时间只能执行一个任务），为解决阻塞问题，采用**同步异步**机制：

### 1. 同步与异步

- **同步任务**：按顺序执行，阻塞后续代码（如变量声明、函数调用、循环等）
- **异步任务**：不阻塞后续代码，先放入任务队列，等待同步任务执行完后再执行（如定时器、DOM 事件、AJAX 请求）

### 2. 事件循环（Event Loop）

JS 执行流程：

1. 同步任务进入**主线程**执行
2. 异步任务进入**异步队列**（如定时器触发后、事件触发后）
3. 主线程同步任务执行完毕后，轮询读取异步队列中的任务，放入主线程执行
4. 重复步骤 3（事件循环）

### 3. 常见异步任务

- 定时器（`setTimeout`、`setInterval`）
- DOM 事件（`click`、`load` 等）
- AJAX 请求（`fetch`、`XMLHttpRequest`）
- Promise 回调（`then`、`catch`）

## 十六、TypeScript 基础（TS）

TypeScript 是 JavaScript 的超集，添加了**静态类型系统**，编译后生成 JS 代码，核心特性如下：

### 1. 变量声明与类型注解

TS 要求明确变量类型（或通过类型推断），语法：`let 变量名: 类型 = 值`

```TypeScript
// 基础类型
let num: number = 10;
let str: string = 'abc';
let isDone: boolean = false;
let undef: undefined = undefined;
let nul: null = null;

// 联合类型（多种类型可选）
let id: number | string = 123;
id = 'abc'; // 允许

// 数组类型（两种写法）
let arr1: number[] = [1, 2, 3];
let arr2: Array<string> = ['a', 'b', 'c'];

// 元组类型（固定长度和类型的数组）
let tuple: [string, number] = ['张三', 18];

// 任意类型（禁用类型检查，不推荐）
let anyVal: any = 10;
anyVal = 'abc';
anyVal = true;

// 无返回值函数
function fn(): void {
  console.log('无返回值');
}

// 永不返回（死循环/报错）
function error(): never {
  throw new Error('报错');
}
```

### 2. 函数类型

```TypeScript
// 基础函数类型
function add(a: number, b: number): number {
  return a + b;
}

// 可选参数（必须放在最后）
function greet(name: string, age?: number): string {
  return age ? `你好，${name}，${age}岁` : `你好，${name}`;
}

// 默认参数
function multiply(a: number, b: number = 1): number {
  return a * b;
}

// 剩余参数
function sum(...args: number[]): number {
  return args.reduce((a, b) => a + b, 0);
}

// 箭头函数类型
const subtract: (a: number, b: number) => number = (a, b) => a - b;
```

### 3. 接口（Interface）

用于定义对象的结构（类型契约），支持可选属性、只读属性等：

```TypeScript
interface Person {
  readonly id: number; // 只读属性（不可修改）
  name: string;
  age?: number; // 可选属性
  [key: string]: any; // 任意属性（允许添加未定义的属性）
}

const person: Person = {
  id: 1,
  name: '张三',
  age: 18,
  gender: '男' // 任意属性
};

person.id = 2; // 报错（只读属性）
```

### 4. 类（Class）

TS 增强了 ES6 类的功能，支持访问修饰符、继承、接口实现等：

```TypeScript
// 基础类
class Animal {
  // 访问修饰符：public（默认，任意访问）、private（仅类内部访问）、protected（类内部+子类访问）
  private name: string;
  public age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  sayHello(): void {
    console.log(`我是${this.name}，${this.age}岁`);
  }
}

// 子类继承
class Dog extends Animal {
  constructor(name: string, age: number) {
    super(name, age); // 调用父类构造函数
  }

  bark(): void {
    console.log('汪汪汪');
  }
}

const dog = new Dog('旺财', 3);
dog.sayHello(); // "我是旺财，3岁"
dog.bark(); // "汪汪汪"
```

### 5. 类型推断与类型守卫

- **类型推断**：TS 会自动推断未显式声明的变量类型
    - ```TypeScript
        let num = 10; // 推断为 number 类型
        num = 'abc'; // 报错（类型不匹配）
        ```
- **类型守卫**：用于缩小变量类型范围（如 `typeof`、`instanceof`）
    - ```TypeScript
        function getType(value: number | string): string {
          if (typeof value === 'number') {
            return '数字';
          }
          return '字符串';
        }
        ```

## 十七、常用工具与技巧

### 1. 正则表达式

用于字符串匹配和处理，核心语法：

```JavaScript
// 定义正则（字面量方式）
const reg = /^\d{3}-\d{4}$/; // 匹配 123-4567 格式

// 测试匹配（返回布尔值）
reg.test('123-4567'); // true
reg.test('1234-567'); // false

// 捕获匹配结果（返回数组）
const str = '电话：123-4567，邮箱：test@xxx.com';
const phoneReg = /(\d{3})-(\d{4})/;
const result = phoneReg.exec(str);
console.log(result[1]); // "123"（第一个分组）
console.log(result[2]); // "4567"（第二个分组）
```

#### 常用元字符：

- 边界符：`^`（开头）、`$`（结尾）
- 量词：`*`（0+次）、`+`（1+次）、`?`（0-1次）、`{n}`（n次）、`{n,}`（n+次）、`{n,m}`（n-m次）
- 字符类：`[a-z]`（小写字母）、`[0-9]`（数字）、`\d`（数字）、`\w`（字母+数字+下划线）、`.`（任意字符，除换行）

### 2. 数组常用方法（补充）

- **`reduce`**：累加/归约（强大的数组处理方法）
    - ```JavaScript
        // 求和
        const sum = [1, 2, 3].reduce((acc, cur) => acc + cur, 0); // 6
        
        // 数组转对象
        const arr = [
          { id: 1, name: '张三' },
          { id: 2, name: '李四' }
        ];
        const obj = arr.reduce((acc, cur) => {
          acc[cur.id] = cur.name;
          return acc;
        }, {}); // { 1: "张三", 2: "李四" }
        ```
- **`find`****/****`findIndex`**：查找元素/索引
    - ```JavaScript
        const arr = [1, 2, 3, 4];
        const found = arr.find(item => item > 2); // 3（第一个匹配元素）
        const index = arr.findIndex(item => item > 2); // 2（第一个匹配元素索引）
        ```

### 3. 解构赋值

快速从数组或对象中提取数据，简化代码：

```JavaScript
// 数组解构
const [a, b, ...rest] = [1, 2, 3, 4]; // a=1, b=2, rest=[3,4]

// 对象解构
const { name, age } = { name: '张三', age: 18 };

// 重命名
const { name: userName, age: userAge } = { name: '张三', age: 18 };

// 默认值
const [x = 0, y = 0] = [1]; // x=1, y=0
```

### 4. 模板字符串

用反引号（```）包裹，支持变量插值和多行字符串：

```JavaScript
const name = '张三';
const age = 18;

// 变量插值（${} 中可放表达式）
const str = `我是${name}，${age}岁，明年${age + 1}岁`;

// 多行字符串（无需拼接）
const html = `
  <div class="box">
    <p>${name}</p>
  </div>
`;
```

## 十八、常见问题与最佳实践

### 1. 变量命名规范

- 采用小驼峰命名（`userName`、`getUserInfo`）
- 常量用全大写+下划线（`MAX_COUNT`、`API_BASE_URL`）
- 类/构造函数用大驼峰（`Person`、`UserService`）
- 避免使用关键字（`let`、`function`、`if` 等）

### 2. 性能优化

- 减少 DOM 操作（频繁操作时先存为变量，批量更新）
- 避免使用 `var`，优先用 `let`/`const`
- 合理使用事件委托（减少事件监听数量）
- 避免内存泄漏（清除定时器、解绑事件、释放DOM引用）

### 3. 常见 Bug 排查

- `NaN`：检查是否有字符串与数字运算，用 `isNaN()` 判断
- `undefined`：检查变量是否声明赋值，对象属性是否存在（用可选链 `?.`）
- 数组越界：检查数组长度，避免索引超出范围
- 事件冒泡：必要时用 `e.stopPropagation()` 阻止

### 4. 代码规范

- 统一缩进（2 空格或 4 空格）
- 语句结尾加分号（避免自动插入分号 Bug）
- 用 `===` 代替 `==`（避免隐式类型转换）
- 复杂逻辑拆分函数（单一职责原则）
- 注释关键业务逻辑（避免"自解释"代码误区）