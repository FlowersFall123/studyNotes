# MQ（消息队列）

## 一、RabbitMQ 简介

**RabbitMQ** 是一个基于 **AMQP（Advanced Message Queuing Protocol）** 的开源消息中间件，主要用于：

- 异步处理（提升系统性能）
- 应用解耦（降低模块间依赖）
- 流量削峰（应对高并发）

📘 官网：[RabbitMQ 官方网站](https://www.rabbitmq.com/)

------

## 二、安装与部署（Docker 方式）

```
docker run \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=admin \
 -v mq-plugins:/plugins \
 --name mq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 --network hm-net \
 -d \
 rabbitmq:3.8-management
```

### 📋 参数说明

| 参数                       | 含义                  |
| -------------------------- | --------------------- |
| `-e RABBITMQ_DEFAULT_USER` | 默认用户名            |
| `-e RABBITMQ_DEFAULT_PASS` | 默认密码              |
| `-v mq-plugins:/plugins`   | 挂载插件目录          |
| `--name mq`                | 容器名称              |
| `--hostname mq`            | 主机名                |
| `-p 15672:15672`           | Web 管理控制台端口    |
| `-p 5672:5672`             | 消息传输端口          |
| `--network hm-net`         | 指定 Docker 网络      |
| `-d`                       | 后台运行              |
| `rabbitmq:3.8-management`  | 带 Web 管理插件的镜像 |

------

### 启动日志查看

```
docker logs -f mq
```

### Web 管理页面

访问：

```
http://192.168.195.131:15672/
```

使用上面设置的账号密码：

```
用户名：admin
密码：admin
```

------

## 三、RabbitMQ 管理操作

### 创建队列（Queue）

在管理控制台中：

1. 打开 **Queues** 标签页
2. 点击 **Add a new queue**
3. 填写队列名称（如 `testQueue`），点击 **Add queue**

> ✅ 队列用于存放消息，是消息传递的中间存储区域。

<img width="1868" height="909" alt="image" src="https://github.com/user-attachments/assets/6ab14c09-191f-41ac-8de3-035a857a3902" />
------

### 绑定队列（Binding）

1. 在 **Exchanges** 页面选择一个交换机（如 `amq.direct`）
2. 点击进入后绑定目标队列
3. 设置 Routing Key，完成绑定

> ✅ 绑定决定了消息路由的路径。
<img width="1838" height="860" alt="image" src="https://github.com/user-attachments/assets/70bb2ae7-c8cd-43e9-ba03-e0218459f045" />


------

### 发送消息（Publish Message）

1. 在 **Exchanges** 页面选择交换机
2. 点击 **Publish message**
3. 填写 routing key 与消息内容
4. 点击 **Publish message** 发送
<img width="1824" height="865" alt="image" src="https://github.com/user-attachments/assets/edd2f7d3-937e-41ab-928e-e7358f3b826b" />
<img width="1829" height="879" alt="image" src="https://github.com/user-attachments/assets/25042217-cbc1-4f50-acd6-a897638ea3ec" />

------

### 查看消息（Get Messages）

1. 打开 **Queues** 页面
2. 选择队列
3. 点击 **Get messages**
4. 查看消息内容、状态、是否被消费等信息
<img width="1841" height="873" alt="image" src="https://github.com/user-attachments/assets/da0f897a-d48f-4f99-964e-79b24352ba06" />


------

### 添加用户（Add User）

1. 打开 **Admin → Users**
2. 点击 **Add user**
3. 设置用户名、密码、角色（如 `administrator`）
<img width="1832" height="872" alt="image" src="https://github.com/user-attachments/assets/a305edf2-8d36-4123-ae92-1b9800cac0f9" />


------

### 创建虚拟主机（Virtual Host）

1. 打开 **Admin → Virtual Hosts**
2. 点击 **Add a new virtual host**
3. 输入名称（如 `/myvhost`），点击 **Add**

> ✅ 虚拟主机相当于逻辑隔离空间，不同业务模块可使用不同 vhost 实现数据隔离。
<img width="1837" height="870" alt="image" src="https://github.com/user-attachments/assets/86c3dc91-92e6-4a98-8bdf-478820a91618" />


------

### 数据隔离

### 通过 **虚拟主机（Virtual Host）** 和 **用户权限控制** 实现数据隔离：

- 不同项目/模块使用不同的虚拟主机
- 用户仅能访问被授权的虚拟主机资源（队列、交换机等）
- 提高安全性与管理灵活性
<img width="1852" height="890" alt="image" src="https://github.com/user-attachments/assets/d8f73a76-7c35-4df1-8c75-9a9ed4091878" />
