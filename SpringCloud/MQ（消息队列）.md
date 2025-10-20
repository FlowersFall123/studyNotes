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
<img width="1875" height="949" alt="image" src="https://github.com/user-attachments/assets/317876eb-b422-483c-86e4-0081ddb6036e" />


------

## 三、RabbitMQ 管理操作

### 创建队列（Queue）

在管理控制台中：

1. 打开 **Queues** 标签页
2. 点击 **Add a new queue**
3. 填写队列名称（如 `testQueue`），点击 **Add queue**

> ✅ 队列用于存放消息，是消息传递的中间存储区域。
<img width="1868" height="909" alt="image" src="https://github.com/user-attachments/assets/2c8f9253-56ed-440f-ab96-cb8fa362996b" />

------

### 绑定队列（Binding）

1. 在 **Exchanges** 页面选择一个交换机（如 `amq.direct`）
2. 点击进入后绑定目标队列
3. 设置 Routing Key，完成绑定

> ✅ 绑定决定了消息路由的路径。
<img width="1837" height="870" alt="image" src="https://github.com/user-attachments/assets/33d76f38-33f4-4b92-9dd8-32a805c05e64" />



------

### 发送消息（Publish Message）

1. 在 **Exchanges** 页面选择交换机
2. 点击 **Publish message**
3. 填写 routing key 与消息内容
4. 点击 **Publish message** 发送
<img width="1838" height="860" alt="image" src="https://github.com/user-attachments/assets/eab553a0-c05a-4429-8126-aaff4b46ed37" />

<img width="1824" height="865" alt="image" src="https://github.com/user-attachments/assets/9fa6a9b8-9545-4f4e-8115-9dddf590d656" />


------

### 查看消息（Get Messages）

1. 打开 **Queues** 页面
2. 选择队列
3. 点击 **Get messages**
4. 查看消息内容、状态、是否被消费等信息
<img width="1852" height="890" alt="image" src="https://github.com/user-attachments/assets/57426efe-506f-42d7-a232-507e7505e2e8" />



------

### 添加用户（Add User）

1. 打开 **Admin → Users**
2. 点击 **Add user**
3. 设置用户名、密码、角色（如 `administrator`）
<img width="1829" height="879" alt="image" src="https://github.com/user-attachments/assets/abbb4b3a-27cd-4479-9805-fc177e8c9ea2" />



------

### 创建虚拟主机（Virtual Host）

1. 打开 **Admin → Virtual Hosts**
2. 点击 **Add a new virtual host**
3. 输入名称（如 `/myvhost`），点击 **Add**

> ✅ 虚拟主机相当于逻辑隔离空间，不同业务模块可使用不同 vhost 实现数据隔离。
<img width="1841" height="873" alt="image" src="https://github.com/user-attachments/assets/3bb297d9-5584-4b8d-9e0c-0ee3427616a2" />



------

### 数据隔离

### 通过 **虚拟主机（Virtual Host）** 和 **用户权限控制** 实现数据隔离：

- 不同项目/模块使用不同的虚拟主机
- 用户仅能访问被授权的虚拟主机资源（队列、交换机等）
- 提高安全性与管理灵活性
<img width="1832" height="872" alt="image" src="https://github.com/user-attachments/assets/ff89170c-1c22-4002-9129-18e74aebb289" />


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

###  九、消息转换器（Message Converter）
<img width="1771" height="873" alt="消息转换器1" src="https://github.com/user-attachments/assets/d21683e2-d029-4e22-a7c3-da8dddb075db" />


####  一、添加依赖

在 `pom.xml` 中引入 **Jackson 的 XML 数据格式化支持**：

```
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

------

#### 二、配置消息转换器

在 **Spring Boot 启动类**（或任意配置类）中配置消息转换器为 JSON 格式：

> ⚠️ 注意：**不要导包错误！**
>  需使用以下两个包👇

```
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
```

配置 Bean：

```
@Bean
public MessageConverter messageConverter() {
    return new Jackson2JsonMessageConverter();
}
```

📖 **作用**：
 让 RabbitMQ 支持 **对象消息的自动序列化和反序列化**（即在发送时将 Java 对象转换为 JSON，在接收时自动还原为 Map 或实体类）。

------

#### 三、发送消息（对象）

```
@Test
public void TestObjectQueue() {
    Map<String, Object> map = new HashMap<>();
    map.put("name", "fz");
    map.put("age", 18);

    // 发送对象消息到队列
    rabbitTemplate.convertAndSend("object.queue", map);
}
```

🧠 **说明：**

- `convertAndSend()` 会自动将对象序列化为 JSON 格式后发送；
- 无需手动序列化，简单高效。

<img width="1780" height="870" alt="消息转换器2" src="https://github.com/user-attachments/assets/b0171368-23e0-45dd-8dd7-b6bedbe9171e" />


------

#### 四、接收消息（对象）

```
@RabbitListener(queues = "object.queue")
public void listenObjectQueue(Map<String, Object> message) throws InterruptedException {
    System.out.println("object.queue 的消息: " + message);
}
```

🧠 **说明：**

- Spring AMQP 会自动将接收到的 JSON 消息反序列化为 `Map`；
- 如果你有自定义实体类，也可以直接接收为对应对象。
