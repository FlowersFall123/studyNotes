# MQ-基础

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


##  四、RabbitMQ 在 Java 客户端中的实践

------

### 1、引入依赖

在 Spring Boot 中使用 RabbitMQ，需要添加 AMQP 依赖：

```
<!-- AMQP 依赖（包含 RabbitMQ） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

------

### 2、基本配置（`application.yml`）

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

### 3、简单队列（Simple Queue）

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

### 4、工作队列（Work Queues）

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

### 5、发布订阅模式（Fanout Exchange）

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

### 6、路由模式（Direct Exchange）

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

### 7、主题模式（Topic Exchange）

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

### 8、Java 端创建队列、交换机与绑定关系

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

###  9、消息转换器（Message Converter）
<img width="1771" height="873" alt="消息转换器1" src="https://github.com/user-attachments/assets/d21683e2-d029-4e22-a7c3-da8dddb075db" />


####  1、添加依赖

在 `pom.xml` 中引入 **Jackson 的 XML 数据格式化支持**：

```
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

------

#### 2、配置消息转换器

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

#### 3、发送消息（对象）

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

#### 4、接收消息（对象）

```
@RabbitListener(queues = "object.queue")
public void listenObjectQueue(Map<String, Object> message) throws InterruptedException {
    System.out.println("object.queue 的消息: " + message);
}
```

🧠 **说明：**

- Spring AMQP 会自动将接收到的 JSON 消息反序列化为 `Map`；
- 如果你有自定义实体类，也可以直接接收为对应对象。



#  MQ 高级机制

## 一、发送者重连机制（Retry 机制）

> 当由于**网络波动**或**MQ不可用**导致连接失败时，可开启**自动重试机制**。

### 配置示例

```
spring:
  rabbitmq:
    connection-timeout: 1s # 设置连接MQ的超时时间
    template:
      retry:
        enabled: true        # 开启超时重试机制
        initial-interval: 1000ms # 第一次失败后的等待时间
        multiplier: 1        # 下次等待时长倍数（等待时间 = 上次等待时间 * multiplier）
        max-attempts: 3      # 最大重试次数
```

### 💡 说明

- **enabled**：是否启用重试。
- **initial-interval**：第一次失败后等待多久再重试。
- **multiplier**：倍数因子，每次重试的等待间隔递增。
- **max-attempts**：最多重试几次。

### ⚠️ 注意

- 该机制是**阻塞式**的，会影响性能。
- 一般用于 **测试环境** 或 **对可靠性要求极高** 的场景。
- 在生产环境通常通过 **异步重试** 或 **补偿机制（定时任务 + 消息表）** 实现更优雅的重发。

------

## 二、发送者确认机制（Publisher Confirm）

### 1、机制简介

> **发送者确认机制（Publisher Confirm）** 是 RabbitMQ 提供的一种消息可靠投递机制，
>  用于确保生产者发送的消息能可靠地从**生产者 → 交换机（Exchange） → 队列（Queue）**，
>  并可在任一环节失败时得到回调通知，防止消息丢失。

------

### 2、机制组成

发送者确认机制包含 **两大回调机制**：

| 回调类型            | 触发时机                                | 作用                                 |
| ------------------- | --------------------------------------- | ------------------------------------ |
| **ConfirmCallback** | 消息是否成功到达 **交换机（Exchange）** | 确认消息已被 MQ 接收                 |
| **ReturnCallback**  | 消息是否被成功路由到 **队列（Queue）**  | 检测消息是否投递失败（如路由键错误） |

这两部分协同工作，构成完整的消息投递确认链路：

```
生产者 → 交换机（ConfirmCallback）
       → 队列（ReturnCallback）
```
<img width="1403" height="751" alt="发送者确认" src="https://github.com/user-attachments/assets/cb67c4b5-5489-4dc3-9773-34baa133290c" />

------

### 3、配置示例

```
spring:
  rabbitmq:
    publisher-confirm-type: correlated  # 开启 Confirm 机制（异步回调）
    publisher-returns: true             # 开启 Return 机制（路由失败回调）
    template:
      mandatory: true                   # 必须设置为 true，否则 ReturnCallback 不会触发
```

------

### 4、ConfirmCallback — 交换机确认机制

> 用于确认消息是否成功到达交换机（Exchange）。

#### 说明

- `publisher-confirm-type` 参数控制 confirm 模式。
- 有三种取值：

| 模式         | 说明                         |
| ------------ | ---------------------------- |
| `none`       | 关闭 confirm 机制            |
| `simple`     | 同步阻塞等待确认（性能低）   |
| `correlated` | **异步回调**接收确认（推荐） |

#### 使用示例

```
@Test
public void testConfirmCallback() throws InterruptedException {
    // 1. 创建唯一 ID
    CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());

    // 2. 添加确认回调
    correlationData.getFuture().addCallback(new ListenableFutureCallback<CorrelationData.Confirm>() {
        @Override
        public void onSuccess(CorrelationData.Confirm result) {
            if (result.isAck()) {
                log.debug("✅ 消息成功到达交换机！");
            } else {
                log.error("❌ 消息未能到达交换机！");
            }
        }

        @Override
        public void onFailure(Throwable ex) {
            log.error("❌ 消息发送异常: {}", ex.getMessage());
        }
    });

    // 3. 发送消息
    rabbitTemplate.convertAndSend("fz.direct", "red", "Hello, MQ!", correlationData);

    // 4. 避免测试提前结束
    Thread.sleep(2000);
}
```

------

### 5、ReturnCallback — 路由确认机制

> 用于确认消息是否成功被交换机**路由到队列**。
>  若找不到对应队列（如 routingKey 错误），则触发该回调。

####  配置与使用

```
@Slf4j
@Configuration
@RequiredArgsConstructor
public class RabbitReturnConfig {

    private final RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init() {
        // 配置 ReturnCallback
        rabbitTemplate.setReturnsCallback(returned -> {
            log.error("❌ 消息未能路由到队列！");
            log.debug("exchange: {}", returned.getExchange());
            log.debug("routingKey: {}", returned.getRoutingKey());
            log.debug("replyCode: {}", returned.getReplyCode());
            log.debug("replyText: {}", returned.getReplyText());
            log.debug("message: {}", returned.getMessage());
        });
    }
}
```

> ⚠️ 注意：
>
> - 每个 `RabbitTemplate` **只能绑定一个 ReturnCallback**。
> - 若未设置 `mandatory: true`，路由失败的消息会被直接丢弃，不会触发回调！

## 三、RabbitMQ 消息持久化

### 1、持久化机制概述

- **RabbitMQ 的消息默认是持久化的（前提是队列持久化 + 消息持久化）**。
- **持久化数据会写入磁盘**，即使 RabbitMQ 重启后也能恢复。
- **临时消息（非持久化消息）只存储在内存中**，不会写入磁盘。

------

### 2、消息阻塞与持久化的关系

| 消息类型   | 是否写入磁盘 | 是否占用内存       | 是否可能造成阻塞       |
| ---------- | ------------ | ------------------ | ---------------------- |
| 持久化消息 | ✅ 是         | ❌ 否（主要在磁盘） | ❌ 不会阻塞             |
| 临时消息   | ❌ 否         | ✅ 是（存内存）     | ⚠️ 容易阻塞（内存满时） |

> 💡 **阻塞的原因**：非持久化消息堆积在内存中，若消费者消费速度慢，就可能触发内存水位限流机制（内存占用过高）。

------

### 3、Lazy Queue（惰性队列）
<img width="1368" height="508" alt="lazyQueue" src="https://github.com/user-attachments/assets/6c5bfc7a-43ba-4103-8b4a-363fa807880d" />

- **Lazy Queue**：消息**默认直接存储在磁盘上**，不进入内存。
- 仅在消息即将被消费时，才会从磁盘**读取到内存**。
- 可极大减少内存压力，适合**高堆积场景**（如日志、IoT、监控数据等）。

------

### 4、RabbitMQ 3.12 之后的变化

- **RabbitMQ 3.12+ 默认队列模式已经是 Lazy Queue 模式**。

  > 即使未显式设置，队列中的消息也**默认写入磁盘**，不再占用内存。



## **四、消费者确认机制（Consumer Acknowledgement）**
<img width="1386" height="703" alt="消费者确认" src="https://github.com/user-attachments/assets/7e73a8e9-40c5-4a75-bcf8-0964c85f4bb1" />

> 消费者从队列中取出消息后，需要通过回执（acknowledgement）告诉 RabbitMQ 消息的处理结果。

------

### 1、回执类型

| 回执类型   | 含义                   | RabbitMQ行为                    |
| ---------- | ---------------------- | ------------------------------- |
| **ack**    | 消息成功处理           | RabbitMQ 将消息从队列中**删除** |
| **nack**   | 消息处理失败（可重试） | RabbitMQ **重新投递**该消息     |
| **reject** | 消息处理失败并拒绝     | RabbitMQ **删除消息**，不会重试 |

------

### 2、ACK 处理模式（Acknowledge Mode）

| 模式       | 描述                                                   | 特点                                           |
| ---------- | ------------------------------------------------------ | ---------------------------------------------- |
| **none**   | 不做确认，消息一投递即删除                             | 不安全，容易造成消息丢失，不建议使用           |
| **manual** | 手动确认，业务代码中显式调用 `ack` / `nack` / `reject` | 灵活，可精确控制消息确认时机，但业务入侵性较强 |
| **auto**   | 自动确认，Spring AMQP 利用 AOP 自动判断                | 安全且易用，推荐使用                           |

------

### 3、`auto` 模式的自动回执逻辑

Spring AMQP 会对消费者方法进行环绕增强，根据执行结果自动返回不同回执：

| 情况                     | 自动返回结果 | 行为说明                   |
| ------------------------ | ------------ | -------------------------- |
| 业务逻辑执行成功         | **ack**      | 消息正常处理，删除消息     |
| 业务异常（可重试性错误） | **nack**     | RabbitMQ 会重新投递消息    |
| 消息格式或校验异常       | **reject**   | 消息被拒绝并删除，不再重试 |

------

### 4、配置示例

```
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto  # 自动确认模式
```

## 五、消费者重试机制

### 作用

防止消费者**处理消息失败时无限重试**，导致**队列消息频繁重新投递**（可能每秒上千次，造成性能压力）。
 开启“本地重试机制”后，失败重试仅发生在**消费者本地**，不会重复投递到队列中。

> ⚠️ 注意：默认情况下，超过最大重试次数后，消息会**直接丢弃**。

------

###  基本配置

```
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true           # 开启消费者失败重试
          initial-interval: 1000ms # 初始失败等待时间：1秒
          multiplier: 1           # 每次重试等待时间倍率
          max-attempts: 3         # 最大重试次数
          stateless: true         # 是否无状态（true：无事务；false：有事务）
```

#### 说明：

- **enabled**：开启重试机制。
- **initial-interval**：第一次重试的等待时长。
- **multiplier**：每次重试等待的倍数（例如 1.5 表示递增）。
- **max-attempts**：最大重试次数（超过即触发`MessageRecoverer`策略）。
- **stateless**：若消费者方法涉及事务，需设为`false`。

------

### MessageRecoverer（消息恢复策略）

当**重试次数耗尽**后，Spring AMQP 会调用 `MessageRecoverer` 接口的实现类来处理失败的消息。

三种常用实现：

| 实现类                               | 行为说明                                            |
| ------------------------------------ | --------------------------------------------------- |
| **RejectAndDontRequeueRecoverer**    | 重试耗尽后，直接 `reject`，**丢弃消息**（默认策略） |
| **ImmediateRequeueMessageRecoverer** | 重试耗尽后，返回 `nack`，**消息重新入队**           |
| **RepublishMessageRecoverer**        | 重试耗尽后，**将失败消息投递到指定交换机/队列**     |

------

### 示例 1：ImmediateRequeueMessageRecoverer

> 重试失败后让消息**重新回到队列**，由其他消费者或相同消费者再次处理。

```
@Configuration
public class RabbitRetryConfig {

    @Bean
    public MessageRecoverer messageRecoverer() {
        // 重试耗尽后重新入队
        return new ImmediateRequeueMessageRecoverer();
    }
}
```

**效果**：

- 消息在本地重试失败后，会重新回到队列中；
- RabbitMQ 会重新分配该消息（可被相同或其他消费者再次消费）；
- 适合 **临时异常**（如网络抖动、外部接口超时等）。

------

### 示例 2：RepublishMessageRecoverer

> 重试失败后，将错误消息投递到**错误交换机**（error.direct），供后续人工分析。

```
@Configuration
public class ErrorMessageConfiguration {

    @Bean
    public DirectExchange errorExchange() {
        return new DirectExchange("error.direct");
    }

    @Bean
    public Queue errorQueue() {
        return new Queue("error.queue");
    }

    @Bean
    public Binding errorBinding(Queue errorQueue, DirectExchange errorExchange) {
        return BindingBuilder.bind(errorQueue).to(errorExchange).with("error");
    }

    @Bean
    public MessageRecoverer messageRecoverer(RabbitTemplate rabbitTemplate) {
        // 将失败消息转发到 error.direct 交换机中
        return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
    }
}
```

**效果**：

- 消息重试3次后仍失败，会被发送到 `"error.direct"` → `"error.queue"`；
- 可在此队列中记录日志、报警或人工介入排查问题。
- 常用于 **重要业务**，以防止消息丢失。

------

### 对比

| 策略类型                           | 行为             | 使用场景                 |
| ---------------------------------- | ---------------- | ------------------------ |
| `RejectAndDontRequeueRecoverer`    | 丢弃消息         | 非关键消息               |
| `ImmediateRequeueMessageRecoverer` | 重新入队         | 临时性错误（如网络异常） |
| `RepublishMessageRecoverer`        | 投递到错误交换机 | 关键业务、日志追踪       |

## 六、幂等性（Idempotency）

### 概念说明

幂等性：

> **无论同一条消息被消费多少次，最终结果都应一致**。

RabbitMQ 在网络抖动、消费者宕机或消息重试等情况下，可能导致**同一消息被多次消费**。
 因此需要为消息设置唯一 ID，用于判定消息是否已被处理。

------

### 1、在消息转换器中自动生成消息 ID

在配置类中开启 **自动生成 MessageId** 功能：

```
@Bean
public MessageConverter messageConverter(){
    Jackson2JsonMessageConverter jsonMessageConverter = new Jackson2JsonMessageConverter();
    jsonMessageConverter.setCreateMessageIds(true); // 自动为每条消息生成唯一 ID
    return jsonMessageConverter;
}
```
<img width="1239" height="803" alt="消息ID" src="https://github.com/user-attachments/assets/175d364f-92ca-4b21-853a-2b558123f4ac" />

#### ✨ 说明：

- `Jackson2JsonMessageConverter` 用于将对象自动转换为 JSON 格式消息；
- `setCreateMessageIds(true)`：发送消息时，系统自动为每条消息分配唯一 `MessageId`；
- 消费者可根据 `MessageId` 判断消息是否重复处理。

------

###  2、消费者端接收消息并打印 MessageId

消费者方法需接收 `Message` 类型（而不是直接接收对象），
 这样才能读取消息属性（如 ID、Header、Body）。

```
@RabbitListener(queues = "object.queue")
public void listenObjectQueue(Message message) {
    log.info("消息的ID为：【{}】", message.getMessageProperties().getMessageId());
    log.info("消息为：【{}】", new String(message.getBody()));
    System.out.println("object.queue的消息: " + message);
}
```
<img width="1082" height="307" alt="查看消息ID" src="https://github.com/user-attachments/assets/8162c326-2b3a-4f8c-90eb-a60a6b1209d7" />
# RabbitMQ 消息延迟机制

------

## 一、死信交换机（DLX）实现延迟消息

### 原理说明

当消息在队列中超时未被消费、被拒绝（`reject` / `nack` 且不重新入队）、或队列达到最大长度时，会被转发到 **死信交换机（DLX）**。

通过这种机制，可以模拟“消息延迟消费”的效果。
<img width="1500" height="712" alt="死信交换机" src="https://github.com/user-attachments/assets/5abfb003-3b15-4cd2-bae1-f0b4d37e2e0e" />

------

### 核心配置

#### 生产者发送消息并设置过期时间（TTL）

```
@Test
public void TestDlx() {
    rabbitTemplate.convertAndSend("normal.direct", "dlx", "hello, spring amqp!", new MessagePostProcessor() {
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            // 设置消息过期时间（单位：毫秒）
            message.getMessageProperties().setExpiration("10000");
            return message;
        }
    });
}
```

------

#### 普通交换机与死信交换机配置

```
@Configuration
public class NormalConfiguration {

    // 普通交换机
    @Bean
    public DirectExchange normalExchange() {
        return ExchangeBuilder.directExchange("normal.direct").build();
    }

    // 普通队列，绑定死信交换机
    @Bean
    public Queue normalQueue() {
        return QueueBuilder
                .durable("normal.queue")
                .deadLetterExchange("dlx.direct") // 绑定死信交换机
                .build();
    }

    // 普通队列与普通交换机绑定
    @Bean
    public Binding directBinding(Queue normalQueue, DirectExchange normalExchange) {
        return BindingBuilder.bind(normalQueue)
                .to(normalExchange)
                .with("dlx"); // routingKey 要与死信交换机一致
    }
}
```

------

#### 死信交换机监听消费者

```
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "dlx.queue"),
        exchange = @Exchange(name = "dlx.direct", type = ExchangeTypes.DIRECT),
        key = {"dlx"}
))
public void listenDlxQueue(String message) {
    System.out.println("接收到 dlx.queue 的死信消息：" + message);
}
```

✅ **效果**：
 消息发送到普通队列 → 超过 10 秒未消费 → 转发到死信交换机 → 被消费者监听到。

------

## 二、延迟插件实现消息延迟

> 使用官方插件 **rabbitmq-delayed-message-exchange** 实现更灵活的延迟消息。

------

### 插件安装步骤（Docker 环境）

1️⃣ 下载插件
 👉 [Releases · rabbitmq/rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases)

2️⃣ 查找插件卷路径

```
docker volume inspect mq-plugins
```

3️⃣ 进入路径并复制插件文件进去

```
cd /var/lib/docker/volumes/mq-plugins/_data
```

4️⃣ 启用插件

```
docker exec -it mq rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```
<img width="1846" height="934" alt="安装插件" src="https://github.com/user-attachments/assets/9390f1b0-07bb-4e27-9e89-b9a40770180c" />

------

### 声明延迟交换机（delayed = true）

```
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "delay.queue", durable = "true"),
        exchange = @Exchange(name = "delay.direct", delayed = "true"), // 延迟交换机
        key = "delay"
))
public void listenDelayMessage(String msg) {
    log.info("接收到 delay.queue 的延迟消息：{}", msg);
}
```

------

### 发送延迟消息

```
@Test
void testPublisherDelayMessage() {
    String message = "hello, delayed message";

    rabbitTemplate.convertAndSend("delay.direct", "delay", message, new MessagePostProcessor() {
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            // 设置延迟时间（单位：毫秒）
            message.getMessageProperties().setDelay(5000);
            return message;
        }
    });
}
```

✅ **效果**：
 **发送后 5 秒消息才会被投递到 `delay.queue` 并被消费。**





#  RabbitMQ 公共封装

## 一、设计背景

在多模块项目中，例如微服务架构，常常会有多个服务需要使用消息队列（MQ）进行异步通信或解耦。但同时，也有一些模块并不依赖 MQ。
 为了**降低耦合、提高可复用性**，我们在 `hm-common` 模块中进行以下设计：

- **提供统一的 MQ 工具类**（`RabbitMqHelper`），简化消息发送逻辑；
- **提供必要依赖（scope = provided）**，保证编译不报错；
- **共享 MQ 配置到 Nacos**，保证各服务配置一致；
- **允许按需引入 MQ 依赖**，让不使用 MQ 的模块保持轻量化。

------

## 二、依赖选择

在 `hm-common` 中引入的依赖如下：

```
<!-- AMQP 依赖 -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-amqp</artifactId>
    <scope>provided</scope>
</dependency>

<!-- Spring 整合 RabbitMQ -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <scope>provided</scope>
</dependency>

<!-- JSON 处理依赖 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <scope>provided</scope>
</dependency>
```

### ✅ 为什么使用 `provided` 而不是 `starter`

| 对比项       | `provided`                         | `starter`                      |
| ------------ | ---------------------------------- | ------------------------------ |
| **作用**     | 编译时依赖，运行时由使用者自行引入 | 自动引入所有 MQ 相关依赖与配置 |
| **灵活性**   | ✅ 使用者可按需决定是否引入 MQ      | ❌ 强制包含 MQ 依赖             |
| **打包结果** | 不会被打入最终 jar                 | 会被打入 jar 包                |
| **适用场景** | 公共模块、底层封装库               | 实际业务模块                   |

> **总结：**
>  在公共模块中使用 `provided`，可以让公共类在编译时识别 `RabbitTemplate` 等类型而不报错，但不会强制打包进最终运行环境，避免不必要的依赖。

------

## 三、共享配置（Nacos）

将 MQ 连接配置集中放入 **Nacos 配置中心**，方便所有模块统一读取。

### Nacos 配置示例

```
spring:
  rabbitmq:
    host: ${fz.mq.host:192.168.195.131}   # 主机名
    port: ${fz.mq.port:5672}              # 端口
    virtual-host: ${fz.mq.vhost:/fz}      # 虚拟主机
    username: ${fz.mq.un:fz}              # 用户名
    password: ${fz.mq.pw:123}             # 密码
```

- `${fz.mq.xxx}` 代表配置项可被覆盖；

- `:` 后面的值为默认值；

- 配置上传至 Nacos 后，各微服务只需在 `bootstrap.yml` 中声明：

  ```
  spring:
    cloud:
      nacos:
        config:
          server-addr: localhost:8848
          file-extension: yaml
          shared-configs:
            - data-id: rabbitmq-config.yaml
  ```

------

## 四、工具类封装 —— `RabbitMqHelper`

通过封装 `RabbitTemplate`，简化消息发送逻辑。

```
import cn.hutool.core.lang.UUID;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.util.concurrent.ListenableFutureCallback;

@Slf4j
@RequiredArgsConstructor
public class RabbitMqHelper {

    private final RabbitTemplate rabbitTemplate;

    /**
     * 发送普通消息
     */
    public void sendMessage(String exchange, String routingKey, Object msg){
        log.debug("准备发送消息，exchange:{}, routingKey:{}, msg:{}", exchange, routingKey, msg);
        rabbitTemplate.convertAndSend(exchange, routingKey, msg);
    }

    /**
     * 发送延迟消息
     */
    public void sendDelayMessage(String exchange, String routingKey, Object msg, int delay){
        rabbitTemplate.convertAndSend(exchange, routingKey, msg, message -> {
            message.getMessageProperties().setDelay(delay);
            return message;
        });
    }

    /**
     * 发送消息并确认（带重试机制）
     */
    public void sendMessageWithConfirm(String exchange, String routingKey, Object msg, int maxRetries){
        log.debug("准备发送消息，exchange:{}, routingKey:{}, msg:{}", exchange, routingKey, msg);
        CorrelationData cd = new CorrelationData(UUID.randomUUID().toString(true));

        cd.getFuture().addCallback(new ListenableFutureCallback<>() {
            int retryCount;

            @Override
            public void onFailure(Throwable ex) {
                log.error("处理ack回执失败", ex);
            }

            @Override
            public void onSuccess(CorrelationData.Confirm result) {
                if (result != null && !result.isAck()) {
                    log.debug("消息发送失败，收到nack，已重试次数：{}", retryCount);
                    if (retryCount >= maxRetries) {
                        log.error("消息发送重试次数耗尽，发送失败");
                        return;
                    }
                    CorrelationData cd = new CorrelationData(UUID.randomUUID().toString(true));
                    cd.getFuture().addCallback(this);
                    rabbitTemplate.convertAndSend(exchange, routingKey, msg, cd);
                    retryCount++;
                }
            }
        });

        rabbitTemplate.convertAndSend(exchange, routingKey, msg, cd);
    }
}
```

------

## 五、封装优点

| 功能             | 优点                                                         |
| ---------------- | ------------------------------------------------------------ |
| **模块解耦**     | 公共模块仅提供接口，不强制引入依赖                           |
| **易于维护**     | MQ 配置信息统一存储在 Nacos 中                               |
| **代码复用**     | `RabbitMqHelper` 封装常见的发送模式                          |
| **增强可靠性**   | 带确认机制与重试功能                                         |
| **降低心智负担** | 开发者只需调用 `sendMessage()`、`sendDelayMessage()` 即可使用 MQ |

------

## 六、使用示例（在需要 MQ 的模块中）

1. 引入依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

1. 注入工具类并使用：

```
@Autowired
private RabbitMqHelper rabbitMqHelper;

rabbitMqHelper.sendMessage("order.exchange", "order.create", orderData);
```

------

✅ **总结**

- `hm-common` 仅提供**接口和工具类定义**，依赖范围为 `provided`；
- Nacos 负责**配置集中管理**；
- 实际业务模块按需**引入 starter 并使用封装工具类**；
- 这种设计兼顾了 **灵活性、复用性和低耦合性**，是企业级项目中常见的最佳实践。
