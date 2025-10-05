# **WebSocket**使用

## **WebSocket 的特点**

**双向通信**：服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，实现真正的双向平等对话
**低延迟**：由于 WebSocket 是全双工的协议，服务器可以随时主动给客户端下发数据，相对于 HTTP 请求需要等待客户端发起请求，延迟明显更少
**保持连接状态**：与 HTTP 不同，WebSocket 需要先创建连接，这使得其成为一种有状态的协议，之后通信时可以省略部分状态信息
**支持二进制数据**：WebSocket 定义了二进制帧，可以更轻松地处理二进制内容
## 工作原理
![](https://www.fzpersonalweb.xyz/api/uploads/9ac504e1-9591-4504-ba6d-c0d34a8bc3cc_cef40755-813b-4362-bd9e-164a0162a5ed.png)

## 环境准备
### 安装WebSocket框架
**如果没有安装**
1.我们可以在设置里面下载一个插件`EditStarters`
![](https://www.fzpersonalweb.xyz/api/uploads/e05c3e5e-7af8-46a8-aac3-2ef254bf9d74_5468bcf96c56671630fcc48284bf9671.png)
2.安装好之后在`pom.xml`文件里面右键选择生成，再选择图片中对应的部分，就可以重新回到安装框架的地方继续选择需要安装的框架
![](https://www.fzpersonalweb.xyz/api/uploads/653d0e71-00e9-4664-ba40-36b99e1dea75_e1605cfbe6ec70e6c0667daaa33d3521.png)



## 后端实现
### WsServerEndpoint类
```language
//监听websocket地址（/myWs）
@ServerEndpoint("/myWs")
@Component
@Slf4j
public class WsServerEndpoint {

    static Map<String, Session> sessionMap = new ConcurrentHashMap<>();

    // 建立连接
    @OnOpen
    public void onOpen(Session session) {
        // 获取前端传递的 userId 参数
        String userId = session.getRequestParameterMap().get("userId").getFirst();
        sessionMap.put(userId, session); // 使用 userId 作为键
        log.info("websocket is open, userId: {}", userId);
    }

 // 收到客户端消息
    @OnMessage
    public void onMessage(String message, Session session) {
        try {
            // 尝试解析 JSON 消息
            //ObjectMapper是Jackson库的核心类，负责：
            // 将Java对象转换为JSON字符串（序列化）将JSON字符串转换为Java对象（反序列化）
            // 这里主要用于解析客户端发送的JSON格式消息
            ObjectMapper mapper = new ObjectMapper();
            try {
                //将JSON字符串解析为Map键值对结构
                Map<String, String> msgMap = mapper.readValue(message, Map.class);
                String toUserId = msgMap.get("toUserId");
                String content = msgMap.get("content");
//                session.getRequestParameterMap()：获取连接时URL的所有查询参数
//                例如连接URL：ws://localhost:8080/myWs?userId=1001
//                返回的是Map<String, List<String>>结构（因为同名的参数可能有多个值）
//                .get("userId")：获取名为"userId"的参数值列表
//                .getFirst()：获取该参数的第一个值（因为WebSocket连接时userId通常只有一个）
                String fromUserId = session.getRequestParameterMap().get("userId").getFirst();

                // 检查目标用户是否存在
                if (sessionMap.containsKey(toUserId)) {
                    Session targetSession = sessionMap.get(toUserId);
                    if (targetSession.isOpen()) {
                        // 发送给目标用户
                        targetSession.getBasicRemote().sendText("用户 " + fromUserId + " 对你说: " + content);
                    }
                } else {
                    // 可选：通知发送者目标用户不在线
                    session.getBasicRemote().sendText("用户 " + toUserId + " 不在线");
                }
            } catch (Exception e) {
                // 如果不是JSON格式，则广播消息给所有用户
                String fromUserId = session.getRequestParameterMap().get("userId").getFirst();
                String broadcastMessage = "用户 " + fromUserId + " 广播消息: " + message;
                broadcast(broadcastMessage);
                log.info("收到非JSON格式消息，已广播: {}", message);
            }
        } catch (Exception e) {
            log.error("消息处理失败: {}", e.getMessage());
        }
    }

    // 关闭连接
    @OnClose
    public void onClose(Session session) {
        // 获取前端传递的 userId 参数
        String userId = session.getRequestParameterMap().get("userId").getFirst();
        sessionMap.remove(userId); // 使用 userId 移除对应的 Session
        log.info("websocket is close, userId: {}", userId);
    }

    // 广播消息给所有客户端
    private void broadcast(String message) {
        sessionMap.forEach((id, session) -> {
            if (session.isOpen()) {
                try {
                    session.getBasicRemote().sendText(message);
                } catch (IOException e) {
                    log.error("广播消息失败: {}", e.getMessage());
                }
            }
        });
    }

    //定时任务，每隔两秒像客户端发送一个心跳
//    @Scheduled(fixedRate = 2000)
//    public void sendMessage() throws IOException {
//        for(String key:sessionMap.keySet()){
//            sessionMap.get(key).getBasicRemote().sendText("心跳");
//        }
//    }
}

```
### WebSocketConfig 类
```language
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }
}

```
## 前端代码--用于测试
```language
<template>
  <div class="chat-container">
    <h2>WebSocket 聊天测试（组合式API）</h2>

    <!-- 用户ID设置 -->
    <div class="user-id-input">
      <input
          v-model="userId"
          placeholder="输入你的用户ID（如1001）"
      />
      <button @click="connect">连接</button>
      <button @click="disconnect" :disabled="!isConnected">断开</button>
    </div>

    <!-- 消息展示区 -->
    <div class="message-box">
      <div v-for="(msg, index) in messages" :key="index" class="message">
        {{ msg }}
      </div>
    </div>

    <!-- 消息输入框 -->
    <div class="message-input">
      <input
          v-model="inputMessage"
          placeholder="输入消息"
          @keyup.enter="send"
          :disabled="!isConnected"
      />
      <button @click="send" :disabled="!isConnected">发送</button>
    </div>

    <!-- 目标用户ID（私聊用） -->
    <div class="target-user-input">
      <input
          v-model="targetUserId"
          placeholder="目标用户ID（私聊用，留空则广播）"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onBeforeUnmount } from 'vue';
import {userUserStore} from "@/stores/userStore.js";

// 响应式数据
const ws = ref(null);
const isConnected = ref(false);
const userId = userUserStore().user.id;
const targetUserId = ref('');
const inputMessage = ref('');
const messages = ref([]);

// 连接WebSocket
const connect = () => {
  if (!userId) {
    alert('请输入用户ID');
    return;
  }

  ws.value = new WebSocket(`ws://localhost:8080/myWs?userId=${userId}`);

  ws.value.onopen = () => {
    isConnected.value = true;
    messages.value.push('✅ 连接成功！');
  };

  ws.value.onmessage = (event) => {
    messages.value.push(`📩 ${event.data}`);
  };

  ws.value.onclose = () => {
    isConnected.value = false;
    messages.value.push('❌ 连接已断开');
  };

  ws.value.onerror = (error) => {
    messages.value.push(`❌ 连接错误: ${error}`);
  };
};

// 断开连接
const disconnect = () => {
  if (ws.value) {
    ws.value.close();
  }
};

// 发送消息
const send = () => {
  if (!isConnected.value || !inputMessage.value.trim()) return;

  if (targetUserId.value) {
    // 私聊消息（JSON格式）
    const privateMsg = {
      toUserId: targetUserId.value,
      content: inputMessage.value
    };
    ws.value.send(JSON.stringify(privateMsg));
    messages.value.push(`📤 你发给 ${targetUserId.value}: ${inputMessage.value}`);
  } else {
    // 广播消息
    ws.value.send(inputMessage.value);
    messages.value.push(`📤 你广播: ${inputMessage.value}`);
  }

  inputMessage.value = '';
};

// 组件卸载时自动断开连接
onBeforeUnmount(() => {
  if (ws.value) {
    ws.value.close();
  }
});
</script>

<style scoped>
/* 样式与选项式API版本相同 */
.chat-container {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.user-id-input, .message-input, .target-user-input {
  margin: 10px 0;
  display: flex;
  gap: 10px;
}

input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 12px;
  background: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.message-box {
  height: 300px;
  border: 1px solid #eee;
  padding: 10px;
  overflow-y: auto;
  margin-bottom: 10px;
}

.message {
  margin: 5px 0;
  padding: 5px;
  background: #f9f9f9;
  border-radius: 4px;
}
</style>
```
## 关键功能说明

1. **连接管理**：
   - 使用用户ID作为连接标识
   - 支持主动连接和断开
2. **消息类型**：
   - 私聊消息：JSON格式，包含目标用户ID和内容
   - 广播消息：普通文本格式，发送给所有在线用户
3. **状态反馈**：
   - 实时显示连接状态变化
   - 显示消息发送和接收情况
4. **错误处理**：
   - 捕获并显示连接错误
   - 处理目标用户不在线情况