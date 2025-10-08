# SpringCloud

## MyBatis-Plus 基础配置

### 1. 引入依赖

在 `pom.xml` 中添加 MyBatis-Plus 的 starter：

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.7</version>
</dependency>
```

### 2. 编写实体类（Entity）

例如 `User` 实体：

```
@Data
@TableName("user") // 对应数据库中的表名
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

### 3. 编写 Mapper 接口

继承 MyBatis-Plus 提供的 `BaseMapper<T>`：

```
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

### 4.相关注解

#### 1. `@TableName`

- 作用：指定实体类对应的表名。

```
@TableName("user")  // 对应数据库表 user
public class User {
    private Long id;
    private String name;
}
```

------

#### 2. `@TableId`

- 作用：标记主键字段。
- 常用属性：
  - `value`：数据库字段名。
  - `type`：主键生成策略（`IdType` 枚举）。

```
@TableId(value = "id", type = IdType.AUTO) // 自增主键
private Long id;
```

👉 主键策略 `IdType` 常见值：

- `AUTO` → 数据库自增。
- `ASSIGN_ID` → 雪花算法（默认）。
- `ASSIGN_UUID` → UUID。
- `INPUT` → 手动输入。

------

#### 3. `@TableField`

- 作用：指定非主键字段与数据库字段的映射关系。
- 常用属性：
  - `value`：数据库字段名。
  - `exist`：是否为数据库表字段，`false` 表示实体类中存在但数据库不存在的字段。

```
@TableField("user_name")
private String name;

@TableField(exist = false) // 数据库中没有该字段
private String extraInfo;
```

------

#### 4. `@TableLogic`

- 作用：逻辑删除标记。
- 数据库字段值通常 0（未删除）、1（已删除）。

```
@TableLogic
private Integer deleted;
```

使用后：

- `deleteById` → 实际执行的是 **更新 deleted=1**，而不是物理删除。
- 查询时自动带上 `deleted=0` 条件。

------

#### 5. `@Version`

- 作用：乐观锁字段。
- 通常配合 `Integer` 或 `Long` 使用。

```
@Version
private Integer version;
```

使用后：

- `updateById` 时会自动带上 `version` 条件，保证数据一致性。

### 5.条件构造器（QueryWrapper）

```
void testQueryByIds() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<User>()
            .select("id","username","info","balance")
            .like("username","o")
            .ge("balance", 1000);
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

### 6.自定义sql

<img width="1317" height="745" alt="image" src="https://github.com/user-attachments/assets/d3913efa-3680-47ae-87e5-9f989e75cdcd" />


#### 1. 调用方法

```
@Test
void testUpdateById() {
    // 要更新的 id 列表
    List<Long> ids = List.of(1L, 2L, 4L);
    int amount = 200;

    // 构造条件
    QueryWrapper<User> queryWrapper = new QueryWrapper<User>()
            .in("id", ids);

    // 调用自定义 SQL
    userMapper.updateBalance(queryWrapper, amount);
}
```

------

#### 2. Mapper 接口

```
public interface UserMapper extends BaseMapper<User> {

    int updateBalance(@Param("ew") QueryWrapper<User> queryWrapper,
                      @Param("amount") int amount);

}
```

------

#### 3. Mapper XML

```
<update id="updateBalance">
    UPDATE user 
    SET balance = balance - #{amount}
    ${ew.customSqlSegment}
</update>
```

------

#### 4. 注意事项 

1. **参数命名要一致**

   - 接口里是 `@Param("amount") int amount` → XML 里要写 `#{amount}`

2. **必须是ew**	

   ```
   @Param("ew") QueryWrapper<User> queryWrapper
   ```


### 7. Service 接口与实现

#### 1. 定义 Service 接口

```
public interface IUserService extends IService<User> {
    // 如果有业务扩展，可以在这里定义额外的方法
}
```

👉 说明：

- `IService<T>` 是 MyBatis-Plus 提供的 **通用 Service 接口**，封装了 `CRUD` 方法（如 `save、getById、list、updateById、removeById` 等）。
- 我们的业务接口 **继承 IService<User>**，就自动拥有这些方法。

------

#### 2. 定义 Service 实现类

```
@Service
public class IUserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {
    // 继承了 ServiceImpl，就自动获得了通用方法的实现
}
```

👉 说明：

- `ServiceImpl<M, T>`：
  - `M` 表示 Mapper（这里是 `UserMapper`）
  - `T` 表示实体类（这里是 `User`）
- 继承后就不需要手写实现 `save()`、`getById()` 等方法。
- 如果有特殊业务逻辑，可以在这里拓展。

------

#### 3. 使用 Service 插入数据

```
@Test
void testInsert() {
    User user = new User();

    // 如果 User 实体类里主键是自增
    // @TableId(value = "id", type = IdType.AUTO)
    // 就不要手动设置 id，数据库会自动生成
    // user.setId(5L); // ❌ 不需要

    user.setUsername("lisi");
    user.setPassword("123");
    user.setPhone("18688990011");
    user.setBalance(200);
    user.setInfo("{\"age\": 24, \"intro\": \"英文老师\", \"gender\": \"female\"}");
    user.setCreateTime(LocalDateTime.now());
    user.setUpdateTime(LocalDateTime.now());

    // 调用 Service 提供的 save 方法
    iUserService.save(user);
}
```

👉 说明：

- `save()` 方法由 `IService` 提供，无需写 SQL，就能完成插入操作。
- 插入后，如果数据库主键是自增，会自动回填到 `user.id`。

### 8.例子

####  1、批量查询用户接口（stream）

```
@RequiredArgsConstructor
@RestController
@RequestMapping("/user")
public class UserController {

    private final IUserService iUserService;

    @GetMapping("/batch")
    @ApiOperation("根据id批量查询用户接口")
    public List<UserVO> queryUserByIds(
            @ApiParam("用户id集合") 
            @RequestParam("ids") List<Long> ids) {
        
        // 根据ID集合批量查询用户
        List<User> userList = iUserService.listByIds(ids);

        // 将 User 转换为 UserVO
        return userList.stream().map(user -> {
            UserVO vo = new UserVO();
            BeanUtils.copyProperties(user, vo); // 属性拷贝
            return vo;
        }).collect(Collectors.toList());
    }
}
```

1. **`@RequiredArgsConstructor`**
   - Lombok 注解，自动生成构造函数，把 `final` 的字段注入进来。
   - 这里 `private final IUserService iUserService;` 会自动通过构造注入，不需要写 `@Autowired`。

------

#### 2、Lambda 条件查询用户

```
public List<User> queryUsers(String name, Integer status, Integer minBalance, Integer maxBalance) {
    return lambdaQuery()
            .like(name != null, User::getUsername, name)     // 用户名模糊查询
            .eq(status != null, User::getStatus, status)     // 精确匹配状态
            .ge(minBalance != null, User::getBalance, minBalance) // 大于等于 最小余额
            .le(maxBalance != null, User::getBalance, maxBalance) // 小于等于 最大余额
            .list();  // 执行查询返回 List<User>
}
```

这是 MyBatis-Plus 的链式查询写法（`lambdaQuery()`）：

1. **`like(name != null, User::getUsername, name)`**
   - 如果 `name` 不为 `null`，就拼接 SQL: `username like '%name%'`。
   - 否则跳过该条件。
2. **`eq(status != null, User::getStatus, status)`**
   - 精确匹配用户状态。
3. **`ge(minBalance != null, User::getBalance, minBalance)`**
   - 大于等于最小余额（Greater or Equal）。
4. **`le(maxBalance != null, User::getBalance, maxBalance)`**
   - 小于等于最大余额（Less or Equal）。
5. **`list()`**
   - 执行 SQL，返回 `List<User>`。



## Docker常用命令

<img width="1269" height="511" alt="image" src="https://github.com/user-attachments/assets/ce8de4f1-567d-4cf0-8357-2ea24c6c3a93" />


| 功能                           | 命令                                                         | 示例                                                   |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| 1. 查看本地镜像                | `docker images`                                              | `docker images`                                        |
| 2. 拉取镜像                    | `docker pull <镜像名>:<tag>`                                 | `docker pull nginx:latest`                             |
| 3. 运行容器（后台 + 端口映射） | `docker run -d -p <宿主端口>:<容器端口> --name <容器名> <镜像名>` | `docker run -d -p 8080:80 --name mynginx nginx:latest` |
| 4. 查看运行中的容器            | `docker ps`                                                  | `docker ps`                                            |
| 5. 查看所有容器（包括已停止）  | `docker ps -a`                                               | `docker ps -a`                                         |
| 6. 进入容器                    | `docker exec -it <容器ID或名称> /bin/bash`                   | `docker exec -it mynginx /bin/bash`                    |
| 7. 查看容器日志                | `docker logs -f <容器ID或名称>`                              | `docker logs -f mynginx`                               |
| 8. 停止容器                    | `docker stop <容器ID或名称>`                                 | `docker stop mynginx`                                  |
| 9. 删除容器                    | `docker rm <容器ID或名称>`                                   | `docker rm mynginx`                                    |
| 10. 删除镜像                   | `docker rmi <镜像ID或名称>`                                  | `docker rmi nginx:latest`                              |



## Linux 虚拟机环境安装 Docker + MySQL

### 1、 启动虚拟机 + Mobaxterm

👉 参考文档：[Linux环境搭建 - 飞书云文档](https://b11et3un53m.feishu.cn/wiki/FJAnwOhpIihMkLkOKQocdWZ7nUc)
 安装完虚拟机后，用 **MobaXterm** 远程连接。

------

### 2、 卸载旧版本 Docker（如有）

```
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine \
    docker-selinux
```

------

### 3、 安装依赖 & 配置 Docker YUM 源

```
# 安装依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加清华源镜像
sudo yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo

# 修改为清华源
sudo sed -i 's|https://download.docker.com|https://mirrors.tuna.tsinghua.edu.cn/docker-ce|g' /etc/yum.repos.d/docker-ce.repo

# 更新 yum 缓存
sudo yum makecache fast
```

------

### 4、 安装 Docker

```
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

------

### 5、 启动 Docker 并设置开机自启

```
# 启动
systemctl start docker

# 停止
systemctl stop docker

# 重启
systemctl restart docker

# 设置开机自启
systemctl enable docker

# 测试是否安装成功
docker ps
```

------

### 6、 配置 Docker 镜像加速

阿里云镜像加速： [容器镜像服务 ACR 控制台](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

```
# 创建配置目录
mkdir -p /etc/docker

# 写入配置
tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://docker.registry.cyou",
        "https://mirror.aliyuncs.com",
        "https://dockerproxy.com",
        "https://mirror.baidubce.com",
        "https://docker.m.daocloud.io",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.ustc.edu.cn",
        "https://mirror.iscas.ac.cn"
    ]
}
EOF

# 重新加载 & 重启 Docker
systemctl daemon-reload
systemctl restart docker
```

------

### 7、 安装 MySQL

### 创建 Docker 网络

```
docker network create hm-net
```

#### （方案一）带配置挂载安装 MySQL(SpringCloud学习项目里面有mysql文件)

```
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e TZ=Asia/Shanghai \
  -e MYSQL_ROOT_PASSWORD=123 \
  -v /root/mysql/data:/var/lib/mysql \
  -v /root/mysql/conf:/etc/mysql/conf.d \
  -v /root/mysql/init:/docker-entrypoint-initdb.d \
  --network hm-net \
  mysql
```

#### （方案二）简单安装 MySQL

```
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  mysql
```

#### 查看容器是否运行

```
docker ps
```

#### 设置 MySQL 自动重启

```
docker update mysql --restart=always
```

------

### 8、 连接 MySQL

👉 如果使用可视化软件（Navicat / DBeaver）：

- 主机地址：虚拟机 IP（ip addre获取ip地址）

- 端口：3306

- 用户：root

- 密码：安装时设置的（如 `123456`）

<img width="1272" height="582" alt="image" src="https://github.com/user-attachments/assets/2ec00d32-0517-4970-ba35-644ebc0bcf61" />


### 9、常见报错及解决办法

#### YUM 源找不到（Cannot find a valid baseurl for repo）

报错信息：

```
Cannot find a valid baseurl for repo: base/7/x86_64
```

👉 解决办法：换国内 YUM 源（如阿里云/清华源）
 🔗 参考：[已解决：Cannot find a valid baseurl for repo: base/7/x86_64 - CSDN博客](https://blog.csdn.net/weixin_52597907/article/details/141113817)

------

#### Docker 拉取镜像超时 / 无法访问 `registry-1.docker.io`

报错信息：

```
docker: Error response from daemon: Get “https://registry-1.docker.io/v2/“: net/http: request canceled while waiting for connection
```

👉 解决办法：配置国内 Docker 镜像加速源（如阿里云/DaoCloud/清华镜像）
 🔗 参考：[解决 docker 拉取镜像失败问题 - CSDN博客](https://blog.csdn.net/Liiiiiiiiiii19/article/details/142438122)



## Spring Cloud 微服务

### 补充：SpringCloud相关依赖

```
<!--spring cloud-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2021.0.3</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<!--spring cloud alibaba-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2021.0.4.0</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
<!--hutool工具包-->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.11</version>
</dependency>
```

### 1.微服务拆分原则对比表

| 原则                 | 合理拆分示例                                                | 不合理拆分示例                           |
| -------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| **按业务能力拆分**   | 订单服务、支付服务、用户服务                                | 登录接口单独拆成一个服务                 |
| **高内聚，低耦合**   | 订单服务内部包含下单、取消、查询；与支付服务仅通过 API 通信 | 订单服务直接调用支付数据库               |
| **数据独立**         | 每个服务有独立数据库（如订单库、用户库）                    | 所有服务共用一个大数据库                 |
| **单一职责**         | 支付服务只处理支付相关逻辑                                  | 一个服务既管支付又管库存                 |
| **匹配团队边界**     | 用户团队负责用户服务，订单团队负责订单服务                  | 一个团队同时维护多个复杂交织的服务       |
| **演进式拆分**       | 单体 → 模块化 → 微服务，逐步演进                            | 一开始就拆成几十个微服务                 |
| **避免过度细化**     | 用户服务、订单服务                                          | 把“用户登录”“用户注册”拆成两个服务       |
| **考虑非功能性需求** | 高频调用的下单服务单独拆分，便于扩展                        | 把低频的后台统计功能独立拆分，增加复杂度 |

### 2.微服务拆分

#### 1. 服务端口配置

- 每个微服务必须使用独立端口，避免冲突。
- **示例**：

```
server:
  port: 8081
```

#### 2. 微服务名称

- 每个微服务必须有唯一名称，用于注册中心、服务发现和日志追踪。
- **示例**：

```
spring:
  application:
    name: item-service
```

#### 3. 数据库配置与隔离

- 每个微服务使用独立数据库，避免数据干扰。
- 可选方案：
  1. 在同一个 MySQL 实例中建立多个数据库。
  2. 在不同虚拟机中安装多个 MySQL 实例。
- **示例**：

```
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/itemdb
    username: root
    password: 123
```

- **注意**：
  - 数据库名、账号、密码必须独立。
  - 不同微服务之间不要直接访问对方数据库表。

#### 4. 代码模块拆分

- 将原项目按业务拆分为独立模块。
- **示例**：
  - 用户服务：`user-service`
  - 商品服务：`item-service`
  - 订单服务：`order-service`
- 每个模块单独启动、单独配置数据库。

### 3.远程调用（ RestTemplate）

#### 1. 配置 RestTemplate

```
@Component
public class RestConfig {
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

- **作用**：向 Spring 容器中注入一个 `RestTemplate` Bean，方便其他类直接注入使用。
- **RestTemplate**：Spring 提供的一个 **HTTP 客户端工具类**，用来调用远程服务（GET、POST、PUT、DELETE 等）。

------

#### 2. 发起远程调用

```
ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
        "http://localhost:8081/items?ids={ids}", // 远程请求地址
        HttpMethod.GET,                          // 请求方式
        null,                                    // 请求实体（headers、body），此处为 null
        new ParameterizedTypeReference<List<ItemDTO>>() {}, // 返回类型（泛型安全）
        Map.of("ids", CollUtils.join(itemIds,","))          // 路径或查询参数
);
```

- **请求地址**：`http://localhost:8081/items?ids={ids}`
  - `{ids}` 是占位符，后续用 `Map` 传参替换。
- **HttpMethod.GET**：HTTP 方法为 GET。
- **null**：表示不需要请求体。
- **返回类型**：
  - 这里返回的是 `List<ItemDTO>`。
  - 因为泛型擦除问题，必须用 `new ParameterizedTypeReference<List<ItemDTO>>() {}` 指定。
- **Map.of("ids", CollUtils.join(itemIds,","))**：
  - 把 `itemIds` 拼接成 `"1,2,3"` 这样的字符串，填充到 `{ids}` 占位符中。

------

#### 3. 响应处理

```
if(!response.getStatusCode().is2xxSuccessful()){
    return ;
}
List<ItemDTO> items = response.getBody();
```

- **response.getStatusCode().is2xxSuccessful()**
  - 检查返回状态码是否是 2xx（200, 201, 204 …）。
  - 如果不是，直接返回（调用失败）。
- **response.getBody()**
  - 获取响应体（JSON 会自动转换成 `List<ItemDTO>`）。

## 微服务注册中心（Nacos）

### 1. 前置条件

- 已有 **MySQL 容器** 并能正常访问。
- 已有 **Nacos 镜像**（`docker pull nacos/nacos-server:v2.1.0-slim` 或 `docker load -i nacos.tar`）。

------

### 2. 创建 `custom.env`并上传到MobarXterm

新建文件 `./nacos/custom.env`（**custom.env放到nacos文件夹里面**），内容如下：

```
PREFER_HOST_MODE=hostname
MODE=standalone
SPRING_DATASOURCE_PLATFORM=mysql
MYSQL_SERVICE_HOST=192.168.195.131       # MySQL 容器/宿主机 IP
MYSQL_SERVICE_DB_NAME=nacos              # 数据库名
MYSQL_SERVICE_PORT=3306                  # MySQL 端口
MYSQL_SERVICE_USER=root                  # MySQL 用户
MYSQL_SERVICE_PASSWORD=123               # MySQL 密码
MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
```

⚠️ **说明**：

- 这个文件就是纯文本，可以自己手写。
- `MYSQL_SERVICE_HOST` 改成你自己的 MySQL IP。
- `MYSQL_SERVICE_PASSWORD` 改成你自己的 MySQL 密码。

------

### 3. 创建 `nacos.sql`（可以把文件建好上传）

Nacos 启动需要的表结构和默认账号，官方提供了 **mysql-schema.sql** 文件：

👉 **官方 GitHub 链接**：
 [Nacos mysql-schema.sql（GitHub）](https://github.com/alibaba/nacos/blob/develop/distribution/conf/mysql-schema.sql)

### 4. 启动 Nacos 容器

```
docker run -d \
--name nacos \
--env-file ./nacos/custom.env \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--restart=always \
--network hm-net \
nacos/nacos-server:v2.1.0-slim
```

------

### 5. 查看运行情况

日志：

```
docker logs -f nacos
```

<img width="1415" height="822" alt="image" src="https://github.com/user-attachments/assets/5bba7587-c7dc-497c-8529-6a9354392d4f" />


网络检查：

```
docker inspect nacos | grep -A 10 "Networks"
docker inspect mysql | grep -A 10 "Networks"
```

<img width="1057" height="505" alt="image" src="https://github.com/user-attachments/assets/740a3fa4-0748-48fc-8d22-498f8dbd6c1a" />


### 6. 登录控制台

- 地址：http://192.168.195.131:8848/nacos/（需要换成自己的虚拟机IP地址）
- 默认账号：`nacos`
- 默认密码：`nacos`

### 7. 服务注册与发现

#### 1. 添加依赖

在 Spring Boot 微服务中引入 Nacos 作为注册中心：

```
<!-- nacos 服务注册发现 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

#### 2. 配置 Nacos 地址

在 `application.yml` 中配置服务名称和 Nacos 地址：

```
spring:
  application:
    name: item-service # 服务名称，注册到 nacos 的名字
  cloud:
    nacos:
      server-addr: 192.168.195.131:8848 # nacos 服务器地址
```

#### 3. 服务注册

- 启动服务后，`item-service` 会自动注册到 Nacos 注册中心。
- 其他服务可以通过 **服务名** 而不是 IP 地址访问它。

------

#### 4. 多实例部署

在生产环境下，一个服务往往会运行多个实例。
 方式：可以在不同服务器上运行，也可以在同一台机器上用不同端口运行，比如：

```
server:
  port: 8081
server:
  port: 8082
```

这样，`item-service` 就会有多个实例注册到 Nacos。

<img width="1822" height="945" alt="image" src="https://github.com/user-attachments/assets/1491a3fe-2ab5-499f-8ad4-746fbfdfd41d" />


##### 为什么要多实例部署？

1. **高可用**
   - 如果某个实例宕机，其他实例可以继续提供服务，避免单点故障。
2. **负载均衡**
   - 请求可以分发到多个实例，减少单个实例的压力，提高整体并发能力。
3. **弹性扩展**
   - 根据流量情况动态增加或减少实例，节省资源，提升性能。
4. **灰度发布 / 蓝绿部署**
   - 新版本上线时，可以先发布一部分实例，逐步替换老版本，降低风险。

------

#### 5. 服务发现 & RestTemplate 优化

##### 手动服务发现 + 负载均衡

```
private final DiscoveryClient discoveryClient;

// 1.根据服务名称获取实例列表
List<ServiceInstance> instances = discoveryClient.getInstances("item-service");
if(instances.isEmpty()) return;

// 2.手写负载均衡（随机挑选一个实例）
ServiceInstance instance = instances.get(RandomUtil.randomInt(instances.size()));

// 3.利用 RestTemplate 发起请求
ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
        instance.getUri() + "/items?ids={ids}",
        HttpMethod.GET,
        null,
        new ParameterizedTypeReference<List<ItemDTO>>() {},
        Map.of("ids", CollUtils.join(itemIds, ","))
);

if(!response.getStatusCode().is2xxSuccessful()){
    return;
}
List<ItemDTO> items = response.getBody();
```

##### 为什么要优化 RestTemplate？

- **以前的写法**：只能写死 `http://localhost:8081/items`，不灵活。
- **优化后**：通过 **服务名** + **Nacos 注册中心**，动态获取实例，支持多实例 + 负载均衡。

###  OpenFeign 

#### 1. 引入依赖

```
<!-- OpenFeign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!-- 负载均衡器（替代 Ribbon） -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

✅ `openfeign`：声明式 HTTP 客户端，简化远程调用。
 ✅ `loadbalancer`：Spring Cloud 自带的负载均衡，支持多实例调用。

------

#### 2. 开启 Feign 功能

在 Spring Boot 启动类上添加：

```
@SpringBootApplication
@EnableFeignClients // 开启Feign客户端
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

------

#### 3. 定义 Feign 客户端接口

```
@FeignClient("item-service") // 声明要调用的微服务名称（与注册中心中的服务名一致）
public interface ItemClient {
    
    // 对应 item-service 的接口
    @GetMapping("/items")
    List<ItemDTO> queryByIds(@RequestParam("ids") Collection<Long> ids);
}
```

📌 说明：

- `@FeignClient("item-service")`：表示调用 `item-service` 微服务。
- `@GetMapping("/items")`：对应远程服务的 API 路径。
- 参数、返回值与远程接口保持一致。

------

#### 4. 使用 Feign 远程调用

在业务类中直接注入调用：

```
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ItemClient itemClient; // 注入 Feign 客户端

    public List<ItemDTO> getItems(List<Long> itemIds) {
        // 直接像调用本地方法一样
        return itemClient.queryByIds(itemIds);
    }
}
```

✅ OpenFeign 底层会自动构建 HTTP 请求，并通过 **负载均衡器** 调用 `item-service` 的对应接口。

#### 5. 使用连接池（OKHttp）

默认 Feign 使用 **JDK 原生 HttpURLConnection**，性能较低，建议改为 **OKHttp**，支持连接池、并发更高。

**添加依赖**

```
<!-- OKHttp 连接池 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

**开启 OKHttp**

```
feign:
  okhttp:
    enabled: true # 启用 OKHttp
```

#### 6. 公共模块抽取（hm-api）

参考链接：[day03-微服务01 - 飞书云文档](https://b11et3un53m.feishu.cn/wiki/R4Sdwvo8Si4kilkSKfscgQX0niB)

**目的**

- 避免各微服务重复定义 **DTO、VO、工具类、常量、接口**
- 提高微服务间的 **代码复用性和维护性**

##### 1.模块示例

**hm-api 模块 `pom.xml`**

```
<project>
    <modelVersion>4.0.0</modelVersion>

    <!-- 继承父工程，统一 groupId 和 version -->
    <parent>
        <groupId>com.heima</groupId>
        <artifactId>hmall</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>hm-api</artifactId>
</project>
```

✅ 说明：

- `hm-api` 继承了父工程 `hmall` 的 `groupId` 和 `version`。
- 父工程统一管理版本和依赖，子模块只需要关注自己的 `artifactId`。

------

##### 2. 调用方引入公共模块

在微服务模块 `pom.xml` 中添加依赖：

```
<dependency>
    <groupId>com.heima</groupId>
    <artifactId>hm-api</artifactId>
    <version>1.0.0</version>
</dependency>
```

✅ 注意：

- **groupId + artifactId + version** 必须与 `hm-api` 保持一致
- 多模块项目直接引用即可
- 单独 jar 需先 `mvn install` 安装到本地仓库

------

##### 3. Feign 客户端扫描包

如果公共模块中定义了 **Feign Client 接口**，调用方必须扫描包才能生效：

```
@SpringBootApplication
@EnableFeignClients(basePackages = "com.hmall.api.client") // 指向公共模块Feign接口包
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

✅ 说明：

- `basePackages` 指向公共模块中 Feign Client 接口的包路径
- 必须在启动类加上 `@EnableFeignClients`

#### 7.Feign 日志级别

Feign 提供 4 种日志级别（`feign.Logger.Level`）：

| 级别             | 描述                                                 |
| ---------------- | ---------------------------------------------------- |
| **NONE**（默认） | 不记录日志                                           |
| **BASIC**        | 记录请求方法、URL、响应状态码、执行时间              |
| **HEADERS**      | 在 BASIC 的基础上，记录请求和响应的头信息            |
| **FULL**         | 记录所有请求和响应的明细，包括请求体、响应体及元数据 |

------

##### 1.配置步骤

**① 定义日志配置类**

```
import feign.Logger;
import org.springframework.context.annotation.Bean;

public class DefaultFeignConfig {
    @Bean
    public Logger.Level feignLoggerLevel() {
        // 设置为 FULL，输出完整日志
        return Logger.Level.FULL;
    }
}
```

**② 启用 Feign 并指定配置**

```
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients(
    basePackages = "com.hmall.api.client", // 指定Feign接口所在包
    defaultConfiguration = DefaultFeignConfig.class // 指定日志配置
)
public class UserApplication {
}
```

------

##### 2.补充配置（application.yml）

除了 Java 配置类，还要在 `application.yml` 设置日志级别，否则日志可能不会输出：

```
logging:
  level:
    com.hmall: debug  # 指定Feign接口包的日志级别
```



## 网关

###  一、网关的作用

网关（Gateway）是微服务架构中的**统一入口**，主要功能包括：

- **路由转发**：根据路径将请求转发到对应的微服务。
- **负载均衡**：结合 `Nacos` 和 `LoadBalancer`，在多个实例之间自动分配请求。
- **统一鉴权**、**日志**、**跨域处理** 等（后续可扩展）。

------

### 二、引入依赖

```
<!-- 网关 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!-- Nacos 服务注册与发现 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!-- 负载均衡 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

📘 **说明：**

- `gateway`：核心组件，负责请求路由。
- `nacos-discovery`：从 Nacos 注册中心获取微服务列表。
- `loadbalancer`：支持微服务的多实例负载均衡。

------

###  三、配置文件 `application.yml`

```
server:
  port: 8080

spring:
  application:
    name: hm-gateway
  cloud:
    nacos:
      server-addr: 192.168.195.131:8848
    gateway:
      default-filters:
        - AddRequestHeader=token,*asjdklj1   #给所有经过网关的请求 自动添加一个请求头
      routes:
        - id: item-service   # 路由规则名称
          uri: lb://item-service  # lb表示启用负载均衡，后面是微服务在Nacos中的名称
          predicates:
            - Path=/items/**       # 匹配 /items 开头的请求
            - Path=/search/**      # 匹配 /search 开头的请求

        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/users/**,/addresses/**  # 匹配多个路径
```

------

###  四、路由匹配逻辑

| 条件                           | 匹配说明           | 转发目标              |
| ------------------------------ | ------------------ | --------------------- |
| `/items/**` 或 `/search/**`    | 商品相关接口       | `item-service` 微服务 |
| `/users/**` 或 `/addresses/**` | 用户、地址相关接口 | `user-service` 微服务 |

🧠 Gateway 会自动从 Nacos 获取对应微服务的实例地址，例如：

- `item-service` → `http://192.168.195.131:8081`
- `user-service` → `http://192.168.195.131:8082`

------

###  五、前端如何请求

前端只需请求 **网关地址**，无需关心后端微服务的实际端口或地址。

#### 示例一：请求商品服务

```
// Vue 前端 axios 请求示例
axios.get("http://localhost:8080/items/1")
     .then(res => {
         console.log("商品详情：", res.data);
     });
```

➡️ 网关根据规则自动转发到：

```
lb://item-service/items/1
（可能会路由到 8081 或 8083 等实例）
```

------

#### 示例二：请求用户登录

```
axios.post("http://localhost:8080/users/login", {
    username: "admin",
    password: "123456"
}).then(res => {
    console.log("登录结果：", res.data);
});
```

➡️ 自动转发到：

```
lb://user-service/users/login
```

### 六、**自定义全局过滤器（GlobalFilter）笔记**

#### **1. 代码示例**

```
@Component
public class MyGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求对象
        ServerHttpRequest request = exchange.getRequest();

        // 获取请求头信息
        HttpHeaders httpHeaders = request.getHeaders();
        System.out.println("httpHeaders = " + httpHeaders);

        // 继续执行过滤链
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        // 返回过滤器执行顺序，值越小优先级越高
        return 0;
    }
}
```

------

#### **2. 核心概念**

1. **GlobalFilter**
   - 全局过滤器，用于拦截 **所有经过网关的请求**。
   - 适用于统一处理请求/响应，例如日志记录、鉴权、限流等。
2. **Ordered**
   - 控制过滤器执行顺序的接口。
   - `getOrder()` 返回值越小，优先级越高（越先执行）。
3. **ServerWebExchange**
   - Spring WebFlux 的请求响应上下文对象。
   - 包含 `request`、`response` 等信息。
4. **GatewayFilterChain**
   - 过滤器链，调用 `chain.filter(exchange)` 可以继续执行后续过滤器。
   - 如果不调用，链条会被中断，后续请求不会继续。
5. **Mono<Void>**
   - WebFlux 异步响应类型，表示异步处理完成。
   - 这里返回 `chain.filter(exchange)`，表示继续链条处理。

### 例子：Spring Cloud Gateway + JWT 

#### 一、总体思路

🔹 通过 **Gateway 全局过滤器（GlobalFilter）** 拦截所有请求。
 🔹 对配置文件中设定的路径放行（如登录、商品查询）。
 🔹 其他路径则验证 **JWT Token** 是否有效。
 🔹 验证成功后将用户 ID 透传给下游微服务。
 🔹 微服务通过拦截器提取用户信息并保存到 `ThreadLocal`，实现用户上下文管理。

------

#### 二、AuthGlobalFilter（全局过滤器）

**位置**：`gateway/filters/AuthGlobalFilter.java`
 **作用**：拦截所有经过网关的请求并进行鉴权。

```
@Component
@RequiredArgsConstructor
public class AuthGlobalFilter implements GlobalFilter, Ordered {

    private final AuthProperties authProperties;
    private final JwtTool jwtTool;
    private final AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1️⃣ 获取请求对象
        ServerHttpRequest request = exchange.getRequest();

        // 2️⃣ 判断是否是放行路径
        if (isExclude(request.getPath().toString())) {
            return chain.filter(exchange);
        }

        // 3️⃣ 获取 Token
        String token = null;
        List<String> headers = request.getHeaders().get("authorization");
        if (headers != null && !headers.isEmpty()) {
            token = headers.get(0);
        }

        // 4️⃣ 校验 Token
        Long userId;
        try {
            userId = jwtTool.parseToken(token);
        } catch (UnauthorizedException e) {
            // 校验失败 → 返回 401
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }

        // 5️⃣ 将用户信息传递给下游服务
        Long finalUserId = userId;
        ServerWebExchange newExchange = exchange.mutate()
                .request(builder -> builder.header("userId", finalUserId.toString()))
                .build();

        // 6️⃣ 放行
        return chain.filter(newExchange);
    }

    private boolean isExclude(String path) {
        for (String pattern : authProperties.getExcludePaths()) {
            if (antPathMatcher.match(pattern, path)) {
                return true;
            }
        }
        return false;
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

##### 🔵 重点说明

| 元素             | 说明                                |
| ---------------- | ----------------------------------- |
| `GlobalFilter`   | 全局过滤器，作用于所有路由请求      |
| `Ordered`        | 控制执行顺序（值越小优先级越高）    |
| `AntPathMatcher` | 支持 `/users/**` 这种通配符路径匹配 |
| `Mono<Void>`     | 响应式编程返回类型                  |

------

#### 三、AuthProperties（鉴权配置类）

**作用**：定义哪些路径放行、不做 Token 校验。

```
@Data
@Component
@ConfigurationProperties(prefix = "hm.auth")
public class AuthProperties {
    private List<String> includePaths;
    private List<String> excludePaths;
}
```

**对应配置文件 `application.yml`**

```
hm:
  auth:
    excludePaths:
      - /search/**
      - /users/login
      - /items/**
      - /hi
```

🟩 **要点：**

- `excludePaths` → 不需要 Token 的接口路径
- `@ConfigurationProperties` → 自动绑定配置文件

------

#### 四、JwtTool（JWT 工具类）

**作用**：生成、解析、验证 JWT。

```
@Component
public class JwtTool {
    private final JWTSigner jwtSigner;

    public JwtTool(KeyPair keyPair) {
        this.jwtSigner = JWTSignerUtil.createSigner("rs256", keyPair);
    }

    // ✅ 生成 Token
    public String createToken(Long userId, Duration ttl) {
        return JWT.create()
                .setPayload("user", userId)
                .setExpiresAt(new Date(System.currentTimeMillis() + ttl.toMillis()))
                .setSigner(jwtSigner)
                .sign();
    }

    // ✅ 解析 Token
    public Long parseToken(String token) {
        if (token == null) throw new UnauthorizedException("未登录");

        JWT jwt = JWT.of(token).setSigner(jwtSigner);
        if (!jwt.verify()) throw new UnauthorizedException("无效的token");

        JWTValidator.of(jwt).validateDate(); // 校验过期

        Object userPayload = jwt.getPayload("user");
        if (userPayload == null) throw new UnauthorizedException("无效的token");

        return Long.valueOf(userPayload.toString());
    }
}
```

🟥 **要点：**

| 功能            | 说明                                  |
| --------------- | ------------------------------------- |
| 加密算法        | 使用 **RS256（RSA 非对称加密）**      |
| `createToken()` | 创建带有效期的 Token                  |
| `parseToken()`  | 校验签名、检查过期、提取用户信息      |
| 异常处理        | 抛出 `UnauthorizedException` 返回 401 |

------

#### 五、JwtProperties（JWT 配置类）

**作用**：加载 `.jks` 密钥文件、密码、别名、Token 有效期等信息。

```
@Data
@ConfigurationProperties(prefix = "hm.jwt")
public class JwtProperties {
    private Resource location;
    private String password;
    private String alias;
    private Duration tokenTTL = Duration.ofMinutes(10);
}
```

**配置文件示例：**

```
hm:
  jwt:
    location: classpath:hmall.jks
    alias: hmall
    password: hmall123
    tokenTTL: 30m
```

------

#### 六、SecurityConfig（安全配置类）

**作用**：加载密钥、生成 `KeyPair` 并注入到 `JwtTool`。

```
@Configuration
@EnableConfigurationProperties(JwtProperties.class)
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public KeyPair keyPair(JwtProperties properties) {
        KeyStoreKeyFactory keyStoreKeyFactory =
                new KeyStoreKeyFactory(
                        properties.getLocation(),
                        properties.getPassword().toCharArray());
        return keyStoreKeyFactory.getKeyPair(
                properties.getAlias(),
                properties.getPassword().toCharArray());
    }
}
```

🟦 **要点：**

- `.jks` 文件存放公私钥对
- `KeyPair` 注入 `JwtTool`
- `PasswordEncoder` 用于加密用户密码

------

#### 七、微服务中提取用户信息

##### 1️⃣ UserInfoInterceptor（用户信息拦截器）

```
public class UserInfoInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 1. 获取登录用户ID
        String userId = request.getHeader("userId");
        if (StrUtil.isNotBlank(userId)) {
            UserContext.setUser(Long.valueOf(userId));
        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) {
        // 2. 清理 ThreadLocal，防止内存泄漏
        UserContext.removeUser();
    }
}
```

------

##### 2️⃣ MvcConfig（拦截器配置）

```
@Configuration
@ConditionalOnClass(DispatcherServlet.class) // 微服务使用，网关不使用
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new UserInfoInterceptor());
    }
}
```

💡 **关键点：`@ConditionalOnClass(DispatcherServlet.class)`**

这行注解是核心。

- `@ConditionalOnClass` 表示：**只有当指定的类存在于当前项目的类路径中时，这个配置类才会生效**。
- `DispatcherServlet` 是 Spring MVC 的核心类，**只有在 MVC 环境（即普通微服务）中才存在**。
- Spring Cloud Gateway 是基于 WebFlux 的，使用的是 `DispatcherHandler`，**没有 DispatcherServlet 类**。



##### 3️⃣ spring.factories（自动装配）

**位置**：`META-INF/spring.factories`

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hmall.common.config.MvcConfig,\
```

让 `MvcConfig` 能够被 **Spring Boot 自动装配机制** 自动加载。

Spring Boot 启动时会自动扫描所有依赖中 `META-INF/spring.factories` 文件，
 并加载里面声明的 `EnableAutoConfiguration` 类。

🟢 **作用：**

- 让 `MvcConfig` 自动生效
- 仅在微服务中启用，网关不会加载

#### 八、OpenFeign 用户信息传递

##### 1、原因

在微服务架构中，每个服务都是独立运行的进程。
 虽然同属一个系统，但 **不同微服务之间的 ThreadLocal 数据并不会自动传递**。

------

**🧩 示例说明**

1. 用户登录后，**用户信息（userId）被放入 ThreadLocal（UserContext）中**。
2. 当请求经过拦截器（如 `UserInfoInterceptor`）时，ThreadLocal 能取到当前用户 ID。
3. **但**：
    当微服务之间通过 **OpenFeign** 调用时，这个 `ThreadLocal` 是**不会自动传递**的，因为每个微服务都是独立的 JVM。

👉 也就是说，**A 服务调用 B 服务时，B 服务的 ThreadLocal 是新的、空的。**

------

**✅ 因此必须：**

1. 在发起 Feign 请求时，**手动把 userId 放入请求头**；
2. 在下游服务收到请求后，**从请求头中再取出 userId 并放回 ThreadLocal**；
3. 请求结束后，**清理 ThreadLocal 防止内存泄漏**。

##### 2、解决方案设计

###### 1️⃣ 在 MVC 拦截器中清理 ThreadLocal

```
@Override
public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                            Object handler, Exception ex) {
    // 防止内存泄漏：请求结束后清理
    UserContext.removeUser();
}
```

🔹 **原因**：
 ThreadLocal 是每个线程独立的存储，如果不清理，线程被线程池复用时，会导致用户信息串号或泄漏。

------

###### 2️⃣ 在 OpenFeign 中添加请求拦截器

创建配置类（例如：`DefaultFeignConfig`）

```
@Bean
public RequestInterceptor userInfoRequestInterceptor() {
    return new RequestInterceptor() {
        @Override
        public void apply(RequestTemplate template) {
            // 获取当前用户ID
            Long userId = UserContext.getUser();
            if (userId == null) {
                // 未登录或非用户请求，直接跳过
                return;
            }
            // 将 userId 放入请求头，传递给下游服务
            template.header("userId", userId.toString());
        }
    };
}
```

🟢 **作用**：
 当 Feign 发送 HTTP 请求时，自动给每个请求加上 `userId` 请求头。

------

###### 3️⃣ 在微服务中启用 Feign 配置

在你的主启动类或配置类上添加：

```
@EnableFeignClients(
    basePackages = "com.hmall.api.client",  // 公共模块 Feign 接口所在包
    defaultConfiguration = DefaultFeignConfig.class  // 指向上面的拦截器配置
)
```

💡 **为什么必须写这个？**

- `basePackages`：告诉 Feign 去哪里找接口（比如 `com.hmall.api.client.ItemClient`）。
- `defaultConfiguration`：让每个 FeignClient 默认使用你的 `RequestInterceptor`，从而自动携带用户信息。

![](D:\GitHub\studyNotes\SpringCloud\图片\微服务登陆校验.png)

## Spring Cloud Alibaba Nacos 配置管理与共享配置

### 一、依赖配置

在 `pom.xml` 中引入以下依赖：

```
<!-- Nacos 配置管理 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<!-- 读取 bootstrap 配置文件 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

<img width="1839" height="896" alt="image" src="https://github.com/user-attachments/assets/13a8f5d8-6e6e-4194-9593-310b0f60b126" />

<img width="1792" height="887" alt="image" src="https://github.com/user-attachments/assets/981a97d7-7b90-4879-9e91-4b55c8b13d71" />


------

### 二、bootstrap.yaml 配置（配置管理入口）

> **bootstrap.yaml** 是 Spring Boot 启动时最早加载的配置文件，用于在应用启动前加载 Nacos 配置。

```
spring:
  application:
    name: cart-service # 服务名称
  profiles:
    active: dev
  cloud:
    nacos:
      server-addr: 192.168.195.131 # Nacos 服务地址
      config:
        file-extension: yaml # 文件后缀名
        shared-configs:      # 共享配置（可被多个服务共用）
          - dataId: jdbc.yaml  # 共享数据库配置
          - dataId: log.yaml   # 共享日志配置
```

------

### 三、本地配置（application.yaml）

```
server:
  port: 8082

hm:
  db:
    database: hm-cart

feign:
  okhttp:
    enabled: true # 开启 OKHttp 功能（提升 Feign 性能）
```

------

### 四、配置热更新

1. **创建配置类**

```
@Component
@Data
@ConfigurationProperties(prefix = "hm.cart")
public class CartProperties {
    private Integer maxItems;
}
```

- 使用 `@ConfigurationProperties` 自动绑定 Nacos 配置。
- 配合 `@RefreshScope`（如有使用）可实现 **配置热更新**（即修改后自动生效）。

------

### 五、Nacos 配置中心添加配置

<img width="1253" height="624" alt="image" src="https://github.com/user-attachments/assets/f569827f-af0f-4509-87ab-9daf72ff165b" />


在 Nacos 控制台中新建配置：

- **Data ID 格式：**

  ```
  [服务名]-[spring.profiles.active].[后缀名]
  ```

- 对于本例：

  ```
  cart-service-dev.yaml
  ```

- **内容示例：**

  ```
  hm:
    cart:
      max-items: 10
  ```

> 修改 `max-items` 值后，应用无需重启，配置会自动刷新（前提是配置类启用了热更新机制）。



## 微服务保护笔记（Sentinel）

### 一、雪崩问题简介

> 微服务架构中，**一个服务故障可能会引发级联失败**，导致整个系统不可用。
>  为防止这种“雪崩”效应，需要使用 **服务保护机制**（限流、隔离、降级、熔断等）。

------

### 二、Sentinel 简介

> **Sentinel** 是阿里巴巴开源的高可用防护框架，主要提供：

- **流量控制（限流）**
- **熔断降级**
- **系统自适应保护**
- **实时监控**

### 📦 下载与启动

#### 1️⃣ 下载

从 GitHub 下载：[Releases · alibaba/Sentinel](https://github.com/alibaba/Sentinel/releases)

#### 2️⃣ 启动

```
java -Dserver.port=8090 \
     -Dcsp.sentinel.dashboard.server=localhost:8090 \
     -Dproject.name=sentinel-dashboard \
     -jar sentinel-dashboard.jar
```

访问控制台：http://localhost:8090/
 **账号：** sentinel
 **密码：** sentinel

<img width="1026" height="777" alt="image" src="https://github.com/user-attachments/assets/9138a7d5-4102-4ca9-9a5a-6e0ae670b712" />

------

### 三、Spring Cloud 集成 Sentinel

#### 1️⃣ 引入依赖

```
<!-- Sentinel 服务保护依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

#### 2️⃣ 配置文件

```
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8090  # Sentinel 控制台地址
      http-method-specify: true     # 是否将 HTTP 方法作为资源名一部分
```

------

### 四、服务保护机制

#### （1）请求限流（QPS）

> 控制访问速率，防止瞬时高并发冲击。

操作步骤：

1. 启动控制台，访问应用后，簇点链路会显示资源；
2. 点击 **“流控”按钮**；
3. 设置规则：
   - 模式：**QPS**
   - 阈值：最大每秒请求数

🧠 **示例：**
 当设置 QPS=6 时，超过 6 次/秒 的请求会被拒绝。

<img width="1817" height="947" alt="image" src="https://github.com/user-attachments/assets/d533e0f1-2b7c-43eb-af6b-d041a8715e74" />

<img width="1576" height="788" alt="image" src="https://github.com/user-attachments/assets/80c1f1c0-ff0e-45f0-a3de-abef42269284" />

------

#### （2）线程隔离

> 为防止某个接口耗时过长，**阻塞其他接口**，可对其设置线程数限制。

配置方式：

1. 点击流控按钮；
2. 模式选择：**线程数（并发线程数）**；
3. 设置并发阈值。

```
ThreadUtil.sleep(500); // 模拟接口延迟
```
<img width="1810" height="869" alt="image" src="https://github.com/user-attachments/assets/a3c093d0-35ed-4a73-9f7a-ec233b6cca96" />


📘 **Tomcat资源耗尽示例：**

```
server:
  port: 8082
  tomcat:
    threads:
      max: 50           # 最大线程数
    accept-count: 50    # 最大排队等待数量
    max-connections: 100
```

⚙️ 说明：

- 当未设置线程隔离时，一个接口延迟会拖慢所有请求；
- 设置并发线程数后，仅该资源被限制，其他接口不受影响。

<img width="1827" height="737" alt="image" src="https://github.com/user-attachments/assets/95310f5e-2de0-41b0-a4fe-f30f2c7c3857" />

<img width="1534" height="766" alt="image" src="https://github.com/user-attachments/assets/16ebd10f-2a74-433f-aa84-2e659a210946" />

<img width="1803" height="900" alt="image" src="https://github.com/user-attachments/assets/8c2ab940-9089-44b3-99e8-260c0bc925e5" />

------

#### （3）Feign 降级与 Fallback

> 当调用下游微服务失败时，使用 **fallback 降级策略** 保证服务可用。

##### 1️⃣ 开启 Sentinel 对 Feign 的支持

```
feign:
  sentinel:
    enabled: true
```

##### 2️⃣ 编写 FallbackFactory

```
@Slf4j
public class ItemClientFallbackFactory implements FallbackFactory<ItemClient> {
    @Override
    public ItemClient create(Throwable cause) {
        return new ItemClient() {
            @Override
            public List<ItemDTO> queryByIds(Collection<Long> ids) {
                log.error("查询商品失败！原因：", cause);
                return CollUtils.emptyList();
            }

            @Override
            public void deductStock(List<OrderDetailDTO> items) {
                log.error("扣减商品库存失败！原因：", cause);
                throw new RuntimeException(cause);
            }
        };
    }
}
```

##### 3️⃣ 注册 Bean

```
@Bean
public ItemClientFallbackFactory itemClientFallbackFactory() {
    return new ItemClientFallbackFactory();
}
```

> ⚙️ 该 `@Bean` 写在 **DefaultFeignConfig** 中，因为该配置类已被 `@EnableFeignClients` 扫描：

```
@EnableFeignClients(
    basePackages = "com.hmall.api.client",  
    defaultConfiguration = DefaultFeignConfig.class
)
```

##### 4️⃣ FeignClient 中指定降级类

```
@FeignClient(value = "item-service", fallbackFactory = ItemClientFallbackFactory.class)
public interface ItemClient { ... }
```

💡 **这样即使下游服务失败，也不会抛出异常，而是使用 Fallback 返回默认数据或执行备用逻辑。**
<img width="1497" height="811" alt="image" src="https://github.com/user-attachments/assets/c101cf80-773e-4466-9d4e-cdb271e142da" />

------

#### （4）Controller 层的降级处理

如果不是 Feign 调用，而是在 **Controller 层的接口**，可以使用：

##### ✅ Sentinel 注解方式：

```
@GetMapping("/items")
@SentinelResource(value = "queryItems", fallback = "fallbackHandler")
public List<ItemDTO> queryItems() {
    // 模拟异常
    if (true) throw new RuntimeException("服务异常");
    return itemService.findAll();
}

// 降级方法
public List<ItemDTO> fallbackHandler(Throwable ex) {
    log.error("queryItems 服务降级：", ex);
    return CollUtils.emptyList();
}
```

> 📌 注意：
>
> - `@SentinelResource` 注解用于标记资源；
> - `fallback` 指定降级方法；
> - 降级方法签名需与原方法一致，并可多一个 `Throwable` 参数；
> - **⚠️ 该注解只能标注在具体方法上，不能标注在类上。**
>    （因为 Sentinel 以“方法调用”为粒度创建资源入口，类本身不执行逻辑，无法被 Sentinel 监控。）
> - **✅ 在 Controller 层使用 `@SentinelResource` 不需要实现 `FallbackFactory`。**
>    `FallbackFactory` 仅在 **Feign 调用远程服务** 时使用，用于在服务不可用时提供备用逻辑；
>    而 `@SentinelResource` 适用于 **本地方法（Controller 或 Service）** 的异常降级，两者机制不同。

#### （5）熔断（Circuit Breaker）

> 当某个接口**错误率过高**或**响应时间过长**时，自动“熔断”该接口一段时间，避免继续请求。

配置方式：

- 在 Sentinel 控制台选择“熔断降级”；
- 选择资源；
- 选择触发类型：
  1. **RT**（平均响应时间）
  2. **异常比例**（如>50%异常）
  3. **异常数**（如连续出错5次）
- 设置熔断时间（如5秒）。

🧠 熔断状态：

- **Closed**：正常状态；
- **Open**：熔断状态，拒绝请求；
- **Half-Open**：探测状态，允许部分请求测试恢复。
 <img width="1533" height="788" alt="image" src="https://github.com/user-attachments/assets/6dbc4970-3fd6-4642-affd-4267df474798" />
