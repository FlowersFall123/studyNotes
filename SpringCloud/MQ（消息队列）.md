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

![创建queue](F:\SpringCloud\图片\创建queue.png)

------

### 绑定队列（Binding）

1. 在 **Exchanges** 页面选择一个交换机（如 `amq.direct`）
2. 点击进入后绑定目标队列
3. 设置 Routing Key，完成绑定

> ✅ 绑定决定了消息路由的路径。

![绑定队列](F:\SpringCloud\图片\绑定队列.png)

------

### 发送消息（Publish Message）

1. 在 **Exchanges** 页面选择交换机
2. 点击 **Publish message**
3. 填写 routing key 与消息内容
4. 点击 **Publish message** 发送

![发送之后1](F:\SpringCloud\图片\发送之后1.png)

![发送之后2](F:\SpringCloud\图片\发送之后2.png)

------

### 查看消息（Get Messages）

1. 打开 **Queues** 页面
2. 选择队列
3. 点击 **Get messages**
4. 查看消息内容、状态、是否被消费等信息

![查看消息](F:\SpringCloud\图片\查看消息.png)

------

### 添加用户（Add User）

1. 打开 **Admin → Users**
2. 点击 **Add user**
3. 设置用户名、密码、角色（如 `administrator`）

![添加用户](F:\SpringCloud\图片\添加用户.png)

------

### 创建虚拟主机（Virtual Host）

1. 打开 **Admin → Virtual Hosts**
2. 点击 **Add a new virtual host**
3. 输入名称（如 `/myvhost`），点击 **Add**

> ✅ 虚拟主机相当于逻辑隔离空间，不同业务模块可使用不同 vhost 实现数据隔离。

![添加虚拟主机](F:\SpringCloud\图片\添加虚拟主机.png)

------

### 数据隔离

### 通过 **虚拟主机（Virtual Host）** 和 **用户权限控制** 实现数据隔离：

- 不同项目/模块使用不同的虚拟主机
- 用户仅能访问被授权的虚拟主机资源（队列、交换机等）
- 提高安全性与管理灵活性

![数据隔离](F:\SpringCloud\图片\数据隔离.png)