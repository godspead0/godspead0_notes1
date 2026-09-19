# **主题**：Vue 全家桶核心笔记（包括 Vue2 的基础核心、工程化开发、组件通信、高级特性等方面）

# Vue 全家桶核心笔记（Vue2 + Vue3）

## 一、Vue2 基础核心

### 1. Vue 实例与挂载

#### 核心概念

- **挂载点（el）**：指定 Vue 管理的 DOM 元素，需是单个根元素
- **插值表达式**：`{{ 内容 }}`，支持文本渲染、表达式求值（不支持语句）
- **数据响应式**：data 中的数据变更会自动同步到视图

#### 基础语法

```HTML
<!-- HTML 结构 -->
<div id="app">
  {{ msg }} <!-- 文本渲染 -->
  {{ 1 + 1 }} <!-- 表达式求值 -->
</div>

<!-- JS 实例 -->
<script>
const app = new Vue({
  el: '#app', // 挂载到 #app 元素
  data: {
    msg: 'Hello Vue2' // 响应式数据
  }
})

// 修改数据（视图自动更新）
app.msg = 'Hello Vue2 Updated'
</script>
```

### 2. Vue 核心指令（14个）

| 指令        | 作用                  | 语法示例                                                     | 备注                                      |
| ----------- | --------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| `v-html`    | 渲染 HTML 标签        | `<div v-html="htmlStr"></div>`                               | 防止 XSS 攻击                             |
| `v-text`    | 渲染纯文本            | `<div v-text="msg"></div>`                                   | 等价于 `{{ msg }}`，无闪烁问题            |
| `v-bind`    | 绑定元素属性          | `<img :src="imgUrl">`                                        | 缩写 `:`，支持动态属性值                  |
| `v-on`      | 绑定事件              | `<button @click="handleClick">`                              | 缩写 `@`，支持事件修饰符                  |
| `v-model`   | 双向绑定              | `<input v-model="username">`                                 | 仅用于表单元素                            |
| `v-for`     | 遍历渲染              | `<li v-for="(item, index) in list" :key="item.id">{{ item }}</li>` | `key` 必须唯一，优化渲染性能              |
| `v-if`      | 条件渲染（销毁/创建） | `<div v-if="isShow">显示</div>`                              | 切换开销大，适合低频切换                  |
| `v-else`    | 配合 v-if             | `<div v-else>隐藏</div>`                                     | 必须紧接 v-if 之后                        |
| `v-else-if` | 多条件判断            | `<div v-else-if="status === 'loading'">加载中</div>`         | -                                         |
| `v-show`    | 条件渲染（显示/隐藏） | `<div v-show="isShow">显示</div>`                            | 本质是 `display: none`，切换开销小        |
| `v-cloak`   | 隐藏未编译模板        | `<div v-cloak>{{ msg }}</div>`                               | 需配合 CSS：`[v-cloak] { display: none }` |
| `v-pre`     | 跳过编译              | `<div v-pre>{{ msg }}</div>`                                 | 直接显示 `{{ msg }}`，不解析              |
| `v-once`    | 仅渲染一次            | `<div v-once>{{ msg }}</div>`                                | 数据变更不更新视图                        |
| `v-slot`    | 插槽占位              | `<slot name="header"></slot>`                                | Vue2.6+ 新增，替代 `slot` 属性            |

### 3. 指令修饰符

#### （1）按键修饰符

```HTML
<!-- 按下回车键触发 -->
<input @keyup.enter="handleSubmit">
<!-- 支持：.enter/.tab/.delete/.esc/.space 等 -->
```

#### （2）v-model 修饰符

```HTML
<!-- 去除首尾空格 -->
<input v-model.trim="username">
<!-- 转为数字类型 -->
<input v-model.number="age">
<!-- 失去焦点才更新 -->
<input v-model.lazy="content">
```

#### （3）事件修饰符

```HTML
<!-- 阻止事件冒泡 -->
<button @click.stop="handleClick">点击</button>
<!-- 阻止默认行为 -->
<a @click.prevent="handleLink">链接</a>
<!-- 事件只触发一次 -->
<button @click.once="handleOnce">一次点击</button>
<!-- 事件捕获模式 -->
<div @click.capture="handleCapture">父元素</div>
```

### 4. 样式绑定增强（v-bind:class）

#### （1）对象语法（条件切换类）

```HTML
<div :class="{ active: isActive, 'text-red': isRed }"></div>
<!-- isActive 为 true 时添加 active 类 -->
```

#### （2）数组语法（批量添加类）

```HTML
<div :class="[ 'class1', 'class2', activeClass ]"></div>
<!-- 直接添加数组中的所有类 -->
```

#### （3）样式绑定（:style）

```HTML
<div :style="{ color: textColor, fontSize: fontSize + 'px' }"></div>
<!-- 支持对象/数组格式 -->
```

### 5. 计算属性（computed）

#### 核心特性

- 基于依赖数据缓存，依赖不变则不会重新计算
- 支持 getter（读取）和 setter（修改）
- 优先于 methods 用于数据加工（性能更优）

#### 语法示例

```JavaScript
new Vue({
  data: {
    firstName: '张',
    lastName: '三'
  },
  computed: {
    // 简化写法（仅 getter）
    fullName() {
      return this.firstName + this.lastName
    },
    // 完整写法（getter + setter）
    fullName: {
      get() {
        return this.firstName + this.lastName
      },
      set(newValue) {
        const [first, last] = newValue.split(' ')
        this.firstName = first
        this.lastName = last
      }
    }
  }
})
```

### 6. 监听器（watch）

#### 核心特性

- 监听数据变化并执行副作用操作（如异步请求、日志打印）
- 支持深度监听、立即执行、特殊字符属性监听

#### 语法示例

```JavaScript
new Vue({
  data: {
    words: '苹果',
    obj: { words: '香蕉' },
    user: { name: '张三', age: 20 }
  },
  watch: {
    // 基础类型监听
    words(newVal, oldVal) {
      console.log('words变化：', newVal, oldVal)
    },
    // 对象属性监听（需加引号）
    'obj.words'(newVal) {
      console.log('obj.words变化：', newVal)
    },
    // 深度监听（对象内部属性变化）
    user: {
      deep: true, // 开启深度监听
      immediate: true, // 初始化时立即执行
      handler(newVal) {
        console.log('user变化：', newVal)
      }
    }
  }
})
```

## 二、Vue2 工程化开发

### 1. 核心目录结构

```Plain
src/
├── main.js        # 入口文件（创建 Vue 实例）
├── App.vue        # 根组件（页面入口）
├── components/    # 复用组件（子组件）
├── views/         # 页面组件（路由对应页面）
└── utils/         # 工具函数（如 eventBus）
public/
└── index.html     # 模板文件（挂载点）
```

### 2. 组件基础

#### （1）组件结构（.vue 文件）

```Plain
<!-- 模板部分（必须有且仅有一个根元素） -->
<template>
  <div class="component">
    {{ msg }}
  </div>
</template>

<!-- 逻辑部分 -->
<script>
export default {
  name: 'MyComponent', // 组件名（ PascalCase 规范）
  data() {
    // 组件的 data 必须是函数（保证数据独立性）
    return {
      msg: '组件内容'
    }
  }
}
</script>

<!-- 样式部分 -->
<style scoped>
/* scoped：样式仅作用于当前组件（防止冲突） */
.component {
  color: red;
}
</style>
```

#### （2）组件注册

##### 局部注册（仅当前组件可用）

```Plain
<!-- 父组件中 -->
<template>
  <MyComponent />
</template>

<script>
import MyComponent from './components/MyComponent.vue'

export default {
  components: {
    MyComponent // 注册组件（键值同名可简写）
  }
}
</script>
```

##### 全局注册（所有组件可用）

```JavaScript
// main.js 中
import Vue from 'vue'
import MyComponent from './components/MyComponent.vue'

Vue.component('MyComponent', MyComponent) // 全局注册
```

#### （3）样式隔离（scoped 原理）

- 给当前组件所有 DOM 元素添加自定义属性 `data-v-hash`
- 样式自动转换为 `选择器[data-v-hash]`，仅匹配当前组件元素

### 3. 组件通信

#### （1）父子通信

##### 父传子（props）

1. 父组件通过属性传递数据
2. 子组件通过 `props` 接收数据（支持校验）
3. 遵循**单向数据流**：子组件不能直接修改 props

```Plain
<!-- 父组件 -->
<template>
  <Son :title="parentTitle" :count="10" />
</template>

<script>
export default {
  data() {
    return { parentTitle: '父组件标题' }
  }
}
</script>

<!-- 子组件 -->
<template>
  <div>{{ title }} - {{ count }}</div>
</template>

<script>
export default {
  // props 校验
  props: {
    title: {
      type: String, // 类型
      required: true, // 必传
      default: '默认标题' // 默认值（非必传时生效）
    },
    count: {
      type: Number,
      validator: (value) => {
        // 自定义校验：值必须大于 0
        return value > 0
      }
    }
  }
}
</script>
```

##### 子传父（$emit）

1. 子组件通过 `this.$emit('事件名', 数据)` 触发自定义事件
2. 父组件通过 `@事件名` 监听并接收数据

```Plain
<!-- 子组件 -->
<template>
  <button @click="sendMsg">向父组件传值</button>
</template>

<script>
export default {
  data() {
    return { childMsg: '子组件消息' }
  },
  methods: {
    sendMsg() {
      // 触发自定义事件，传递数据
      this.$emit('child-msg', this.childMsg)
    }
  }
}
</script>

<!-- 父组件 -->
<template>
  <Son @child-msg="getChileMsg" />
</template>

<script>
export default {
  methods: {
    getChileMsg(data) {
      console.log('接收子组件数据：', data)
    }
  }
}
</script>
```

#### （2）非父子通信

##### 方案1：EventBus（事件总线）

适用于中小型项目，跨组件简单通信

1. 创建事件总线实例

```JavaScript
// src/utils/eventBus.js
import Vue from 'vue'
export default new Vue() // 空 Vue 实例作为总线
```

1. 发送方触发事件

```JavaScript
import bus from '@/utils/eventBus'

// 触发事件并传递数据
bus.$emit('custom-event', data)
```

1. 接收方监听事件

```JavaScript
import bus from '@/utils/eventBus'

export default {
  created() {
    // 监听事件
    bus.$on('custom-event', (data) => {
      console.log('接收数据：', data)
    })
  },
  beforeDestroy() {
    // 销毁时移除监听（防止内存泄漏）
    bus.$off('custom-event')
  }
}
```

##### 方案2：Provide & Inject（跨层级）

适用于祖孙组件通信，无需逐层传递

```Plain
<!-- 顶层组件（提供数据） -->
<script>
export default {
  provide() {
    return {
      globalName: '全局数据' // 提供给所有子组件
    }
  }
}
</script>

<!-- 底层组件（注入数据） -->
<script>
export default {
  inject: ['globalName'], // 注入顶层数据
  mounted() {
    console.log(this.globalName) // 访问全局数据
  }
}
</script>
```

##### 方案3：Vuex（全局状态管理）

适用于大型项目，统一管理全局数据（见下文详细说明）

### 4. 高级特性

#### （1）v-model 本质

`v-model` 是 `:value` + `@input` 的语法糖，支持自定义组件双向绑定

```HTML
<!-- 原生表单 -->
<input v-model="msg" />
<!-- 等价于 -->
<input :value="msg" @input="msg = $event.target.value" />

<!-- 自定义组件 -->
<CustomInput v-model="msg" />
<!-- 等价于 -->
<CustomInput :value="msg" @input="msg = $event" />
```

#### （2）.sync 修饰符（双向绑定简化）

适用于父子组件属性双向同步（替代 v-model 自定义）

```Plain
<!-- 父组件 -->
<BaseDialog :visible.sync="dialogVisible" />
<!-- 等价于 -->
<BaseDialog :visible="dialogVisible" @update:visible="dialogVisible = $event" />

<!-- 子组件 -->
<script>
export default {
  props: { visible: Boolean },
  methods: {
    closeDialog() {
      // 触发更新事件
      this.$emit('update:visible', false)
    }
  }
}
</script>
```

#### （3）ref 与 $refs（DOM/组件引用）

- 访问 DOM 元素：给 DOM 加 `ref` 属性，通过 `this.$refs.refName` 获取
- 访问组件实例：给组件加 `ref` 属性，可调用组件方法/访问属性

```Plain
<template>
  <div ref="domRef">DOM 元素</div>
  <ChildComponent ref="compRef" />
</template>

<script>
export default {
  mounted() {
    // 访问 DOM 元素
    console.log(this.$refs.domRef)
    // 访问组件实例并调用方法
    this.$refs.compRef.childMethod()
  }
}
</script>
```

#### （4）$nextTick（异步更新）

解决 Vue 异步 DOM 更新导致的视图未同步问题

```JavaScript
export default {
  methods: {
    updateDom() {
      this.msg = '更新后的值'
      // DOM 未立即更新，直接操作会失败
      console.log(this.$refs.domRef.textContent) // 旧值
      
      // 等待 DOM 更新完成后执行
      this.$nextTick(() => {
        console.log(this.$refs.domRef.textContent) // 新值
      })
    }
  }
}
```

#### （5）自定义指令

用于对 DOM 元素进行底层操作（如权限控制、输入限制）

##### 全局指令（main.js）

```JavaScript
Vue.directive('focus', {
  // 元素插入页面时触发
  inserted(el) {
    el.focus() // 自动聚焦
  }
})

// 使用：<input v-focus />
```

##### 局部指令（组件内）

```Plain
<script>
export default {
  directives: {
    color: {
      inserted(el, binding) {
        // binding.value 是指令参数
        el.style.color = binding.value
      }
    }
  }
}
</script>

<!-- 使用：<div v-color="red">红色文本</div> -->
```

#### （6）插槽（Slot）

让组件结构支持自定义，实现组件复用与灵活扩展

##### 1. 默认插槽（单插槽）

```Plain
<!-- 子组件（SlotDemo.vue） -->
<template>
  <div>
    <h3>组件标题</h3>
    <!-- 插槽占位，默认内容（不传值时显示） -->
    <slot>默认内容</slot>
  </div>
</template>

<!-- 父组件使用 -->
<template>
  <SlotDemo>
    <p>自定义插槽内容</p> <!-- 替换默认内容 -->
  </SlotDemo>
</template>
```

##### 2. 具名插槽（多插槽）

用于多个自定义区域，通过 `name` 区分

```Plain
<!-- 子组件 -->
<template>
  <div>
    <slot name="header"></slot> <!-- 头部插槽 -->
    <slot></slot> <!-- 默认插槽（name="default"） -->
    <slot name="footer"></slot> <!-- 底部插槽 -->
  </div>
</template>

<!-- 父组件使用 -->
<template>
  <SlotDemo>
    <template #header> <!-- 缩写：v-slot:header → #header -->
      <h1>页面标题</h1>
    </template>
    <p>主体内容</p> <!-- 默认插槽 -->
    <template #footer>
      <p>页脚信息</p>
    </template>
  </SlotDemo>
</template>
```

##### 3. 作用域插槽（传递数据给父组件）

子组件向父组件传递数据，父组件自定义渲染逻辑

```Plain
<!-- 子组件 -->
<template>
  <div>
    <!-- 向父组件传递数据：id 和 name -->
    <slot :user="user" :id="100"></slot>
  </div>
</template>

<script>
export default {
  data() {
    return { user: { name: '张三' } }
  }
}
</script>

<!-- 父组件使用 -->
<template>
  <SlotDemo>
    <!-- 接收子组件传递的数据（obj 是自定义变量名） -->
    <template v-slot:default="obj">
      {{ obj.id }} - {{ obj.user.name }}
    </template>
  </SlotDemo>
</template>
```

## 三、Vue2 路由（VueRouter）

### 1. 核心概念

- 单页应用（SPA）：通过路由切换组件，不刷新页面
- 路由映射：URL 路径与组件的对应关系
- 路由出口：`router-view`，用于渲染匹配的组件

### 2. 基础使用

#### （1）安装与配置

```Bash
npm install vue-router@3.x # Vue2 对应 router 3.x 版本
// src/router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '@/views/Home.vue'
import About from '@/views/About.vue'

Vue.use(VueRouter) // 注册路由插件

// 路由规则
const routes = [
  { path: '/', redirect: '/home' }, // 默认跳转
  { path: '/home', component: Home, name: 'Home' }, // 命名路由
  { path: '/about', component: About },
  { path: '*', component: () => import('@/views/404.vue') } // 404 页面（需放最后）
]

// 创建路由实例
const router = new VueRouter({
  mode: 'history', // 路由模式：hash（默认，带 #）/ history（不带 #）
  routes,
  // 自定义激活类名（替代默认的 router-link-active）
  linkActiveClass: 'active',
  linkExactActiveClass: 'exact-active'
})

export default router
// main.js 挂载路由
import Vue from 'vue'
import App from './App.vue'
import router from './router'

new Vue({
  router, // 注入路由
  render: h => h(App)
}).$mount('#app')
```

#### （2）路由跳转

##### 声明式跳转（router-link）

```Plain
<template>
  <div>
    <!-- 普通跳转 -->
    <router-link to="/home">首页</router-link>
    <!-- 命名路由跳转 -->
    <router-link :to="{ name: 'Home' }">首页</router-link>
    <!-- 路由出口（渲染匹配的组件） -->
    <router-view></router-view>
  </div>
</template>
```

##### 编程式跳转（$router）

```JavaScript
export default {
  methods: {
    goHome() {
      // 1. 路径跳转
      this.$router.push('/home')
      // 2. 命名路由跳转
      this.$router.push({ name: 'Home' })
      // 3. 后退
      this.$router.go(-1)
      // 4. 替换当前历史记录（不添加新记录）
      this.$router.replace('/about')
    }
  }
}
```

#### （3）路由传参

##### 1. Query 参数（URL 拼接：?key=value）

```JavaScript
// 跳转传参
this.$router.push({
  path: '/home',
  query: { id: 1, name: '张三' }
})

// 接收参数
this.$route.query.id // 1
this.$route.query.name // 张三
```

##### 2. Params 参数（动态路由：/path/value）

```JavaScript
// 1. 配置动态路由
const routes = [
  { path: '/home/:id/:name', component: Home, name: 'Home' }
]

// 2. 跳转传参
this.$router.push({
  name: 'Home', // params 必须配合命名路由
  params: { id: 1, name: '张三' }
})

// 3. 接收参数
this.$route.params.id // 1
this.$route.params.name // 张三
```

## 四、Vue2 状态管理（Vuex）

### 1. 核心概念

- 全局状态管理：集中管理所有组件的共用数据
- 单向数据流：State → View → Actions → Mutations → State
- 五大核心：State、Mutations、Actions、Getters、Modules

### 2. 基础使用

#### （1）安装与配置

```Bash
npm install vuex@3.x # Vue2 对应 vuex 3.x 版本
// src/store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  // 1. 状态数据（类似组件的 data）
  state: {
    count: 0,
    userInfo: null
  },

  // 2. 同步修改状态（唯一修改 state 的方式）
  mutations: {
    increment(state, num = 1) {
      state.count += num
    },
    setUserInfo(state, user) {
      state.userInfo = user
    }
  },

  // 3. 异步操作（类似组件的 methods，可调用 mutations）
  actions: {
    // 异步修改 count
    asyncIncrement({ commit }, num) {
      setTimeout(() => {
        commit('increment', num) // 调用 mutation
      }, 1000)
    },
    // 异步获取用户信息
    fetchUserInfo({ commit }) {
      return new Promise((resolve) => {
        setTimeout(() => {
          const user = { id: 1, name: '张三' }
          commit('setUserInfo', user)
          resolve(user)
        }, 500)
      })
    }
  },

  // 4. 数据加工（类似组件的 computed，基于 state 派生）
  getters: {
    doubleCount(state) {
      return state.count * 2
    },
    isLogin(state) {
      return !!state.userInfo
    }
  },

  // 5. 模块化（分割复杂状态）
  modules: {
    // 示例：用户模块
    user: {
      namespaced: true, // 开启命名空间（避免命名冲突）
      state: { token: '' },
      mutations: {
        setToken(state, token) {
          state.token = token
        }
      }
    }
  }
})
// main.js 挂载 Vuex
import Vue from 'vue'
import App from './App.vue'
import store from './store'

new Vue({
  store, // 注入 Vuex
  render: h => h(App)
}).$mount('#app')
```

#### （2）组件中使用 Vuex

##### 1. 访问 State

```JavaScript
// 方式1：直接访问
this.$store.state.count
this.$store.state.user.token // 模块化访问

// 方式2：辅助函数 mapState（简化代码）
import { mapState } from 'vuex'

export default {
  computed: {
    ...mapState(['count']), // 映射根模块 state
    ...mapState('user', ['token']) // 映射 user 模块 state
  }
}
```

##### 2. 调用 Mutations

```JavaScript
// 方式1：直接调用
this.$store.commit('increment', 2)
this.$store.commit('user/setToken', 'xxx') // 模块化调用

// 方式2：辅助函数 mapMutations
import { mapMutations } from 'vuex'

export default {
  methods: {
    ...mapMutations(['increment']),
    ...mapMutations('user', ['setToken']),
    handleClick() {
      this.increment(2)
      this.setToken('xxx')
    }
  }
}
```

##### 3. 调用 Actions

```JavaScript
// 方式1：直接调用
this.$store.dispatch('asyncIncrement', 2)
this.$store.dispatch('user/fetchUserInfo')

// 方式2：辅助函数 mapActions
import { mapActions } from 'vuex'

export default {
  methods: {
    ...mapActions(['asyncIncrement']),
    async handleAsync() {
      await this.asyncIncrement(2)
    }
  }
}
```

##### 4. 访问 Getters

```JavaScript
// 方式1：直接访问
this.$store.getters.doubleCount
this.$store.getters['user/isLogin']

// 方式2：辅助函数 mapGetters
import { mapGetters } from 'vuex'

export default {
  computed: {
    ...mapGetters(['doubleCount']),
    ...mapGetters('user', ['isLogin'])
  }
}
```

### 3. 严格模式

开启严格模式后，直接修改 state 会报错（保证状态变更可追踪）

```JavaScript
new Vuex.Store({
  strict: process.env.NODE_ENV !== 'production', // 生产环境关闭
  // ...其他配置
})
```

## 五、Vue3 核心变化

### 1. 工程化差异

| 特性     | Vue2                | Vue3                            |
| -------- | ------------------- | ------------------------------- |
| 构建工具 | Webpack（配置复杂） | Vite（快速热更新）              |
| 核心依赖 | vue@2.x             | vue@3.x                         |
| 入口文件 | new Vue()           | createApp()                     |
| 根组件   | 必须有唯一根元素    | 支持多根元素                    |
| 样式隔离 | scoped              | scoped + CSS Modules + CSS 变量 |

### 2. 入口文件示例（main.js）

```JavaScript
// Vue2
import Vue from 'vue'
import App from './App.vue'
new Vue({ render: h => h(App) }).$mount('#app')

// Vue3
import { createApp } from 'vue'
import App from './App.vue'
createApp(App).mount('#app')
```

### 3. 组件语法（`<script setup>` 语法糖）

#### 核心优势

- 无需 `export default`，自动导出
- 无需注册组件（直接导入使用）
- 支持顶层 await
- 语法更简洁，开发效率更高

#### 基础示例

```Plain
<template>
  <div>
    <h1>{{ msg }}</h1>
    <button @click="handleClick">点击</button>
    <ChildComponent /> <!-- 无需注册 -->
  </div>
</template>

<!-- 脚本部分（setup 语法糖） -->
<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue' // 直接导入

// 响应式数据（ref 用于简单类型）
const msg = ref('Hello Vue3')

// 函数
const handleClick = () => {
  msg.value = 'Vue3 真好用' // ref 需通过 .value 访问
}
</script>

<style scoped>
h1 {
  color: blue;
}
</style>
```

### 4. 响应式 API（Composition API）

Vue3 新增的组合式 API，替代 Vue2 的 Options API，更灵活的代码组织

#### （1）ref 与 reactive

- `ref`：用于简单类型（Number、String、Boolean），返回响应式对象，通过 `.value` 访问
- `reactive`：用于复杂类型（Object、Array），返回响应式代理对象，直接访问属性

```JavaScript
import { ref, reactive } from 'vue'

// 简单类型响应式
const count = ref(0)
count.value++ // 修改值

// 复杂类型响应式
const user = reactive({
  name: '张三',
  age: 20
})
user.age = 21 // 直接修改属性
```

#### （2）computed

```JavaScript
import { ref, computed } from 'vue'

const count = ref(0)
// 只读计算属性
const doubleCount = computed(() => count.value * 2)
// 可写计算属性
const fullName = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(newVal) {
    const [first, last] = newVal.split(' ')
    firstName.value = first
    lastName.value = last
  }
})
```

#### （3）watch

```JavaScript
import { ref, watch } from 'vue'

const count = ref(0)
const user = reactive({ name: '张三' })

// 监听单个 ref
watch(count, (newVal, oldVal) => {
  console.log('count变化：', newVal, oldVal)
}, {
  immediate: true, // 初始化立即执行
  deep: false // 简单类型无需深度监听
})

// 监听 reactive 对象属性
watch(
  () => user.name, // 监听具体属性
  (newVal) => {
    console.log('name变化：', newVal)
  }
)

// 监听多个数据
watch([count, () => user.name], ([newCount, newName]) => {
  console.log('count或name变化', newCount, newName)
})
```

#### （4）生命周期函数

Vue3 生命周期函数前缀加 `on`，且需导入使用，部分函数名变更

| Vue2          | Vue3（Composition API） | 作用       |
| ------------- | ----------------------- | ---------- |
| beforeCreate  | -（setup 中替代）       | 组件创建前 |
| created       | -（setup 中替代）       | 组件创建后 |
| beforeMount   | onBeforeMount           | 挂载前     |
| mounted       | onMounted               | 挂载后     |
| beforeUpdate  | onBeforeUpdate          | 更新前     |
| updated       | onUpdated               | 更新后     |
| beforeDestroy | onBeforeUnmount         | 卸载前     |
| destroyed     | onUnmounted             | 卸载后     |

```JavaScript
import { onMounted, onUnmounted } from 'vue'

onMounted(() => {
  console.log('组件挂载完成')
})

onUnmounted(() => {
  console.log('组件卸载完成')
})
```

### 5. Vue3 组件通信

#### （1）父传子（defineProps）

```Plain
<!-- 父组件 -->
<template>
  <Child :title="parentTitle" :count="10" />
</template>

<script setup>
import { ref } from 'vue'
import Child from './Child.vue'

const parentTitle = ref('父组件标题')
</script>

<!-- 子组件 -->
<template>
  <div>{{ title }} - {{ count }}</div>
</template>

<script setup>
// 定义 props 并校验
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  count: {
    type: Number,
    default: 0
  }
})

// 访问 props
console.log(props.title)
</script>
```

#### （2）子传父（defineEmits）

```Plain
<!-- 父组件 -->
<template>
  <Child @child-msg="getChileMsg" />
</template>

<script setup>
import Child from './Child.vue'

const getChileMsg = (data) => {
  console.log('接收子组件数据：', data)
}
</script>

<!-- 子组件 -->
<template>
  <button @click="sendMsg">传值给父组件</button>
</template>

<script setup>
import { ref } from 'vue'

// 定义自定义事件
const emit = defineEmits(['child-msg'])
const childMsg = ref('子组件消息')

const sendMsg = () => {
  // 触发事件并传递数据
  emit('child-msg', childMsg.value)
}
</script>
```

#### （3）跨层级通信（provide / inject）

```Plain
<!-- 顶层组件 -->
<script setup>
import { ref, provide } from 'vue'

const globalCount = ref(0)
// 提供数据（支持响应式）
provide('globalCount', globalCount)

// 提供修改方法
const incrementGlobal = () => {
  globalCount.value++
}
provide('incrementGlobal', incrementGlobal)
</script>

<!-- 底层组件 -->
<script setup>
import { inject } from 'vue'

// 注入数据
const globalCount = inject('globalCount')
const incrementGlobal = inject('incrementGlobal')
</script>
```

#### （4）模板引用（defineExpose）

Vue3 中组件的属性默认不对外开放，需通过 `defineExpose` 暴露

```Plain
<!-- 子组件 -->
<script setup>
import { ref } from 'vue'

const childCount = ref(0)
const childMethod = () => {
  childCount.value++
}

// 暴露属性和方法
defineExpose({
  childCount,
  childMethod
})
</script>

<!-- 父组件 -->
<template>
  <Child ref="childRef" />
  <button @click="callChildMethod">调用子组件方法</button>
</template>

<script setup>
import { ref } from 'vue'
import Child from './Child.vue'

const childRef = ref(null)

const callChildMethod = () => {
  // 访问子组件暴露的属性和方法
  console.log(childRef.value.childCount)
  childRef.value.childMethod()
}
</script>
```

## 六、模拟接口（json-server）

当后端接口未完成时，使用 json-server 快速模拟 RESTful API

### 1. 安装与使用

```Bash
# 全局安装
npm install -g json-server

# 创建模拟数据文件（db.json）
{
  "users": [
    { "id": 1, "name": "张三", "age": 20 },
    { "id": 2, "name": "李四", "age": 22 }
  ]
}

# 启动服务
json-server --watch db.json --port 3000
```

### 2. 访问接口

- GET：`http://localhost:3000/users`（获取所有用户）
- GET：`http://localhost:3000/users/1`（获取单个用户）
- POST：`http://localhost:3000/users`（新增用户，需传 body）
- PUT：`http://localhost:3000/users/1`（修改用户）
- DELETE：`http://localhost:3000/users/1`（删除用户）

## 七、核心差异总结（Vue2 vs Vue3）

| 特性       | Vue2                               | Vue3                                           |
| ---------- | ---------------------------------- | ---------------------------------------------- |
| 核心 API   | Options API（选项式）              | Composition API（组合式）+ Options API         |
| 响应式原理 | Object.defineProperty              | Proxy（支持更多类型，性能更优）                |
| 组件语法   | 需 `export default`，单一根元素    | `<script setup>` 语法糖，多根元素              |
| 状态管理   | Vuex@3.x                           | Pinia（官方推荐，替代 Vuex）                   |
| 路由       | VueRouter@3.x                      | VueRouter@4.x                                  |
| 构建工具   | Webpack                            | Vite（默认）                                   |
| 生命周期   | 选项式（beforeCreate、created 等） | 组合式（onMounted、onUnmounted 等）            |
| 组件通信   | props/$emit、EventBus、Vuex        | defineProps/defineEmits、provide/inject、Pinia |

## 八、扩展建议

1. **Vue3 进阶**：学习 Pinia（状态管理）、VueRouter4、Teleport（ teleport 组件）、Suspense（异步组件）
2. **工程化**：学习 Vue CLI 配置、Vite 优化、ESLint + Prettier 规范
3. **UI 框架**：Element UI（Vue2）、Element Plus（Vue3）、Vuetify、Ant Design Vue
4. **实战技巧**：组件封装、权限控制、请求封装（Axios）、懒加载、性能优化