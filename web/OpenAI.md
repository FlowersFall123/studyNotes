# ***OpenAI***

## **一、准备**

### 步骤一、首先创建一个**后端模块**

我们需要的安装以下的配置

![image-20250211200439730](C:\Users\鲍克靖\AppData\Roaming\Typora\typora-user-images\image-20250211200439730.png)

### 步骤二、之后我们需要在`pom.xml`文件中修改一下[OpenAI的版本](https://spring-doc.cadn.net.cn/projects/spring-ai)，稳定版直接跳过

```xml
<properties>
    <java.version>21</java.version>
    <spring-ai.version>1.0.0-SNAPSHOT</spring-ai.version> 
</properties>
```

同时我们需要添加对应的**仓库(这个是快照版的)**

```xml
<repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled> <!-- 启用快照版本 -->
            </snapshots>
        </repository>
    </repositories>
```

由于maven仓库使用的是阿里云的镜像，镜像中没有对应的内容，需要添加一个新镜像

```xml
<mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
</mirror>
```

### 步骤三、之后就是在`application.yml`中输入以下代码

```yml
spring:
  application:
    name: 模块名  # 冒号后添加空格

  ai:
    openai:
      api-key:  # 填写实际的API密钥
      base-url:   #路径
      chat:
        options:
          model:   #gpt-4-32k 版本  32k参数
          temperature: #0.4F 越高准确率越低，但是创新性增加
```

## 二、AI聊天代码

### 1.字符串格式

```java
@RestController
public class ChatController {

    /**
     * spring ai 自动装配的 
     */
    private final ChatClient chatClient;

    public AIChatController(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }

    @GetMapping("/chat")
    public String chat(@RequestParam("message") String message) {
        return chatClient.prompt().user(message).call().content();
    }
}
```

**补充：问号后面就是问题**

![image-20250212090145807](C:\Users\鲍克靖\AppData\Roaming\Typora\typora-user-images\image-20250212090145807.png)

### **2.输出`json`格式**

为`JWTFilter`添加上`asyncSupported = true`

![image-20250508155209355](C:\Users\鲍克靖\AppData\Roaming\Typora\typora-user-images\image-20250508155209355.png)

同时还需要在跨域对应的代码里面加上

```
@Override
public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    configurer.setDefaultTimeout(60000); // 设置异步请求超时时间为60秒
}
```

![image-20250508155342581](C:\Users\鲍克靖\AppData\Roaming\Typora\typora-user-images\image-20250508155342581.png)

```java
@RequestMapping(value = "/api/chat2")
public Object chat2(@RequestParam(value = "message") String message) {
    ChatResponse chatResponse =openAiChatModel.call(new Prompt(message));
    return chatResponse.getResult().getOutput().getText();
}
```

### 3.流式

```java
//流式
     @GetMapping(value = "/chat3", produces = MediaType.TEXT_EVENT_STREAM_VALUE )
    public Flux<String> chatStream(@RequestParam("message") String message) {
        return chatClient.prompt()
                .user(message)
                .stream()
                .content()
                .delayElements(Duration.ofMillis(100));//延迟
    }
```

## 3.图像生成

```java
@RequestMapping(value = "/api/image")
    private Object image(@RequestParam(value = "image") String image){
        //chat是在文件里面配置的，二者同时配置优先使用代码里面的
        ImageResponse imageResponse=openAiImageModel.call(new ImagePrompt(image, OpenAiImageOptions.builder().
                withQuality("hd")//高清图片
                .withN(1)//1张图片(这个模型只能画一个)
                .withHeight(1024)//
                .withWidth(1024).build()));
        System.out.println(imageResponse);
        String url=imageResponse.getResult().getOutput().getUrl();

        return imageResponse.getResult().getOutput();
    }
```

## 4.多态

**`message+image`**

```java
@Resource
private ChatModel chatModel;


@RequestMapping(value="/api/multi")
public Object multi(String message,String imageUrl) throws MalformedURLException {
    // 将 String 转为 URL
    URL imageUrlObj = new URL(imageUrl);
    UserMessage userMessage = new UserMessage(message, new Media(MimeTypeUtils.IMAGE_PNG, imageUrlObj));

    ChatResponse response = chatModel.call(new Prompt(userMessage, OpenAiChatOptions.builder()
            .model(OpenAiApi.ChatModel.GPT_4_O.getValue())
            .build()));
    return response.getResult().getOutput().getText();
}

//http://localhost:8080/api/multi?
//message=%E6%8F%8F%E8%BF%B0%E4%B8%80%E4%B8%8B%E5%9B%BE%E7%89%87&
//imageUrl=https://c-ssl.duitang.com/uploads/blog/202303/02/20230302194343_6716b.jpeg
```

![image-20250215095937813](C:\Users\鲍克靖\AppData\Roaming\Typora\typora-user-images\image-20250215095937813.png)