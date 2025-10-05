# Axios请求

## 1. 安装

```
npm install axios
# 或者
yarn add axios
```

------

## 2. 基本使用

### **GET 请求**

```
import axios from "axios";

// 最简单的 GET 请求
axios.get("/api/user?id=1").then(response => {
  console.log(response.data); // 后端返回的数据
}).catch(error => {
  console.error(error);
});

// GET 请求 + 参数
axios.get("/api/user", {
  params: { id: 1 }
}).then(res => {
  console.log(res.data);
});
```

------

### **POST 请求**

```
// POST 请求 - JSON
axios.post("/api/login", {
  username: "admin",
  password: "123456"
}).then(res => {
  console.log(res.data);
}).catch(err => {
  console.error(err);
});
```

------

### **带 headers**

```
axios.get("/api/protected", {
  headers: {
    Authorization: "Bearer " + localStorage.getItem("authToken")
  }
}).then(res => {
  console.log(res.data);
});
```