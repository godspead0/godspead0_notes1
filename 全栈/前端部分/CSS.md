# 主题：CSS 全面知识点详解（包括基础入门、选择器、核心样式属性等内容）

# CSS 全面知识点详解（扩展版）

CSS（Cascading Style Sheets，层叠样式表）是用于描述 HTML 文档呈现样式的语言，核心作用是分离文档结构与表现形式，实现网页美化、布局控制和交互效果。本文基于基础知识点，进行系统扩展与结构化梳理，涵盖核心语法、选择器、样式属性、布局模型、高级特性等内容。

## 一、CSS 基础入门

### 1.1 核心概念

- **作用**：控制 HTML 元素的外观（颜色、字体、间距）、布局（位置、大小、排列）、响应式适配和交互反馈。
- **核心思想**：「选择器 + 声明块」，通过选择器定位 HTML 元素，通过声明块（属性-值对）定义样式。
- **本质**：层叠（样式优先级叠加）、继承（子元素继承父元素样式）、优先级（冲突样式的选择规则）。

### 1.2 CSS 引入方式（3种）

| 引入方式   | 语法格式                                                     | 适用场景                      | 优先级                   |
| ---------- | ------------------------------------------------------------ | ----------------------------- | ------------------------ |
| 内部样式表 | 在 `<head>` 中通过 `<style>` 标签编写 CSS                    | 单页面样式、小型项目          | 中等（次于行内）         |
| 外部样式表 | 单独创建 `.css` 文件，通过 `<link rel="stylesheet" href="路径">` 引入 | 多页面复用、大型项目          | 最低（可通过优先级覆盖） |
| 行内样式表 | 直接在 HTML 标签的 `style` 属性中编写（`style="属性:值;"`）  | 单个元素特殊样式、JS 动态修改 | 最高                     |

#### 补充说明：

- 外部样式表的 `link` 标签属性：
    - `rel="stylesheet"`：告诉浏览器当前文件是样式表文件（必需）。
    - `href`：指定 CSS 文件的**相对路径**（如 `css/style.css`）或绝对路径（如 `https://xxx.com/style.css`）。
    - `media`：指定样式适用的设备（如 `media="screen"` 适配屏幕，`media="print"` 适配打印）。
- 行内样式示例：`<div style="color: red; font-size: 16px;">行内样式</div>`，仅作用于当前标签，无法复用。

### 1.3 基础语法规则

```CSS
/* 注释：单行注释（/* 多行注释 */） */
选择器 {
  属性名1: 属性值1; /* 声明1：键值对，分号结尾 */
  属性名2: 属性值2; /* 声明2：多个声明用分号分隔，大括号包裹 */
}
```

- **语法要求**：
    - 属性名与属性值之间用冒号 `:` 连接，多个声明用分号 `;` 结尾（最后一个声明可省略分号，但建议保留）。
    - 属性值为字符串时（如字体名含空格）需加引号（单双引号均可），如 `font-family: "Microsoft YaHei";`。
    - 大小写不敏感，但建议统一小写（如 `color` 而非 `Color`）。

## 二、CSS 选择器（精准定位元素）

选择器是 CSS 的核心，用于筛选需要应用样式的 HTML 元素。分为**基础选择器**和**复合选择器**，扩展常用高级选择器。

### 2.1 基础选择器

| 选择器类型   | 语法格式          | 作用范围                      | 示例                                                         |
| ------------ | ----------------- | ----------------------------- | ------------------------------------------------------------ |
| 标签选择器   | 标签名            | 页面中所有该标签              | `p { color: red; }`（所有 `<p>` 变红）                       |
| 类选择器     | `.类名`           | 所有 `class="类名"` 的元素    | `.red { color: red; }`（class 为 red 的元素变红）            |
| ID 选择器    | `#ID名`           | 页面中唯一 `id="ID名"` 的元素 | `#header { height: 100px; }`（id 为 header 的元素）          |
| 通配符选择器 | `*`               | 页面中所有元素                | `* { margin: 0; padding: 0; }`（清除默认边距）               |
| 属性选择器   | 选择器[属性名=值] | 具有指定属性及值的元素        | `input[type="text"] { border: 1px solid #ccc; }`（文本输入框样式） |

#### 补充说明：

- 类选择器支持**多类名复用**：一个元素可添加多个类，用空格分隔，如 `<div class="red larger">`，同时应用 `.red` 和 `.larger` 的样式。
- ID 选择器**唯一性要求**：一个页面中 ID 不可重复，否则违反 HTML 规范，且 JavaScript 获取元素时会出错。
- 属性选择器扩展用法：
    - `[attr]`：选择具有 `attr` 属性的元素（无论值是什么），如 `[disabled] { cursor: not-allowed; }`（所有禁用元素的光标样式）。
    - `[attr^=val]`：选择 `attr` 属性值以 `val` 开头的元素，如 `[href^="http"] { color: blue; }`（所有 http 链接）。
    - `[attr$=val]`：选择 `attr` 属性值以 `val` 结尾的元素，如 `[src$=".png"] { border: none; }`（所有 png 图片）。

### 2.2 复合选择器（组合基础选择器）

| 选择器类型     | 语法格式                 | 作用逻辑                                   | 示例                                                         |
| -------------- | ------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| 后代选择器     | 选择器1 选择器2          | 选择器1 的**所有后代**（子、孙、重孙）元素 | `div .box { margin: 10px; }`（div 内所有 class 为 box 的元素） |
| 子代选择器     | 选择器1 > 选择器2        | 选择器1 的**直接子元素**（仅一代）         | `ul > li { list-style: none; }`（ul 的直接子元素 li，不包含 li 内的 li） |
| 并集选择器     | 选择器1, 选择器2, ...    | 同时选择多个选择器的元素                   | `h1, h2, h3 { font-weight: normal; }`（h1-h3 均取消加粗）    |
| 交集选择器     | 选择器1选择器2（无空格） | 同时满足多个选择器条件的元素               | `p.red { color: red; }`（class 为 red 的 p 标签，非其他标签） |
| 相邻兄弟选择器 | 选择器1 + 选择器2        | 选择器1 的**紧接后一个兄弟元素**           | `div + p { margin-top: 20px; }`（div 后面紧跟的 p 标签）     |
| 通用兄弟选择器 | 选择器1 ~ 选择器2        | 选择器1 的**所有后续兄弟元素**             | `input:checked ~ span { color: green; }`（选中的 input 后面所有 span 变绿） |

### 2.3 伪类选择器（状态/位置筛选）

伪类用于选择元素的**特定状态**（如鼠标悬停）或**结构位置**（如第一个子元素），语法为 `选择器:伪类名`。

#### （1）状态伪类

| 伪类名      | 作用                 | 适用元素                         |
| ----------- | -------------------- | -------------------------------- |
| `:hover`    | 鼠标悬停时的状态     | 所有元素（常用於按钮、链接）     |
| `:active`   | 元素被点击时的状态   | 按钮、链接、输入框               |
| `:focus`    | 元素获得焦点时的状态 | 输入框、文本域、链接             |
| `:link`     | 未访问过的超链接     | `<a>` 标签                       |
| `:visited`  | 已访问过的超链接     | `<a>` 标签（仅支持颜色相关样式） |
| `:checked`  | 表单元素被选中的状态 | 复选框、单选框、下拉框           |
| `:disabled` | 表单元素被禁用的状态 | 输入框、按钮等表单元素           |
| `:enabled`  | 表单元素可用的状态   | 与 `:disabled` 相反              |

#### （2）结构伪类

| 伪类名               | 作用                          | 示例                                                         |
| -------------------- | ----------------------------- | ------------------------------------------------------------ |
| `:first-child`       | 父元素的第一个子元素          | `ul li:first-child { color: red; }`（ul 的第一个 li）        |
| `:last-child`        | 父元素的最后一个子元素        | `ul li:last-child { margin-right: 0; }`（ul 的最后一个 li）  |
| `:nth-child(n)`      | 父元素的第 n 个子元素         | `ul li:nth-child(2) { font-weight: bold; }`（ul 的第 2 个 li） |
| `:nth-last-child(n)` | 父元素的倒数第 n 个子元素     | `ul li:nth-last-child(1) { color: blue; }`（ul 的倒数第 1 个 li） |
| `:only-child`        | 父元素中唯一的子元素          | `div:only-child { width: 100%; }`（父元素中唯一的 div）      |
| `:empty`             | 无任何子元素（含文本）的元素  | `p:empty { display: none; }`（空 p 标签隐藏）                |
| `:root`              | HTML 文档的根元素（`<html>`） | `:root { --main-color: red; }`（定义全局 CSS 变量）          |
| `:target`            | 被锚点链接选中的元素          | `#section1:target { background: #f5f5f5; }`（点击 `#section1` 锚点时选中元素） |

#### 关键补充：

- `:nth-child(n)` 的 `n` 支持多种写法：
    - 数字：`n=3` 表示第 3 个元素。
    - 关键字：`odd`（奇数，1、3、5...）、`even`（偶数，2、4、6...）。
    - 公式：`2n`（偶数）、`2n+1`（奇数）、`n+3`（从第 3 个开始）、`-n+5`（前 5 个元素）。
- 超链接伪类顺序：**`:link`** **→** **`:visited`** **→** **`:hover`** **→** **`:active`**（LVHA 顺序），顺序错误会导致样式失效（如 `:hover` 写在 `:link` 前面会被覆盖）。

### 2.4 伪元素选择器（创建虚拟元素）

伪元素用于在元素**内容前后插入虚拟元素**（不影响 HTML 结构），语法为 `选择器::伪元素名`（CSS3 规范，双冒号区分伪类）。

| 伪元素名         | 作用                           | 核心要求                                            | 示例                                                         |
| ---------------- | ------------------------------ | --------------------------------------------------- | ------------------------------------------------------------ |
| `::before`       | 在元素内容**之前**插入虚拟元素 | 必须包含 `content` 属性（空值也需写 `content: ""`） | `p::before { content: "【"; color: red; }`（p 内容前加红色「【」） |
| `::after`        | 在元素内容**之后**插入虚拟元素 | 同上                                                | `a::after { content: "→"; margin-left: 5px; }`（链接后加箭头） |
| `::first-letter` | 选中元素的第一个字符           | 仅作用于块级元素                                    | `p::first-letter { font-size: 2em; }`（p 第一个字放大 2 倍） |
| `::first-line`   | 选中元素的第一行文本           | 仅作用于块级元素                                    | `p::first-line { color: blue; }`（p 第一行文本变蓝）         |
| `::selection`    | 选中用户选中的文本内容         | 仅支持部分样式（颜色、背景等）                      | `::selection { background: yellow; color: black; }`（选中文本黄底黑字） |

#### 补充说明：

- 伪元素默认是**行内元素**（`display: inline`），如需设置宽高，需手动改为块级或行内块：`display: block` 或 `display: inline-block`。
- `content` 属性支持多种值：文本（直接写字符串）、图片（`content: url("img/icon.png")`）、属性值（`content: attr(data-text)`，获取元素的 `data-text` 属性值）。

### 2.5 选择器优先级（冲突解决规则）

当多个选择器同时作用于一个元素且样式冲突时，按以下优先级排序（优先级高的覆盖低的）：

#### （1）基础优先级顺序（从高到低）

`!important` → 行内样式（`style` 属性） → ID 选择器 → 类选择器/伪类选择器/属性选择器 → 标签选择器 → 通配符选择器 → 继承样式

#### （2）关键规则

- `!important`：强制提升单个属性的优先级（仅对当前属性生效），语法：`color: red !important;`（分号前，属性值后）。
    - 注意：慎用 `!important`，过度使用会破坏优先级逻辑，难以维护。
- 继承样式优先级最低：子元素未设置的样式会继承父元素，但父元素样式优先级低于子元素自身的任何样式。
- 复合选择器优先级：按「权重叠加」计算，规则如下：
    - 标签选择器、伪元素选择器：权重 1。
    - 类选择器、伪类选择器、属性选择器：权重 10。
    - ID 选择器：权重 100。
    - 行内样式：权重 1000。
    - 通配符选择器：权重 0。
    - 叠加计算：如 `div#header .nav li` 的权重 = 1（div）+ 100（#header）+ 10（.nav）+ 1（li）= 112。
    - 优先级相同：后定义的样式覆盖先定义的（层叠性）。

## 三、CSS 核心样式属性

### 3.1 文字与文本样式

用于控制文字的字体、大小、颜色、对齐方式等，大部分属性支持**继承**（子元素默认继承父元素的文字样式）。

| 属性名            | 作用                 | 取值范围/示例                                                | 补充说明                                                     |
| ----------------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `color`           | 文字颜色             | 单词（red）、RGB（rgb(255,0,0)）、RGBA（rgba(255,0,0,0.5)）、十六进制（#ff0000） | RGBA 的第四个值是透明度（0-1），仅影响文字                   |
| `font-size`       | 字体大小             | 像素（px）、em（当前字号倍数）、rem（根元素字号倍数）、百分比 | 谷歌浏览器默认字号 16px，rem 用于响应式布局                  |
| `font-weight`     | 字体粗细             | normal（400）、bold（700）、100-900（数字越大越粗）          | 取消 h1-h6 加粗：`font-weight: normal`                       |
| `font-style`      | 字体倾斜             | normal（默认，取消倾斜）、italic（倾斜）、oblique（倾斜，兼容性略差） | 取消 em 标签倾斜：`em { font-style: normal; }`               |
| `line-height`     | 行高（文字上下间距） | 像素（px）、数字（当前字号倍数）、百分比                     | 单行文字垂直居中：`line-height = 盒子高度`                   |
| `font-family`     | 字体家族             | 多个字体用逗号分隔，如 `"Microsoft YaHei", sans-serif`       | 最后一个写字体族名（sans-serif 无衬线、serif 衬线），防止字体未找到 |
| `font`            | 字体复合属性         | 顺序：`font-style font-weight line-height font-size font-family` | 必需包含 `font-size` 和 `font-family`，否则无效，如 `font: italic bold 1.5em 16px "Microsoft YaHei";` |
| `text-align`      | 文本对齐方式         | left（左）、center（中）、right（右）、justify（两端对齐）   | 控制子元素（文字、图片）对齐，需给父元素设置                 |
| `text-decoration` | 文本修饰             | none（无，取消下划线）、underline（下划线）、overline（上划线）、line-through（删除线） | 取消超链接下划线：`a { text-decoration: none; }`             |
| `text-transform`  | 文字大小写           | none、uppercase（全大写）、lowercase（全小写）、capitalize（首字母大写） | 仅对英文有效                                                 |
| `text-indent`     | 首行缩进             | 像素（px）、em（当前字号倍数）                               | 缩进 2 字符：`text-indent: 2em`（推荐）                      |
| `text-shadow`     | 文字阴影             | 语法：`水平偏移 垂直偏移 模糊半径 扩散半径 颜色`             | 示例：`text-shadow: 2px 2px 3px rgba(0,0,0,0.3);`（右下阴影，模糊 3px，透明度 0.3） |
| `letter-spacing`  | 字符间距             | 像素（px）、em                                               | 增加间距：`letter-spacing: 2px`，减少间距：`letter-spacing: -1px` |
| `word-spacing`    | 单词间距             | 像素（px）、em                                               | 仅对英文单词有效（以空格分隔）                               |

### 3.2 背景样式

控制元素的背景颜色、图片、平铺方式等，支持复合属性简化写法。

| 属性名                  | 作用             | 取值范围/示例                                                | 补充说明                                                     |
| ----------------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `background-color`      | 背景颜色         | 同 `color` 的取值（单词、RGB、十六进制等）                   | 透明背景：`background-color: transparent`（默认）            |
| `background-image`      | 背景图片         | `url("图片路径")`（相对/绝对路径）                           | 默认平铺，图片路径错误会显示空白背景                         |
| `background-repeat`     | 背景图片平铺方式 | repeat（默认，全平铺）、no-repeat（不平铺）、repeat-x（水平平铺）、repeat-y（垂直平铺） | 不平铺时图片默认显示在左上角                                 |
| `background-position`   | 背景图片位置     | 关键字（left/center/right + top/center/bottom）、像素（px）、百分比（%） | 示例：`center center`（居中）、`50px 100px`（左移 50px，下移 100px）、`50% 50%`（居中，与关键字等效）；仅写一个值时，另一个默认 center |
| `background-attachment` | 背景图片固定方式 | scroll（默认，随页面滚动）、fixed（固定，不随页面滚动）      | 固定背景常用于全屏背景图                                     |
| `background-size`       | 背景图片大小     | cover（覆盖盒子，可能裁剪）、contain（适应盒子，可能留白）、百分比（如 `100% 100%`）、像素（如 `200px 150px`） | `cover` 常用於全屏背景，`contain` 常用於图标适配             |
| `background-clip`       | 背景裁剪范围     | border-box（默认，包含边框）、padding-box（仅内边距区域）、content-box（仅内容区域） | 配合透明边框使用，如 `border: 5px solid transparent; background-clip: padding-box;` |
| `background-origin`     | 背景图片定位原点 | border-box（边框左上角）、padding-box（内边距左上角，默认）、content-box（内容区域左上角） | 与 `background-position` 配合使用                            |
| `background`            | 背景复合属性     | 顺序：`background-color background-image background-repeat background-position background-attachment / background-size` | 必需保留 `background-size` 前的 `/`，示例：`background: #f5f5f5 url("img/bg.jpg") no-repeat center center fixed / cover;` |

### 3.3 盒子模型（布局基础）

所有 HTML 元素都可视为「盒子」，由「内容区（content）、内边距（padding）、边框（border）、外边距（margin）」组成，是网页布局的核心模型。

#### （1）盒子模型核心属性

| 属性类型 | 相关属性                                         | 作用                     | 取值/示例                                                    |
| -------- | ------------------------------------------------ | ------------------------ | ------------------------------------------------------------ |
| 内容区   | `width`、`height`                                | 盒子内容的宽高           | 像素（px）、百分比（%）、auto（默认，自适应）                |
| 内边距   | `padding`、`padding-top`/`right`/`bottom`/`left` | 内容区与边框之间的距离   | 多值写法：`padding: 10px`（四边）、`10px 20px`（上下 10，左右 20）、`10px 20px 30px`（上 10，左右 20，下 30）、`10px 20px 30px 40px`（上右下左） |
| 边框     | `border`、`border-方向`                          | 盒子的边框               | 复合属性：`border: 1px solid red`（宽度 1px，实线，红色）；单独设置：`border-top: 2px dashed #ccc`（上边框虚线） |
| 外边距   | `margin`、`margin-top`/`right`/`bottom`/`left`   | 盒子与其他盒子之间的距离 | 多值写法同 padding；水平居中：`margin: 0 auto`（需设置宽度） |

#### （2）盒子尺寸计算

- **默认模式（content-box）**：盒子总宽度 = `width` + `padding-left` + `padding-right` + `border-left` + `border-right`（margin 不计算在盒子本身尺寸内，仅影响与其他盒子的间距）。
    - 问题：添加 padding 或 border 会「撑大盒子」，导致布局错乱。
- **内减模式（border-box）**：盒子总宽度 = `width`（包含 padding 和 border），无需手动计算减法。
    - 解决：全局设置 `box-sizing: border-box;`，推荐写法：
        - ```CSS
            * {
              margin: 0;
              padding: 0;
              box-sizing: border-box; /* 统一内减模式，避免布局问题 */
            }
            ```

#### （3）盒子模型常见问题与解决方案

| 问题类型         | 现象描述                                                     | 解决方案                                                     |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 外边距合并       | 两个垂直排列的兄弟元素，上下 margin 取最大值（而非相加）     | 1. 只给一个元素设置 margin（如只设下元素的 margin-top）；2. 用 padding 替代 margin；3. 给其中一个元素添加父容器并设置 `overflow: hidden` |
| 外边距塌陷       | 父子元素中，子元素的 margin-top 会传递给父元素（导致父元素下移） | 1. 给父元素设置 padding-top 替代子元素的 margin-top；2. 给父元素设置 `overflow: hidden`；3. 给父元素设置 border-top（如 `border-top: 1px solid transparent`）；4. 给父元素添加 `::before` 伪元素清除塌陷 |
| 行内元素边距无效 | 行内元素（如 span、a）的 margin-top/bottom 和 padding-top/bottom 无法改变垂直位置 | 1. 用 `line-height` 控制垂直间距；2. 将行内元素转为行内块（`display: inline-block`）或块级元素（`display: block`） |
| 元素溢出         | 内容超出盒子尺寸（如文字、图片）                             | 用 `overflow` 控制：`visible`（默认，显示溢出）、`hidden`（隐藏溢出）、`scroll`（强制显示滚动条）、`auto`（溢出时显示滚动条）；单独控制方向：`overflow-x`（水平）、`overflow-y`（垂直） |

#### （4）盒子美化属性

| 属性名          | 作用         | 取值/示例                                                    |
| --------------- | ------------ | ------------------------------------------------------------ |
| `border-radius` | 圆角边框     | 像素（px）、百分比（%）；多值写法同 padding；正圆：`border-radius: 50%`（需盒子为正方形）；胶囊形状：`border-radius: 盒子高度的一半`（如高度 40px，圆角 20px） |
| `box-shadow`    | 盒子阴影     | 语法：`水平偏移 垂直偏移 模糊半径 扩散半径 颜色 内外阴影`；示例：`box-shadow: 0 2px 10px rgba(0,0,0,0.1);`（下阴影，模糊 10px，浅灰色）；内阴影：添加 `inset`，如 `box-shadow: inset 0 0 5px #ccc;` |
| `opacity`       | 盒子透明度   | 0-1 之间的数值（0 完全透明，1 不透明）                       |
| `cursor`        | 鼠标光标类型 | default（默认箭头）、pointer（小手，链接/按钮）、text（文本光标，输入框）、move（移动光标）、not-allowed（禁止光标，禁用元素） |

### 3.4 显示模式（元素布局行为）

HTML 元素的默认布局行为由「显示模式」决定，可通过 `display` 属性修改，核心分为 3 类：

| 显示模式                   | 核心特征                                                     | 默认元素                    | `display` 属性值        |
| -------------------------- | ------------------------------------------------------------ | --------------------------- | ----------------------- |
| 块级元素（block）          | 1. 独占一行；2. 宽度默认父元素 100%；3. 支持宽高（width/height）、margin/padding 全方向生效 | div、p、h1-h6、ul、li、form | `display: block`        |
| 行内元素（inline）         | 1. 一行可容纳多个；2. 宽度默认随内容自适应；3. 不支持宽高；4. margin/padding 仅水平方向生效（垂直方向不改变布局） | span、a、em、i、strong      | `display: inline`       |
| 行内块元素（inline-block） | 1. 一行可容纳多个；2. 支持宽高；3. margin/padding 全方向生效；4. 元素间默认有空白间距（由 HTML 换行/空格导致） | img、input、button、select  | `display: inline-block` |

#### 补充说明：

- 显示模式转换：通过 `display` 属性修改，如：
    - 行内元素转块级：`span { display: block; }`（span 独占一行，支持宽高）。
    - 块级元素转行内块：`div { display: inline-block; }`（多个 div 同行排列）。
- 行内块元素空白间距解决：
    - 方法 1：父元素设置 `font-size: 0;`，子元素单独设置 `font-size`（推荐）。
    - 方法 2：HTML 中移除元素间的换行/空格（不推荐，影响代码可读性）。

## 四、CSS 布局模型（核心重点）

布局是 CSS 的核心应用，用于控制元素在页面中的位置和排列方式，常用布局模型包括「标准流」「浮动」「Flex 弹性布局」「定位布局」，其中 Flex 布局是目前最常用的现代布局方案。

### 4.1 标准流（Normal Flow）

- **定义**：浏览器默认的布局方式，遵循「块级元素独占一行、行内元素同行排列」的规则。
- **特点**：元素按 HTML 书写顺序自然排列，不脱离文档流（占据页面空间）。
- **适用场景**：简单布局（如单栏文本、基础列表）。
- **局限**：无法实现复杂布局（如多栏并列、元素重叠、精准定位），需结合其他布局模型。

### 4.2 浮动布局（Float）

- **定义**：通过 `float` 属性使元素脱离标准流，向父元素的左侧或右侧浮动，环绕在其他元素周围（最初用于文字环绕图片）。
- **核心属性**：`float: left`（左浮动）、`float: right`（右浮动）、`float: none`（默认，不浮动）。
- **关键特性**：
    - 浮动元素脱离标准流（不占据页面空间），可能覆盖其他元素。
    - 浮动元素具有「行内块特性」（支持宽高，同行排列）。
    - 父元素若未设置高度，且所有子元素均浮动，父元素会「高度塌陷」（高度为 0）。

#### （1）浮动布局示例（实现两栏布局）

```HTML
<div class="parent">
  <div class="left">左栏（宽度 200px）</div>
  <div class="right">右栏（自适应剩余宽度）</div>
</div>
.parent {
  width: 100%;
  height: 300px;
  border: 1px solid #ccc;
}
.left {
  float: left;
  width: 200px;
  height: 100%;
  background: #f5f5f5;
}
.right {
  margin-left: 200px; /* 避开左浮动元素，防止被覆盖 */
  height: 100%;
  background: #eee;
}
```

#### （2）清除浮动（解决高度塌陷和元素覆盖）

当父元素高度塌陷或浮动元素覆盖其他元素时，需「清除浮动」，常用 4 种方法：

| 清除方法    | 实现方式                                                     | 优点                   | 缺点                             |
| ----------- | ------------------------------------------------------------ | ---------------------- | -------------------------------- |
| 额外标签法  | 在浮动子元素最后添加一个空标签（如 `<div style="clear: both;"></div>`） | 简单易懂，兼容性好     | 增加无意义 HTML 标签，冗余       |
| 单伪元素法  | 给父元素添加类，通过 `::after` 伪元素清除浮动：<br>`.clearfix::after { content: ""; display: block; clear: both; }` | 无冗余标签，语义化好   | 仅解决高度塌陷，不解决margin塌陷 |
| 双伪元素法  | 给父元素添加类，同时处理高度塌陷和margin塌陷：<br>`.clearfix::before, .clearfix::after { content: ""; display: table; }`<br>`.clearfix::after { clear: both; }` | 兼顾多种问题，推荐使用 | 代码略多                         |
| overflow 法 | 给父元素设置 `overflow: hidden` 或 `overflow: auto`          | 代码最简单             | 可能隐藏超出父元素的内容         |

#### 推荐用法（双伪元素法）：

```CSS
/* 全局定义清除浮动类，哪里需要加哪里 */
.clearfix::before,
.clearfix::after {
  content: "";
  display: table; /* 触发 BFC，清除 margin 塌陷 */
}
.clearfix::after {
  clear: both; /* 清除浮动，解决高度塌陷 */
}
/* 兼容 IE6-7（可选，现代浏览器无需） */
.clearfix {
  *zoom: 1;
}
```

使用时给父元素添加 `clearfix` 类即可：`<div class="parent clearfix">`。

### 4.3 Flex 弹性布局（现代布局首选）

Flex（Flexible Box，弹性布局）是 CSS3 引入的布局模型，通过给父元素设置 `display: flex`，使子元素成为「弹性项」，可灵活控制子元素的排列、对齐、间距和大小分配，**不脱离文档流**，无需清除浮动，是目前最推荐的布局方案。

#### （1）Flex 核心概念

- **弹性容器（Flex Container）**：设置 `display: flex` 的父元素，负责控制弹性项的布局规则。
- **弹性项（Flex Item）**：弹性容器的直接子元素，自动成为弹性项（无需额外设置）。
- **主轴（Main Axis）**：弹性项的排列方向（默认水平方向，从左到右）。
- **侧轴（Cross Axis）**：与主轴垂直的方向（默认垂直方向，从上到下）。

#### （2）弹性容器核心属性（控制整体布局）

| 属性名            | 作用                                           | 取值范围/示例                                                |
| ----------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `display`         | 启用 Flex 布局                                 | `flex`（块级弹性容器）、`inline-flex`（行内块弹性容器）      |
| `flex-direction`  | 设置主轴方向                                   | row（默认，水平从左到右）、row-reverse（水平从右到左）、column（垂直从上到下）、column-reverse（垂直从下到上） |
| `flex-wrap`       | 弹性项是否换行                                 | nowrap（默认，不换行，挤压弹性项）、wrap（换行）、wrap-reverse（反向换行） |
| `flex-flow`       | `flex-direction` + `flex-wrap` 复合属性        | 示例：`flex-flow: row wrap`（水平排列，自动换行）            |
| `justify-content` | 弹性项在主轴上的对齐方式                       | flex-start（默认，起点对齐）、flex-end（终点对齐）、center（居中对齐）、space-around（均匀分布，项两侧间距相等）、space-between（均匀分布，项之间间距相等）、space-evenly（均匀分布，项与容器间距=项之间间距） |
| `align-items`     | 弹性项在侧轴上的对齐方式（单行）               | stretch（默认，拉伸至填满侧轴，需弹性项无侧轴尺寸）、flex-start（起点对齐）、flex-end（终点对齐）、center（居中对齐）、baseline（基线对齐，文字底部对齐） |
| `align-content`   | 弹性项多行时，行在侧轴上的对齐方式（单行无效） | 取值同 `justify-content` + stretch（默认）                   |

#### （3）弹性项核心属性（控制单个弹性项）

| 属性名        | 作用                                                | 取值范围/示例                                                |
| ------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| `flex-grow`   | 弹性项的拉伸比例（主轴剩余空间分配）                | 数字（默认 0，不拉伸）；如两个弹性项分别设为 1 和 2，则剩余空间按 1:2 分配 |
| `flex-shrink` | 弹性项的收缩比例（空间不足时压缩）                  | 数字（默认 1，允许收缩）；设为 0 则不收缩（保持原宽高）      |
| `flex-basis`  | 弹性项在主轴上的初始尺寸                            | auto（默认，随内容自适应）、像素（px）、百分比（%）          |
| `flex`        | `flex-grow` + `flex-shrink` + `flex-basis` 复合属性 | 常用值：`flex: 1`（等价于 1 1 auto，自适应剩余空间）、`flex: 0 0 200px`（固定宽度 200px，不拉伸不收缩） |
| `align-self`  | 单个弹性项的侧轴对齐方式（覆盖 `align-items`）      | 取值同 `align-items` + auto（默认，继承父容器）              |
| `order`       | 弹性项的排列顺序                                    | 数字（默认 0，越小越靠前）；可设负数（如 -1 排在最前面）     |

#### （4）Flex 布局常用示例

##### 示例 1：水平居中 + 垂直居中（单行）

```HTML
<div class="container">
  <div class="item">居中内容</div>
</div>
.container {
  width: 300px;
  height: 200px;
  border: 1px solid #ccc;
  display: flex;
  justify-content: center; /* 主轴（水平）居中 */
  align-items: center; /* 侧轴（垂直）居中 */
}
.item {
  width: 100px;
  height: 100px;
  background: red;
}
```

##### 示例 2：三栏布局（左右固定，中间自适应）

```HTML
<div class="container">
  <div class="left">左栏（200px）</div>
  <div class="middle">中间栏（自适应）</div>
  <div class="right">右栏（200px）</div>
</div>
.container {
  display: flex;
  height: 300px;
}
.left {
  width: 200px;
  background: #f5f5f5;
}
.middle {
  flex: 1; /* 自适应剩余空间 */
  background: #eee;
}
.right {
  width: 200px;
  background: #f5f5f5;
}
```

##### 示例 3：换行布局（响应式适配）

```HTML
<div class="container">
  <div class="item">1</div>
  <div class="item">2</div>
  <div class="item">3</div>
  <div class="item">4</div>
</div>
.container {
  display: flex;
  flex-wrap: wrap; /* 自动换行 */
  gap: 10px; /* 弹性项之间的间距（替代 margin） */
}
.item {
  flex: 1 1 200px; /* 最小宽度 200px，空间充足时拉伸，不足时换行 */
  height: 100px;
  background: red;
}
```

### 4.4 定位布局（Position）

定位布局通过 `position` 属性控制元素的精准位置，元素可脱离标准流（绝对定位、固定定位）或不脱离（相对定位），配合 `top`/`right`/`bottom`/`left` 边偏移实现位置控制。

#### （1）定位模式分类（4种）

| 定位模式 | 语法（`position`） | 是否脱标                       | 参考点（边偏移基准）                                         | 核心用途                                                    |
| -------- | ------------------ | ------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| 相对定位 | `relative`         | 否（占据原位置）               | 元素自身原来的位置                                           | 1. 作为绝对定位的参考父容器；2. 微调元素位置（如偏移 10px） |
| 绝对定位 | `absolute`         | 是（不占据原位置）             | 最近的已定位祖先元素（`relative`/`absolute`/`fixed`），无则参考浏览器窗口 | 精准定位元素（如弹窗、下拉菜单）                            |
| 固定定位 | `fixed`            | 是（不占据原位置）             | 浏览器窗口（视口）                                           | 固定在页面某个位置（如导航栏、回到顶部按钮）                |
| 粘性定位 | `sticky`           | 滚动前不脱标，滚动到阈值后脱标 | 浏览器窗口                                                   | 滚动时吸附效果（如列表标题、导航栏滚动固定）                |

#### （2）核心用法与示例

##### 用法 1：子绝父相（最常用）

绝对定位的子元素需配合相对定位的父元素，实现「子元素在父元素内精准定位」，避免子元素相对于浏览器窗口定位。

```HTML
<div class="parent">
  <div class="child">子元素（绝对定位）</div>
</div>
.parent {
  width: 300px;
  height: 300px;
  border: 1px solid #ccc;
  position: relative; /* 父元素相对定位，作为参考 */
}
.child {
  width: 100px;
  height: 100px;
  background: red;
  position: absolute; /* 子元素绝对定位 */
  top: 50px; /* 距离父元素顶部 50px */
  right: 50px; /* 距离父元素右侧 50px */
}
```

##### 用法 2：绝对定位居中（水平 + 垂直）

通过「边偏移 50% + 自身偏移 -50%」实现绝对定位元素居中：

```CSS
.child {
  position: absolute;
  top: 50%; /* 父元素垂直方向 50% */
  left: 50%; /* 父元素水平方向 50% */
  transform: translate(-50%, -50%); /* 自身向左、向上偏移 50%（基于自身宽高） */
}
```

优点：无需知道子元素宽高，自适应居中。

##### 用法 3：固定定位（导航栏示例）

```CSS
.nav {
  width: 100%;
  height: 60px;
  background: #333;
  position: fixed;
  top: 0; /* 固定在页面顶部 */
  left: 0;
  z-index: 999; /* 提高层级，避免被其他元素覆盖 */
}
/* 给页面主体添加 padding-top，避免被导航栏覆盖 */
body {
  padding-top: 60px;
}
```

##### 用法 4：粘性定位（滚动吸附）

```CSS
.title {
  position: sticky;
  top: 0; /* 滚动到顶部时吸附 */
  background: white;
  z-index: 100;
}
```

滚动页面时，`title` 元素会在滚动到页面顶部时固定，其他时候跟随滚动。

#### （3）堆叠层级（`z-index`）

当多个定位元素重叠时，通过 `z-index` 控制层级顺序：

- 语法：`z-index: 数字`（默认 0，支持正负整数）。
- 规则：数字越大，层级越高（越靠上）；层级相同则后定义的元素覆盖先定义的。
- 注意：`z-index` 仅对**定位元素**（`relative`/`absolute`/`fixed`/`sticky`）生效。

## 五、CSS 高级特性

### 5.1 CSS 变量（自定义属性）

CSS 变量允许定义可复用的属性值，方便全局样式统一修改，语法：

- 定义：`--变量名: 值`（需在选择器内定义，通常在 `:root` 中全局定义）。
- 使用：`var(--变量名, 默认值)`（默认值可选，变量未定义时使用）。

示例：

```CSS
:root {
  --main-color: #ff4400; /* 全局主色调变量 */
  --font-size: 16px; /* 全局字号变量 */
}
.button {
  background: var(--main-color);
  font-size: var(--font-size);
}
.card {
  border: 1px solid var(--main-color);
  font-size: calc(var(--font-size) * 1.2); /* 配合 calc 计算 */
}
```

优点：修改全局样式时，仅需修改变量值，无需逐个修改属性。

### 5.2 CSS 过渡（Transition）

过渡效果用于实现元素状态变化时的平滑动画（如鼠标悬停、点击），无需 JavaScript，语法：

- 复合属性：`transition: 过渡属性 过渡时长 过渡速度 延迟时间`。
- 核心要求：过渡属性必须有「初始值」和「目标值」（如 `hover` 前后的样式变化）。

示例：

```CSS
.button {
  width: 100px;
  height: 40px;
  background: #ff4400;
  transition: all 0.3s ease; /* 所有属性变化都有 0.3 秒过渡，缓动效果 */
}
.button:hover {
  width: 120px;
  background: #ff6600;
}
```

- 过渡属性：`all` 表示所有变化的属性，也可指定单个属性（如 `transition: width 0.3s`）。
- 过渡速度：`ease`（默认，慢-快-慢）、`linear`（匀速）、`ease-in`（慢-快）、`ease-out`（快-慢）。
- 延迟时间：如 `transition: all 0.3s ease 0.2s`（延迟 0.2 秒后开始过渡）。

### 5.3 CSS 动画（Animation）

动画比过渡更灵活，支持多关键帧、循环播放、自动触发等，语法：

1. 定义关键帧（`@keyframes`）：指定动画的不同阶段样式。
2. 给元素应用动画：通过 `animation` 属性设置。

示例：

```CSS
/* 定义关键帧（动画名称为 fadeIn） */
@keyframes fadeIn {
  0% { opacity: 0; transform: translateY(20px); } /* 初始状态：透明，下移 20px */
  100% { opacity: 1; transform: translateY(0); } /* 结束状态：不透明，回到原位 */
}
.card {
  width: 200px;
  height: 300px;
  background: white;
  /* 应用动画：名称 时长 速度 延迟 循环次数 是否反向 */
  animation: fadeIn 0.5s ease 0.2s 1 normal;
}
```

- 关键帧：可定义多个阶段（如 0%、50%、100%），50% 表示动画进行到一半时的状态。
- 动画属性：
    - `animation-name`：关键帧名称（必需）。
    - `animation-duration`：动画时长（必需，如 `0.5s`）。
    - `animation-timing-function`：过渡速度（同 `transition`）。
    - `animation-delay`：延迟时间（如 `0.2s`）。
    - `animation-iteration-count`：循环次数（`1` 次，`infinite` 无限循环）。
    - `animation-direction`：播放方向（`normal` 正向，`reverse` 反向，`alternate` 交替）。
    - `animation-fill-mode`：动画结束后保持的状态（`forwards` 保持结束状态，`backwards` 保持初始状态）。

### 5.4 CSS 精灵图（CSS Sprites）

#### （1）核心概念

将多个小图标合并为一张大图片（精灵图），通过 `background-position` 定位显示单个图标，减少 HTTP 请求次数，提升页面加载速度。

#### （2）实现步骤

1. 用工具（如 Photoshop、PxCook）合并小图标为精灵图，确保图标之间有间距（避免显示时重叠）。
2. 创建与目标图标尺寸相同的盒子。
3. 设置盒子背景图为精灵图（`background-image: url("sprites.png")`）。
4. 用 PxCook 测量目标图标的左上角相对于精灵图左上角的坐标（x, y）。
5. 设置 `background-position: -xpx -ypx`（负号表示向左/向上移动精灵图，使目标图标显示在盒子中）。

#### 示例：

精灵图中某图标左上角坐标为（50px, 30px），图标尺寸为 20px×20px：

```CSS
.icon {
  width: 20px; /* 图标宽度 */
  height: 20px; /* 图标高度 */
  background-image: url("sprites.png"); /* 精灵图路径 */
  background-repeat: no-repeat; /* 不平铺 */
  background-position: -50px -30px; /* 向左移 50px，向上移 30px */
}
```

### 5.5 字体图标（Icon Font）

#### （1）核心优势

- 本质是字体，支持 `font-size`、`color`、`text-shadow` 等 CSS 样式修改（灵活变色、放大缩小）。
- 矢量图形，放大不失真，适配各种屏幕分辨率。
- 减少 HTTP 请求（多个图标共用一个字体文件）。

#### （2）常用字体图标库

- [IconFont（阿里）](https://www.iconfont.cn/)：免费、图标丰富，支持自定义合并。
- Font Awesome：国外常用，图标风格统一。

#### （3）使用步骤（以 IconFont 为例）

1. 注册 IconFont 账号，搜索需要的图标，加入购物车后添加到项目。
2. 下载项目文件（包含 `iconfont.css`、`iconfont.ttf` 等字体文件）。
3. 将下载的文件解压到项目目录（如 `fonts/` 文件夹）。
4. 在 HTML 中通过 `link` 引入 `iconfont.css`：`<link rel="stylesheet" href="fonts/iconfont.css">`。
5. 在需要显示图标的标签中添加类名 `iconfont`（基础样式）和图标对应的类名（如 `icon-shouye`）：
    1. ```HTML
        <span class="iconfont icon-shouye"></span> <!-- 显示首页图标 -->
        ```
6. 自定义图标样式（通过 `iconfont` 类或自定义类）：
    1. ```CSS
        .iconfont {
          font-size: 24px; /* 图标大小 */
          color: #ff4400; /* 图标颜色 */
        }
        ```

#### （4）补充：上传自定义矢量图

若 IconFont 中无需要的图标，可上传 `.svg` 格式的矢量图，生成字体图标：

1. 进入 IconFont 项目，点击「上传图标」，选择本地 `.svg` 文件。
2. 图标上传成功后，刷新项目，即可像使用默认图标一样调用。

### 5.6 响应式布局基础

响应式布局使网页在不同设备（手机、平板、电脑）上自适应显示，核心技术：

#### （1）视口设置（必需）

在 HTML 的 `<head>` 中添加视口元标签，确保页面在移动设备上正常缩放：

```HTML
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

- `width=device-width`：页面宽度等于设备屏幕宽度。
- `initial-scale=1.0`：初始缩放比例为 1（不缩放）。
- 可选属性：`maximum-scale=1.0`（禁止用户放大）、`user-scalable=no`（禁止用户缩放）。

#### （2）媒体查询（Media Query）

根据设备屏幕尺寸应用不同的 CSS 样式，语法：

```CSS
/* 屏幕宽度 ≤ 768px（手机）时生效 */
@media (max-width: 768px) {
  .container {
    width: 100%;
    padding: 0 10px;
  }
  .nav {
    display: none; /* 隐藏导航栏 */
  }
}

/* 屏幕宽度 > 768px 且 ≤ 1200px（平板）时生效 */
@media (min-width: 769px) and (max-width: 1200px) {
  .container {
    width: 90%;
    margin: 0 auto;
  }
}

/* 屏幕宽度 > 1200px（电脑）时生效 */
@media (min-width: 1201px) {
  .container {
    width: 1200px;
    margin: 0 auto;
  }
}
```

#### （3）响应式单位

- `rem`：相对于根元素（`html`）的 `font-size`，如 `html { font-size: 16px; }`，则 `1rem = 16px`，移动设备可通过媒体查询修改根元素字号实现适配。
- `vw/vh`：相对于视口宽度/高度，`1vw = 视口宽度的 1%`，`1vh = 视口高度的 1%`，适合全屏布局。

## 六、CSS 常见问题与优化技巧

### 6.1 常见问题排查

1. **样式不生效**：
    1. 检查选择器是否正确（如类选择器漏写 `.`，ID 选择器漏写 `#`）。
    2. 检查优先级（是否被其他样式覆盖，可临时加 `!important` 测试）。
    3. 检查属性名和属性值是否正确（如 `font-szie` 拼写错误，`px` 漏写单位）。
2. **盒子尺寸异常**：
    1. 确认是否启用 `box-sizing: border-box`（避免 padding/border 撑大盒子）。
    2. 检查是否有默认 margin/padding（通过通配符清除）。
3. **浮动元素覆盖**：
    1. 给未浮动的元素添加 `margin` 避开浮动元素，或清除浮动。
4. **Flex 布局失效**：
    1. 确认父元素已设置 `display: flex`。
    2. 弹性项的 `float`、`clear`、`vertical-align` 属性会失效，需删除。

### 6.2 优化技巧

1. **样式复用**：
    1. 提取公共样式（如字体、颜色、边距）为类，避免重复代码。
    2. 使用 CSS 变量统一管理全局样式（如主色调、字号）。
2. **性能优化**：
    1. 合并 CSS 文件（减少 HTTP 请求）。
    2. 使用 CSS 精灵图或字体图标（减少图片请求）。
    3. 避免使用复杂选择器（如多层后代选择器，影响浏览器渲染性能）。
3. **兼容性处理**：
    1. 使用 Autoprefixer 自动添加浏览器前缀（如 `-webkit-`、`-moz-`）。
    2. 对旧浏览器（如 IE）使用媒体查询或条件注释单独适配。
4. **代码规范**：
    1. 选择器命名语义化（如 `.nav` 而非 `.red-box`）。
    2. 缩进统一（2 个或 4 个空格），属性按功能分组（如位置相关、样式相关）。
    3. 注释关键样式（如兼容处理、复杂布局逻辑）。

## 七、学习资源推荐

- **文档**：[MDN CSS 文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS)（权威、详细）。
- **工具**：
    - PxCook（测量设计图、生成 CSS 代码）。
    - Autoprefixer（自动添加浏览器前缀）。
    - IconFont（字体图标库）。
- **视频教程**：
    - B 站《黑马程序员 CSS 教程》（基础全面）。
    - B 站《Flex 布局实战》（针对性讲解现代布局）。
- **实战练习**：
    - 模仿知名网站布局（如京东、淘宝移动端）。
    - 参与 CodePen 上的 CSS 挑战（提升动手能力）。