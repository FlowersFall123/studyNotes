# **Axios 请求封装**

## 功能概述

这段代码封装了 axios 的 GET 和 POST 请求，提供了统一的错误处理和认证 token 管理。

## 核心功能

### 1. 认证 Token 获取

```
function getAuthToken() {
    return localStorage.getItem('authToken') || '';
}
```

- 从 localStorage 获取 `authToken`
- 如果不存在则返回空字符串

### 2. 默认错误处理

```
const defaultError = () => ElMessage.error('发生错误，请联系管理员。');
const defaultFailure = (message) => ElMessage.warning(message);
```

- `defaultError`: 用于未捕获异常的默认错误提示
- `defaultFailure`: 用于请求失败时的默认警告提示

### 3. POST 请求封装

```
function post(url, data, success, failure = defaultFailure, error = defaultError)
```

- 参数:
  - `url`: 请求地址
  - `data`: 请求数据
  - `success`: 成功回调
  - `failure`: 失败回调(默认使用 `defaultFailure`)
  - `error`: 异常回调(默认使用 `defaultError`)
- 特点:
  - 设置 `Content-Type` 为 `application/x-www-form-urlencoded`
  - 携带认证 token
  - 启用跨域凭证(withCredentials)

### 4. GET 请求封装

```
function get(url, data = null, success, failure = defaultFailure, error = defaultError)
```

- 参数与 POST 类似
- 特点:
  - 将数据作为查询参数(params)发送
  - 携带认证 token
  - 启用跨域凭证

## 响应处理

两种请求都遵循相同的响应处理模式:

1. 检查 `data.success` 判断请求是否成功
2. 成功时调用 `success` 回调，传入:
   - `data.message`: 消息
   - `data.data`: 返回数据
   - `data.status`: 状态码
3. 失败时调用 `failure` 回调，传入相同参数
## 完整代码
```language
import axios from "axios";
import {ElMessage} from "element-plus";//引入用到的组件

function getAuthToken() {
    return localStorage.getItem('authToken') || '';
}

const defaultError = () => ElMessage.error('发生错误，请联系管理员。') //定义默认错误提示语
const defaultFailure = (message) => ElMessage.warning(message) //后端请求返回失败信息时将其打印
//post请求示例
function post(url, data, success, failure = defaultFailure, error = defaultError) {//导入请求路径url,请求数据data,以及失败和成功的操作
    axios.post(url, data, { //使用axios的post请求 传入路径和数据
        headers: {
            "Content-Type": "application/x-www-form-urlencoded", //设置内容类型
            "Authorization": getAuthToken()
        },
        withCredentials: true
    }).then(({data}) => {
        if (data.success)
            success(data.message, data.data,data.status) //判断数据内含的请求成功或失败并做出对应前端操作，执行的操作在组件中引用时书写
        else
            failure(data.message,data.data, data.status)
    }).catch(error)
}

function get(url, data = null, success, failure = defaultFailure, error = defaultError) {
    const config = {
        withCredentials: true,
        params: data , // 将数据作为查询参数
        headers:{
            "Authorization": getAuthToken()
        }
    };

    axios.get(url, config)
        .then(({data}) => {
            if (data.success)
                success(data.message,data.data, data.status)
            else
                failure(data.message,data.data, data.status)
        })
        .catch(error)
}


export {get, post} //导出get post InternalGet方法 供所有组件使用
```