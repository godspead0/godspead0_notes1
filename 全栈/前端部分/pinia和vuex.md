# Pinia 与 Vuex 核心用法对比及注意事项

# Pinia 核心用法笔记（附与 Vuex 对比）

## 一、Pinia 简介

Pinia 是 Vue 官方推荐的**状态管理库**，用于替代 Vuex（Vuex 4 已停止主动开发），适用于 Vue 2/Vue 3 项目。其设计理念更简洁、API 更友好，原生支持 TypeScript，且去除了 Vuex 中冗余的概念（如 mutations、modules 嵌套等）。

核心优势：

- 语法简洁，无冗余概念（无需 mutations、namespace）
- 原生支持 Vue 3 的 Composition API 和 Vue 2 的 Options API
- 自动支持 TypeScript 类型推导
- 支持异步 actions（无需额外配置）
- 响应式机制优化，解构时可通过工具保持响应式
- 无需嵌套 modules，天然支持多 store 拆分

## 二、Pinia 核心用法

### 1. 安装与基础导入

```Bash
# 安装
npm install pinia
# 或 yarn add pinia
```

在 Vue 项目入口（如 `main.js`）注册 Pinia：

```JavaScript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
app.use(createPinia()) // 注册 Pinia
app.mount('#app')
```

### 2. 定义 Store（核心：`defineStore`）

使用 `defineStore` 函数创建 Store，核心参数：

- 第一个参数：Store 唯一标识（字符串，类似 Vuex 的 `namespace`）
- 第二个参数：配置对象（包含 `state`、`getters`、`actions`）或函数（Composition API 风格）

#### 方式 1：Options API 风格（类似 Vuex 配置）

```JavaScript
// src/stores/counter.js
import { defineStore } from 'pinia'

// 定义并导出 Store，返回一个函数（命名规范：useXxxStore）
export const useCounterStore = defineStore('counter', {
  // 状态：存储原始数据（必须是函数，返回对象，确保组件实例隔离）
  state: () => ({
    count: 0,
    user: { name: '张三', age: 20 },
    list: [1, 2, 3]
  }),

  // 计算属性：派生状态（类似 Vue 的 computed）
  getters: {
    // 1. 基础用法：接收 state 作为参数
    doubleCount: (state) => state.count * 2,
    // 2. 依赖其他 getters：直接通过 this 访问
    doubleCountPlusOne: function () {
      return this.doubleCount + 1 // 这里的 this 指向 state
    },
    // 3. 带参数的 getters（本质返回函数）
    findItem: (state) => (id) => state.list.find(item => item === id)
  },

  // 方法：修改状态（支持同步 + 异步，类似 Vuex 的 actions，但无需 mutations）
  actions: {
    // 同步操作
    increment(num = 1) {
      this.count += num // 直接通过 this 访问 state 数据（无需传入 state）
    },
    // 异步操作（原生支持，无需额外配置）
    async fetchUser() {
      const res = await fetch('/api/user')
      const data = await res.json()
      this.user = data // 异步请求后直接修改状态
    },
    // 调用其他 actions
    incrementAndFetch() {
      this.increment() // 调用自身 actions
      this.fetchUser() // 调用异步 actions
    }
  }
})
```

#### 方式 2：Composition API 风格（更灵活）

适合习惯 Vue 3 Composition API 的场景，直接使用 `ref`/`reactive` 定义状态：

```JavaScript
// src/stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed, onMounted } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // 状态：用 ref/reactive 定义（天然响应式）
  const count = ref(0)
  const user = ref({ name: '张三', age: 20 })
  const list = ref([1, 2, 3])

  // 计算属性：用 computed 定义
  const doubleCount = computed(() => count.value * 2)
  const doubleCountPlusOne = computed(() => doubleCount.value + 1)
  const findItem = (id) => list.value.find(item => item === id)

  // 方法：直接定义函数（支持同步 + 异步）
  const increment = (num = 1) => {
    count.value += num
  }
  const fetchUser = async () => {
    const res = await fetch('/api/user')
    const data = await res.json()
    user.value = data
  }

  // 生命周期钩子（可直接使用 Vue 的组合式钩子）
  onMounted(() => {
    fetchUser() // 组件挂载时执行异步请求
  })

  // 暴露给外部的属性和方法（必须返回）
  return {
    count,
    user,
    list,
    doubleCount,
    doubleCountPlusOne,
    findItem,
    increment,
    fetchUser
  }
})
```

### 3. 在组件中使用 Store

```Plain
<!-- 组件中使用 -->
<template>
  <div>
    <p>count: {{ counterStore.count }}</p>
    <p>doubleCount: {{ counterStore.doubleCount }}</p>
    <button @click="counterStore.increment()">+1</button>
    <button @click="counterStore.fetchUser()">获取用户</button>
  </div>
</template>

<script setup>
// 导入定义好的 Store 函数
import { useCounterStore } from '@/stores/counter'

// 调用函数，获取 Store 实例（全局唯一，多次调用返回同一个实例）
const counterStore = useCounterStore()

// 注意：直接解构会丢失响应式！
// const { count, increment } = useCounterStore() // 错误：count 不再是响应式

// 保持响应式的解构方式：使用 storeToRefs（仅解构 state/getters）
import { storeToRefs } from 'pinia'
const { count, doubleCount } = storeToRefs(counterStore) // 响应式保留
const { increment, fetchUser } = counterStore // actions 可直接解构（本身是函数）
</script>
```

### 4. 状态修改的其他方式

#### （1）直接修改状态（简单场景）

```JavaScript
const counterStore = useCounterStore()
counterStore.count = 10 // 直接修改 state 数据（支持，但大型项目建议用 actions 统一管理）
```

#### （2）批量修改状态（`$patch` 方法）

适合一次性修改多个状态，比多次直接修改更高效：

```JavaScript
// 方式 1：对象形式
counterStore.$patch({
  count: 10,
  user: { name: '李四' } // 注意：会覆盖整个对象，需手动合并属性
})

// 方式 2：函数形式（支持复杂逻辑，推荐）
counterStore.$patch((state) => {
  state.count += 10
  state.list.push(4)
  state.user.name = '李四' // 仅修改对象的某个属性
})
```

#### （3）重置状态（`$reset` 方法）

仅 Options API 风格支持，将 state 恢复为初始值：

```JavaScript
counterStore.$reset() // count 回到 0，user 回到初始值
```

## 三、Pinia 与 Vuex 核心对比

| 特性            | Pinia                                                        | Vuex（3.x/4.x）                                              |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 核心概念        | State、Getters、Actions（无 Mutations、Modules）             | State、Getters、Mutations、Actions、Modules、Namespace       |
| 状态修改        | 1. Actions 直接修改（同步/异步）<br>2. 支持直接修改<br>3. `$patch` 批量修改 | 1. 必须通过 Mutations 同步修改<br>2. Actions 需提交 Mutations（异步逻辑在 Actions） |
| 代码冗余度      | 极低（无 Mutations 模板代码）                                | 较高（Mutations 必须同步，需写大量 `commit`）                |
| 响应式解构      | 支持 `storeToRefs` 轻松解构                                  | 需使用 `mapState`/`mapGetters` 或 `toRefs` 手动处理          |
| 异步支持        | 原生支持（Actions 可直接写异步）                             | 需在 Actions 中提交 Mutations（异步逻辑嵌套）                |
| TypeScript 支持 | 原生支持（自动类型推导，无需额外配置）                       | 需手动编写类型声明（繁琐）                                   |
| 多 Store 支持   | 天然支持（直接创建多个 Store，无需嵌套）                     | 需通过 Modules 嵌套，复杂场景需 Namespace 隔离               |
| 状态重置        | 内置 `$reset` 方法（Options 风格）                           | 需手动实现重置逻辑                                           |
| 体积            | 极小（约 1KB）                                               | 较大（核心 + 辅助函数）                                      |
| Vue 版本支持    | Vue 2/Vue 3（需适配插件）                                    | Vue 2（3.x）/ Vue 3（4.x）                                   |
| 官方推荐        | 是（Vue 3 官方首选）                                         | 否（已停止主动开发，仅维护 bug）                             |

### 关键差异示例

#### 1. 状态修改对比

**Vuex 写法**（需 Mutations + Actions）：

```JavaScript
// Vuex Store
const store = new Vuex.Store({
  state: { count: 0 },
  mutations: {
    INCREMENT(state, num) { // 必须同步
      state.count += num
    }
  },
  actions: {
    increment(context, num) {
      context.commit('INCREMENT', num) // 提交 Mutation
    },
    asyncIncrement({ commit }, num) {
      setTimeout(() => {
        commit('INCREMENT', num) // 异步逻辑需嵌套提交
      }, 1000)
    }
  }
})

// 组件中使用
store.dispatch('increment', 1) // 调用 Action
```

**Pinia 写法**（直接用 Actions）：

```JavaScript
// Pinia Store（Actions 直接修改）
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  actions: {
    increment(num = 1) {
      this.count += num // 同步直接修改
    },
    async asyncIncrement(num = 1) {
      setTimeout(() => {
        this.count += num // 异步直接修改，无需 commit
      }, 1000)
    }
  }
})

// 组件中使用
const counterStore = useCounterStore()
counterStore.increment(1)
counterStore.asyncIncrement(1)
```

#### 2. 多状态管理对比

**Vuex**（需 Modules + Namespace）：

```JavaScript
// Vuex 多模块
const userModule = {
  namespaced: true, // 必须开启命名空间
  state: { name: '张三' },
  mutations: { SET_NAME(state, name) { state.name = name } },
  actions: { setName({ commit }, name) { commit('SET_NAME', name) } }
}

const cartModule = {
  namespaced: true,
  state: { goods: [] },
  mutations: { ADD_GOODS(state, goods) { state.goods.push(goods) } }
}

const store = new Vuex.Store({
  modules: { user: userModule, cart: cartModule }
})

// 组件中使用
store.dispatch('user/setName', '李四') // 需指定命名空间
```

**Pinia**（直接创建多个 Store，无需嵌套）：

```JavaScript
// 单独创建 userStore
export const useUserStore = defineStore('user', {
  state: () => ({ name: '张三' }),
  actions: { setName(name) { this.name = name } }
})

// 单独创建 cartStore
export const useCartStore = defineStore('cart', {
  state: () => ({ goods: [] }),
  actions: { addGoods(goods) { this.goods.push(goods) } }
})

// 组件中使用（直接导入，无命名空间）
const userStore = useUserStore()
userStore.setName('李四')

const cartStore = useCartStore()
cartStore.addGoods({ id: 1, name: '商品' })
```

## 四、注意事项

1. **响应式解构**：直接解构 Store 中的 state/getters 会丢失响应式，需使用 `storeToRefs`（仅用于 state/getters，actions 可直接解构）。
2. **Store 实例唯一性**：`useXxxStore()` 多次调用返回同一个实例，无需担心重复创建。
3. **Composition API 风格限制**：`$reset` 方法仅 Options API 风格支持，Composition API 需手动实现重置逻辑。
4. **对象修改注意**：使用 `$patch` 对象形式修改嵌套对象时，会覆盖整个对象，需手动合并属性（推荐使用 `$patch` 函数形式）。
5. **Vue 2 适配**：Pinia 支持 Vue 2，但需安装 `@pinia/vue2-adapter` 适配插件。

## 五、总结

Pinia 是 Vuex 的极简替代方案，核心优势是**简洁、灵活、原生支持 TS 和异步**。对于新项目，优先选择 Pinia；对于旧 Vuex 项目，可逐步迁移（Pinia 与 Vuex 可共存）。

核心用法口诀：

- 定义 Store：`defineStore` + （Options/Composition）风格
- 使用 Store：调用函数获取实例，`storeToRefs` 保持解构响应式
- 修改状态：Actions（推荐）、`$patch`（批量）、直接修改（简单场景）