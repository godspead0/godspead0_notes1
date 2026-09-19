# AJAX 与异步编程完整指南

## AJAX 概述

AJAX (Asynchronous JavaScript and XML) 允许网页在不重新加载整个页面的情况下与服务器交换数据并更新部分网页内容。

## axios 使用

### 基本语法

```javascript
axios({
  url: '请求网址',
  params: { // GET 请求参数
    参数名: 值1,
    参数名: 值2
  },
  method: '请求方法',
  data: { // POST 请求数据
    参数名: 值
  }
}).then(result => {
  // 处理成功结果
  console.log(result.data);
}).catch(error => {
  // 处理错误
  console.error(error);
});
```

### URL 格式详解

```
http://example.com/api/users?name=john&age=25&city=beijing
└─┬─┘ └─────┬────┘└─┬─┘ └─────────────┬─────────────┘
协议   域名     资源路径 查询参数分隔符     查询参数
```

### HTTP 请求方法

| 方法 | 用途 | 参数位置 |
|------|------|----------|
| GET | 获取数据 | params |
| POST | 提交数据 | data |
| PUT | 修改数据 | data |
| DELETE | 删除数据 | params |
| PATCH | 部分修改 | data |

### 表单验证示例

```javascript
// 用户名：最少8位，中英文和数字
// 密码：最少6位
const userData = {
  username: 'user12345',
  password: 'pass123'
};

axios({
  method: 'POST',
  url: '/api/login',
  data: userData
}).then(response => {
  console.log('登录成功', response.data);
}).catch(error => {
  console.error('登录失败', error);
});
```

### 错误处理

```javascript
axios.get('/api/data')
  .then(result => {
    // 成功处理
    console.log(result.data);
  })
  .catch(error => {
    // 错误处理
    if (error.response) {
      // 服务器响应错误
      console.log('状态码:', error.response.status);
      console.log('错误数据:', error.response.data);
    } else if (error.request) {
      // 请求未收到响应
      console.log('网络错误:', error.request);
    } else {
      // 其他错误
      console.log('错误:', error.message);
    }
  });
```

## HTTP 协议与请求报文

### 请求报文结构

```
请求行: GET /api/users?name=john HTTP/1.1
请求头: 
  Host: example.com
  Content-Type: application/json
  Authorization: Bearer token123
请求体: {"username": "john", "password": "123456"}
```

### 响应报文结构

```
状态行: HTTP/1.1 200 OK
响应头:
  Content-Type: application/json
  Content-Length: 125
响应体: {"success": true, "data": {...}}
```

## 表单处理

### 表单序列化插件

```javascript
// 使用 form-serialized 插件
const form = document.querySelector('form');
const data = serialize(form, { 
  hash: true,    // 以 JS 对象格式输出
  empty: false   // 不获取空值
});

console.log(data);
// 输出: {username: "john", email: "john@example.com"}
```

### 原生表单序列化

```javascript
function serializeForm(form) {
  const formData = new FormData(form);
  const data = {};
  
  for (let [key, value] of formData.entries()) {
    data[key] = value;
  }
  
  return data;
}
```

## Bootstrap 模态框

```html
<!-- 触发按钮 -->
<button type="button" class="btn btn-primary" 
        data-bs-toggle="modal" data-bs-target="#exampleModal">
  打开模态框
</button>

<!-- 模态框结构 -->
<div class="modal fade" id="exampleModal" tabindex="-1" 
     aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">模态框标题</h5>
        <button type="button" class="btn-close" 
                data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
        模态框内容...
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" 
                data-bs-dismiss="modal">关闭</button>
        <button type="button" class="btn btn-primary">保存</button>
      </div>
    </div>
  </div>
</div>
```

### JavaScript 控制

```javascript
// 通过 JavaScript 控制模态框
const myModal = new bootstrap.Modal(document.getElementById('exampleModal'));

// 显示模态框
myModal.show();

// 隐藏模态框
myModal.hide();

// 事件监听
document.getElementById('exampleModal').addEventListener('show.bs.modal', function () {
  console.log('模态框即将显示');
});
```

## 文件上传

### FormData 使用

```javascript
// 图片上传
const fileInput = document.querySelector('#fileInput');
const uploadButton = document.querySelector('#uploadButton');

uploadButton.addEventListener('click', function() {
  const file = fileInput.files[0];
  const fd = new FormData();
  
  fd.append('avatar', file);          // 文件
  fd.append('username', 'john');      // 其他字段
  fd.append('description', '用户头像');
  
  axios.post('/api/upload', fd, {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  }).then(response => {
    console.log('上传成功', response.data);
  }).catch(error => {
    console.error('上传失败', error);
  });
});
```

### 多文件上传

```javascript
function uploadMultipleFiles(files) {
  const fd = new FormData();
  
  // 添加多个文件
  files.forEach((file, index) => {
    fd.append(`files`, file);
  });
  
  // 添加额外数据
  fd.append('uploader', 'john');
  fd.append('timestamp', Date.now());
  
  return axios.post('/api/upload-multiple', fd);
}
```

## 本地存储

### localStorage 操作

```javascript
// 保存数据到本地
function saveToLocalStorage(key, data) {
  try {
    const jsonString = JSON.stringify(data);
    localStorage.setItem(key, jsonString);
    return true;
  } catch (error) {
    console.error('保存失败:', error);
    return false;
  }
}

// 从本地读取数据
function getFromLocalStorage(key) {
  try {
    const jsonString = localStorage.getItem(key);
    return jsonString ? JSON.parse(jsonString) : null;
  } catch (error) {
    console.error('读取失败:', error);
    return null;
  }
}

// 使用示例
const userSettings = {
  theme: 'dark',
  language: 'zh-CN',
  notifications: true
};

saveToLocalStorage('userSettings', userSettings);

const savedSettings = getFromLocalStorage('userSettings');
console.log(savedSettings);
```

### 图片本地存储

```javascript
// 将图片转换为 Base64 并保存
function saveImageToLocalStorage(key, file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    
    reader.onload = function(e) {
      try {
        localStorage.setItem(key, e.target.result);
        resolve(true);
      } catch (error) {
        reject(error);
      }
    };
    
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

// 从本地存储加载图片
function loadImageFromLocalStorage(key) {
  const imageData = localStorage.getItem(key);
  if (imageData) {
    return imageData; // 返回 Base64 数据
  }
  return null;
}
```

## AJAX 原理与 XMLHttpRequest

### 基础 XMLHttpRequest

```javascript
const xhr = new XMLHttpRequest();

xhr.open('GET', 'https://api.example.com/data');

xhr.addEventListener('loadend', function() {
  console.log('请求完成');
  console.log('状态:', xhr.status);
  console.log('响应:', xhr.response);
});

xhr.send();
```

### 完整的 XMLHttpRequest 示例

```javascript
function makeRequest(method, url, data = null) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open(method, url);
    
    // 设置请求头
    xhr.setRequestHeader('Content-Type', 'application/json');
    
    // 事件处理
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        try {
          const response = JSON.parse(xhr.responseText);
          resolve(response);
        } catch (error) {
          resolve(xhr.responseText);
        }
      } else {
        reject(new Error(`请求失败: ${xhr.status}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('网络错误'));
    };
    
    xhr.ontimeout = function() {
      reject(new Error('请求超时'));
    };
    
    // 发送数据
    if (data && method !== 'GET') {
      xhr.send(JSON.stringify(data));
    } else {
      xhr.send();
    }
  });
}

// 使用示例
makeRequest('GET', '/api/users')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### 带进度的文件上传

```javascript
function uploadWithProgress(file, onProgress) {
  const xhr = new XMLHttpRequest();
  const formData = new FormData();
  formData.append('file', file);
  
  xhr.upload.addEventListener('progress', function(e) {
    if (e.lengthComputable) {
      const percentComplete = (e.loaded / e.total) * 100;
      onProgress(percentComplete);
    }
  });
  
  xhr.open('POST', '/api/upload');
  
  return new Promise((resolve, reject) => {
    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.response));
      } else {
        reject(new Error('上传失败'));
      }
    };
    
    xhr.onerror = () => reject(new Error('网络错误'));
    xhr.send(formData);
  });
}
```

## Promise 详解

### Promise 基础

```javascript
const p = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    const success = Math.random() > 0.5;
    
    if (success) {
      resolve('操作成功！');
    } else {
      reject(new Error('操作失败！'));
    }
  }, 1000);
});

p.then(value => {
  console.log(value); // 成功时执行
}).catch(reason => {
  console.error(reason); // 失败时执行
}).finally(() => {
  console.log('无论成功失败都会执行');
});
```

### Promise 状态

```javascript
// Promise 的三种状态
const states = {
  PENDING: 'pending',      // 初始状态
  FULFILLED: 'fulfilled',  // 成功状态
  REJECTED: 'rejected'     // 失败状态
};

// 状态转换示例
const promise = new Promise((resolve, reject) => {
  console.log('初始状态: pending');
  
  setTimeout(() => {
    resolve('成功数据');
    console.log('状态变为: fulfilled');
  }, 1000);
});

promise.then(data => {
  console.log('接收到数据:', data);
});
```

## 封装 axios

### 自定义 AJAX 库

```javascript
function myAxios(config) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    // 处理查询参数
    if (config.params) {
      const paramsObj = new URLSearchParams(config.params);
      const queryString = paramsObj.toString();
      config.url += `?${queryString}`;
    }
    
    xhr.open(config.method || 'GET', config.url);
    
    // 设置请求头
    if (config.headers) {
      Object.keys(config.headers).forEach(key => {
        xhr.setRequestHeader(key, config.headers[key]);
      });
    }
    
    xhr.addEventListener('loadend', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        try {
          resolve(JSON.parse(xhr.response));
        } catch (error) {
          resolve(xhr.response);
        }
      } else {
        reject(new Error(`请求失败: ${xhr.status} ${xhr.statusText}`));
      }
    });
    
    xhr.addEventListener('error', () => {
      reject(new Error('网络错误'));
    });
    
    // 发送数据
    if (config.data) {
      const contentType = config.headers?.['Content-Type'];
      
      if (contentType === 'application/json') {
        const jsonStr = JSON.stringify(config.data);
        xhr.send(jsonStr);
      } else if (contentType === 'multipart/form-data') {
        // FormData 不需要设置 Content-Type，浏览器会自动设置
        xhr.send(config.data);
      } else {
        xhr.send(config.data);
      }
    } else {
      xhr.send();
    }
  });
}
```

### 使用封装的 AJAX 库

```javascript
// GET 请求示例
myAxios({
  method: 'GET',
  url: 'https://api.github.com/users/github',
  params: {
    page: 1,
    limit: 10
  }
}).then(res => {
  console.log('获取成功:', res);
}).catch(err => {
  console.error('获取失败:', err);
});

// POST 请求示例
myAxios({
  method: 'POST',
  url: '/api/users',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  data: {
    name: 'John Doe',
    email: 'john@example.com'
  }
}).then(res => {
  console.log('创建成功:', res);
});
```

## Promise 链式调用

### 链式调用示例

```javascript
function getUser(userId) {
  return myAxios(`/api/users/${userId}`);
}

function getUserPosts(userId) {
  return myAxios(`/api/users/${userId}/posts`);
}

function getPostComments(postId) {
  return myAxios(`/api/posts/${postId}/comments`);
}

// 链式调用避免回调地狱
getUser(1)
  .then(user => {
    console.log('用户信息:', user);
    return getUserPosts(user.id);
  })
  .then(posts => {
    console.log('用户文章:', posts);
    return getPostComments(posts[0].id);
  })
  .then(comments => {
    console.log('文章评论:', comments);
  })
  .catch(error => {
    console.error('操作失败:', error);
  });
```

### Promise 静态方法

```javascript
// Promise.all - 所有 Promise 都成功
Promise.all([
  myAxios('/api/users'),
  myAxios('/api/posts'),
  myAxios('/api/comments')
]).then(([users, posts, comments]) => {
  console.log('所有数据加载完成');
  console.log('用户:', users);
  console.log('文章:', posts);
  console.log('评论:', comments);
});

// Promise.race - 第一个完成的结果
Promise.race([
  fetch('/api/fast-endpoint'),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('超时')), 5000)
  )
]).then(result => {
  console.log('第一个完成的结果:', result);
});

// Promise.allSettled - 所有 Promise 完成（无论成功失败）
Promise.allSettled([
  Promise.resolve('成功'),
  Promise.reject('失败'),
  Promise.resolve('另一个成功')
]).then(results => {
  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      console.log(`Promise ${index}: 成功`, result.value);
    } else {
      console.log(`Promise ${index}: 失败`, result.reason);
    }
  });
});
```

## async/await

### 基础用法

```javascript
async function fetchUserData(userId) {
  try {
    const user = await myAxios(`/api/users/${userId}`);
    const posts = await myAxios(`/api/users/${userId}/posts`);
    const comments = await myAxios(`/api/users/${userId}/comments`);
    
    return {
      user,
      posts,
      comments
    };
  } catch (error) {
    console.error('获取用户数据失败:', error);
    throw error;
  }
}

// 使用 async 函数
fetchUserData(1)
  .then(data => console.log('用户数据:', data))
  .catch(error => console.error('错误:', error));
```

### 并行请求优化

```javascript
async function fetchUserDataParallel(userId) {
  try {
    // 并行执行多个请求
    const [user, posts, comments] = await Promise.all([
      myAxios(`/api/users/${userId}`),
      myAxios(`/api/users/${userId}/posts`),
      myAxios(`/api/users/${userId}/comments`)
    ]);
    
    return { user, posts, comments };
  } catch (error) {
    console.error('获取数据失败:', error);
    throw error;
  }
}

// 错误处理模式
async function robustFetch(url, options = {}) {
  const maxRetries = 3;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await myAxios({ url, ...options });
      return response;
    } catch (error) {
      if (attempt === maxRetries) {
        throw new Error(`请求失败，已重试 ${maxRetries} 次: ${error.message}`);
      }
      
      console.log(`第 ${attempt} 次请求失败，准备重试...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
    }
  }
}
```

## 事件循环 (Event Loop)

### 执行顺序规则

```javascript
console.log('1. 同步任务开始');

// 微任务
Promise.resolve().then(() => {
  console.log('3. 微任务执行');
});

// 宏任务
setTimeout(() => {
  console.log('5. 宏任务执行');
}, 0);

// 同步任务
console.log('2. 同步任务结束');

// 另一个微任务
Promise.resolve().then(() => {
  console.log('4. 另一个微任务');
});

// 输出顺序:
// 1. 同步任务开始
// 2. 同步任务结束
// 3. 微任务执行
// 4. 另一个微任务
// 5. 宏任务执行
```

### 任务队列详解

```javascript
async function demonstrateEventLoop() {
  console.log('1. 同步代码开始');
  
  // 微任务 - Promise
  Promise.resolve().then(() => console.log('3. 微任务 - Promise'));
  
  // 宏任务 - setTimeout
  setTimeout(() => console.log('6. 宏任务 - setTimeout'), 0);
  
  // 微任务 - async/await
  await Promise.resolve();
  console.log('4. 微任务 - async/await');
  
  // 宏任务 - setInterval
  setInterval(() => {
    console.log('7. 宏任务 - setInterval');
  }, 1000);
  
  console.log('2. 同步代码结束');
  
  // 微任务 - queueMicrotask
  queueMicrotask(() => {
    console.log('5. 微任务 - queueMicrotask');
  });
}

demonstrateEventLoop();
```

### 实际应用中的执行顺序

```javascript
// 模拟数据获取场景
async function fetchData() {
  console.log('开始获取数据');
  
  // 同步操作
  const userId = 1;
  console.log('用户ID:', userId);
  
  // 微任务 - Promise
  const userPromise = myAxios(`/api/users/${userId}`);
  
  // 宏任务
  setTimeout(() => {
    console.log('超时检查');
  }, 5000);
  
  try {
    // await 会暂停执行，让出线程
    const user = await userPromise;
    console.log('用户数据:', user);
    
    // 继续执行后续微任务
    const posts = await myAxios(`/api/users/${userId}/posts`);
    console.log('用户文章:', posts);
    
  } catch (error) {
    console.error('获取数据失败:', error);
  }
  
  console.log('数据获取流程结束');
}

fetchData();
console.log('主线程继续执行其他任务');
```

## 总结

AJAX 和异步编程是现代 Web 开发的核心技术。通过掌握 XMLHttpRequest、Promise、async/await 等概念，可以构建出响应迅速、用户体验良好的 Web 应用程序。理解事件循环机制有助于编写更高效、更可预测的异步代码。