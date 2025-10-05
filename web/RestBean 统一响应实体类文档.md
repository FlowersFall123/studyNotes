# **RestBean 统一响应实体类文档**

## 📌 基本信息

| 项目       | 说明                           |
| :--------- | :----------------------------- |
| **包路径** | com.example.test_end_01.entity |
| **注解**   | @Data (Lombok)                 |
| **泛型**   | <T> 数据字段类型               |
| **用途**   | REST API 统一响应封装          |

## 🏗️ 类结构

### 🔠 成员变量

| 变量名  | 类型    | 默认值 | 说明         |
| :------ | :------ | :----- | :----------- |
| status  | int     | -      | HTTP 状态码  |
| success | boolean | -      | 请求是否成功 |
| message | String  | null   | 响应消息     |
| data    | T       | null   | 泛型响应数据 |

### 🛠️ 构造方法

| 方法签名                          | 说明           |
| :-------------------------------- | :------------- |
| RestBean(int, boolean, String, T) | 全参数构造方法 |

## ⚡ 静态工厂方法

| 方法签名             | 返回类型    | 参数说明          | 自动设置                 |
| :------------------- | :---------- | :---------------- | :----------------------- |
| success(String)      | RestBean<T> | 成功消息          | status=200, success=true |
| success(String, T)   | RestBean<T> | 成功消息 + 数据   | status=200, success=true |
| failure(int, String) | RestBean<T> | 状态码 + 错误消息 | success=false, data=null |

## 📄 响应示例

### 成功响应（带数据）

```
{
  "status": 200,
  "success": true,
  "message": "查询成功",
  "data": {
    "id": 1,
    "username": "testUser"
  }
}
```

### 失败响应

```
{
  "status": 404,
  "success": false,
  "message": "资源不存在",
  "data": null
}
```

## 🎯 使用场景

| 场景             | 推荐方法             | 示例                      |
| :--------------- | :------------------- | :------------------------ |
| 普通成功响应     | success(String)      | success("操作成功")       |
| 带数据的成功响应 | success(String, T)   | success("查询成功", data) |
| 业务异常响应     | failure(int, String) | failure(400, "参数错误")  |
| 系统异常响应     | failure(int, String) | failure(500, "系统异常")  |

## 🌟 设计优势

1. **标准化响应格式** - 统一所有API的响应结构
2. **泛型支持** - 灵活适应各种数据类型
3. **链式调用** - 配合Lombok支持链式操作
4. **状态隔离** - 明确区分成功/失败状态
5. **自描述性** - 包含状态码和说明消息
##    完整代码

```java
    package com.example.test_end_01.entity;
    
    import lombok.Data;
    
    
    @Data
    public class RestBean<T> {
        private int status;
        private boolean success;
        private String message;
        private T data;
    
        public RestBean(int status, boolean success, String message , T data) {
            this.status = status;
            this.success = success;
            this.message = message;
            this.data = data;
        }
    
        public static <T> RestBean<T> success(String message) {
            return new RestBean<>(200, true, message ,null);
        }
    
        public static <T> RestBean<T> success(String message , T data) {
            return new RestBean<>(200, true, message, data);
        }
    
        public static <T> RestBean<T> failure(int status, String message) {
            return new RestBean<>(status, false, message, null);
        }
    }

```