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

访问：http://192.168.195.131:15672/

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
<<<<<<< HEAD

![数据隔离](F:\SpringCloud\图片\数据隔离.png)

##  RabbitMQ 在 Java 客户端中的实践

------

### 一、引入依赖

在 Spring Boot 中使用 RabbitMQ，需要添加 AMQP 依赖：

```
<!-- AMQP 依赖（包含 RabbitMQ） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

------

### 二、基本配置（`application.yml`）

```
spring:
  rabbitmq:
    host: 192.168.195.131  # 虚拟机 IP
    port: 5672             # 端口
    virtual-host: /fz      # 虚拟主机（我后面添加的）
    username: fz           # 用户名
    password: 123          # 密码
```

------

### 三、简单队列（Simple Queue）

最基本的模型：**一个生产者，一个消费者，一个队列。**

#### 发送消息

```
@Autowired
RabbitTemplate rabbitTemplate;

@Test
public void publishMessage() {
    // 1. 队列名
    String queueName = "simple.queue";
    // 2. 消息内容
    String message = "hello, spring amqp!";
    // 3. 发送消息
    rabbitTemplate.convertAndSend(queueName, message);
}
```

#### 接收消息

```
@RabbitListener(queues = "simple.queue")
public void listenSimpleQueue(String message) {
    System.out.println("收到 simple.queue 的消息: " + message);
}
```

------

### 四、工作队列（Work Queues）

**Work Queues** 模式：
 多个消费者监听同一个队列，**每条消息只会被一个消费者处理。**

------

#### 发送消息

```
@Test
public void publishWorkQueue() {
    String queueName = "work.queue";
    for (int i = 0; i < 50; i++) {
        String message = "hello, work.queue!" + i;
        rabbitTemplate.convertAndSend(queueName, message);
    }
}
```

------

#### 接收消息（多个消费者）

不会因为消费者的处理能力不同而导致分配数量不一致（默认是**轮询分发**）。

```
@RabbitListener(queues = "work.queue")
public void listenWorkQueue1(String message) throws InterruptedException {
    System.out.println("消费者1收到消息: " + message);
    Thread.sleep(25);
}

@RabbitListener(queues = "work.queue")
public void listenWorkQueue2(String message) throws InterruptedException {
    System.err.println("消费者2收到消息: " + message);
    Thread.sleep(200);
}
```

------

#### 优化：公平分发机制

给性能好的消费者分配更多消息，**只有处理完消息之后才能继续获取信息，所以处理消息快的（性能好的）会分配到更多的消息**：

```
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1  # 每次只获取一条消息，处理完成后再取下一条
```

------

### 五、发布订阅模式（Fanout Exchange）

**Fanout 交换机** 会将消息**广播**到所有绑定的队列。

------

#### 前置准备

创建：

- 队列：`fanout.queue1`、`fanout.queue2`
- 交换机：`fz.fanout`
- 并将交换机与两个队列绑定。

------

#### 发送消息

```
@Test
public void TestFanoutExchange() {
    String exchange = "fz.fanout";
    String message = "hello, everyone!";
    rabbitTemplate.convertAndSend(exchange, null, message);
}
```

------

#### 接收消息

两个队列都会收到广播的消息：

```
@RabbitListener(queues = "fanout.queue1")
public void listenFanoutQueue1(String message) {
    System.out.println("收到 fanout.queue1 的消息: " + message);
}

@RabbitListener(queues = "fanout.queue2")
public void listenFanoutQueue2(String message) {
    System.err.println("收到 fanout.queue2 的消息: " + message);
}
```

------

### 六、路由模式（Direct Exchange）

**Direct 模式**会根据 `Routing Key` 精确匹配队列。

适用于需要**精确控制消息流向**的场景。

------

#### 前置准备

创建：

- 队列：`direct.queue1`、`direct.queue2`
- 交换机：`fz.direct`
- 设置 Routing Key（一个队列可以绑定多个 key）

<img width="1801" height="674" alt="image" src="https://github.com/user-attachments/assets/d8c6b258-3917-4c87-8769-0deea7ad9665" />


------

#### 发送消息

```
@Test
public void TestDirectExchange() {
    String exchange = "fz.direct";
    String message = "hello, everyone!";
    rabbitTemplate.convertAndSend(exchange, "blue", message);
}
```

------

#### 接收消息

```
@RabbitListener(queues = "direct.queue1")
public void listenDirectQueue1(String message) {
    System.out.println("direct.queue1 收到消息: " + message);
}

@RabbitListener(queues = "direct.queue2")
public void listenDirectQueue2(String message) {
    System.err.println("direct.queue2 收到消息: " + message);
}
```

------

### 七、主题模式（Topic Exchange）

**Topic 模式**支持通配符匹配，非常灵活。
 常用于“按主题分发”的复杂路由场景。

<img width="1402" height="741" alt="image" src="https://github.com/user-attachments/assets/8b148b94-5e4d-44a5-b6c8-d0bdafc81e08" />


| 通配符 | 含义               |
| ------ | ------------------ |
| `*`    | 匹配一个单词       |
| `#`    | 匹配零个或多个单词 |
<img width="1175" height="534" alt="image" src="https://github.com/user-attachments/assets/7c926f8a-a91c-4779-a52b-df3cf01993da" />

------

#### 发送消息

```
@Test
public void TestTopicExchange() {
    String exchange = "fz.topic";
    String message = "hello, everyone!";
    rabbitTemplate.convertAndSend(exchange, "chain.people", message);
}
```

------

#### 接收消息

```
@RabbitListener(queues = "topic.queue1")
public void listenTopicQueue1(String message) {
    System.out.println("topic.queue1 收到消息: " + message);
}

@RabbitListener(queues = "topic.queue2")
public void listenTopicQueue2(String message) {
    System.err.println("topic.queue2 收到消息: " + message);
}
```

------

### 八、Java 端创建队列、交换机与绑定关系

Spring AMQP 提供两种方式来声明这些组件：

------

#### 方式一：使用 `@Bean` 声明

```
@Configuration
public class FanoutConfiguration {

    // 创建交换机
    @Bean
    public FanoutExchange fanoutExchange() {
        return ExchangeBuilder.fanoutExchange("fz1.fanout").build();
    }

    // 创建队列
    @Bean
    public Queue fanoutQueue1() {
        return new Queue("fanout.queue1");
    }

    // 绑定队列与交换机
    @Bean
    public Binding fanoutBinding1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }
}
```

------

#### 方式二：使用注解声明（更简洁）

```
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "direct.queue1"),
        exchange = @Exchange(name = "fz1.direct", type = ExchangeTypes.DIRECT),
        key = {"red", "blue"}
))
public void listenDirectQueue1(String message) {
    System.out.println("direct.queue1 收到消息: " + message);
}
```
=======
<img width="1852" height="890" alt="image" src="https://github.com/user-attachments/assets/d8f73a76-7c35-4df1-8c75-9a9ed4091878" />
>>>>>>> 714a3a53041227fb552298d2e38991718daa7d22
