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