---
title: vue对axios的封装和使用
date: 2026-04-18 20:00:00
tags:
  - Hexo
  - Axios
categories:
  - 教程
keywords: "Vue, Axios, HTTP请求, 拦截器, 前端封装"
---
在 Vue 项目中，使用 `axios` 进行 HTTP 请求是非常常见的做法。为了提高代码的可维护性、统一错误处理和请求拦截/响应拦截逻辑，对`axios`进行封装使用。

#
### 一、基础封装（适用于 Vue 2 / Vue 3）

#### 1\. 安装 axios

```bash
npm install axios
```

#### 2\. 创建封装文件：`src/utils/request.js`

```js
import axios from 'axios'
import { Message } from 'element-ui' // 或你使用的 UI 库

// 创建 axios 实例
const service = axios.create({
  baseURL: process.env.VUE_APP_API, // 设置默认 base URL（来自 .env）
  timeout: 5000, // 超时时间
  withCredentials: false // 是否携带 cookie
})

// 请求拦截器
service.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers['Authorization'] = `Bearer ${token}`
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// 响应拦截器
service.interceptors.response.use(
  response => {
    const res = response.data

    if (res.code !== 200) {
      Message.error(res.message || 'Error')

      if (res.code === 401) {
        // 处理 token 失效
        localStorage.removeItem('token')
        window.location.href = '/login'
      }

      return Promise.reject(new Error(res.message || 'Error'))
    } else {
      return res
    }
  },
  error => {
    Message.error('网络异常，请检查网络连接')
    return Promise.reject(error)
  }
)

export default service
```

* * *

### 二、使用封装后的 `axios`

#### 1\. 在组件中直接调用

```js
import request from '@/utils/request'

export default {
  methods: {
    async fetchData() {
      try {
        const res = await request.get('/api/data')
        console.log(res)
      } catch (error) {
        console.error(error)
      }
    }
  }
}
```

#### 2\. 在 Vuex 中使用

```js
import request from '@/utils/request'

export default {
  actions: {
    async login({ commit }, payload) {
      const res = await request.post('/api/login', payload)
      commit('SET_TOKEN', res.token)
    }
  }
}
```

* * *

### 三、支持 TypeScript（可选）

如果你使用的是 **Vue + TypeScript（如 Vite + Vue 3 + TS）**，可以添加类型定义：

#### 1\. 定义统一返回结构

```ts
// src/types/index.ts
export interface ApiResponse<T = any> {
  code: number
  message: string
  data: T
}
```

#### 2\. 使用泛型调用

```ts
import { ApiResponse } from '@/types'

interface User {
  id: number
  name: string
}

const res = await request.get<ApiResponse<User>>('/api/user/1')
console.log(res.data.name)
```

* * *

### 四、取消重复请求（防抖）

建议常用事件无论是否发生请求也做防抖，避免重复性耗费资源  
防止用户多次点击按钮导致重复请求：

```js
import axios from 'axios'

const pendingMap = new Map()

function generateReqKey(config) {
  return [config.method, config.url].join('&')
}

function addPending(config) {
  const key = generateReqKey(config)
  const controller = new AbortController()
  config.signal = controller.signal
  if (!pendingMap.has(key)) {
    pendingMap.set(key, controller)
  }
}

function removePending(key) {
  if (pendingMap.has(key)) {
    const controller = pendingMap.get(key)
    controller.abort()
    pendingMap.delete(key)
  }
}

// 请求拦截器
service.interceptors.request.use(config => {
  addPending(config)
  return config
})

// 响应拦截器
service.interceptors.response.use(response => {
  removePending(generateReqKey(response.config))
  return response
}, error => {
  removePending(generateReqKey(error.config))
  return Promise.reject(error)
})
```

* * *

### 五、全局 loading（可选）

可以在请求拦截器中添加 loading，在响应拦截器中关闭：

```js
let loadingCount = 0

function startLoading() {
  if (loadingCount === 0) {
    // 显示 loading 动画
    store.dispatch('showLoading')
  }
  loadingCount++
}

function endLoading() {
  loadingCount--
  if (loadingCount <= 0) {
    store.dispatch('hideLoading')
  }
}

// 请求拦截器
service.interceptors.request.use(config => {
  startLoading()
  return config
})

// 响应拦截器
service.interceptors.response.use(response => {
  endLoading()
  return response
}, error => {
  endLoading()
  return Promise.reject(error)
})
```

* * *

### 六、推荐目录结构

```
src/
├── utils/
│   └── request.js       # axios 封装
├── types/
│   └── index.ts          # 接口类型定义（TS）
├── api/
│   ├── user.js           # 用户相关接口
│   ├── product.js        # 商品相关接口
│   └── index.js          # 导出所有 API
├── views/
│   └── ...               # 页面组件
└── store/
    └── index.js          # Vuex 状态管理（可选）
```

* * *

### 七、API 模块化示例（`src/api/user.js`）

```js
import request from '@/utils/request'

export function login(data) {
  return request({
    url: '/user/login',
    method: 'post',
    data
  })
}

export function getUserInfo(token) {
  return request({
    url: '/user/info',
    method: 'get',
    params: { token }
  })
}
```

在组件中使用：

```js
import { login } from '@/api/user'

export default {
  methods: {
    async handleLogin() {
      const res = await login(this.loginForm)
      console.log(res)
    }
  }
}
```

* * *

### 总结：要点

| 特性 | 实现方式 |
| --- | --- |
| 请求拦截 | 添加 token、loading |
| 响应拦截 | 统一处理成功/失败逻辑 |
| 错误提示 | 使用 UI 框架提示（如 element-ui、vant） |
| 类型安全 | TypeScript 泛型支持 |
| 取消重复请求 | 使用 `AbortController` |
| 模块化组织 | 按功能拆分 API 文件 |
| 全局 loading | 请求计数器控制显示/隐藏 |
