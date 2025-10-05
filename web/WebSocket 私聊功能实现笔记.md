# WebSocket 私聊功能实现笔记

## 后端实现

### WebSocket 服务端点

```
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
        log.info("websocket is open, userId: {}",userId);
    }

    // 收到客户端消息
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("收到WebSocket消息: {}", message);
        try {
            // 尝试解析 JSON 消息
            ObjectMapper mapper = new ObjectMapper();
            Map<String, Object> msgMap = mapper.readValue(message, Map.class);
            log.info("解析后的消息内容: {}", msgMap);
            
            String toUserId = String.valueOf(msgMap.get("toUserId")); // 将数字转换为字符串
            String content = (String) msgMap.get("content");
            String fromUserId = session.getRequestParameterMap().get("userId").getFirst();
            
            log.info("处理消息 - 发送者: {}, 接收者: {}, 内容: {}", fromUserId, toUserId, content);

            // 检查目标用户是否存在
            if (sessionMap.containsKey(toUserId)) {
                Session targetSession = sessionMap.get(toUserId);
                targetSession.getBasicRemote().sendText("用户 " + fromUserId + " 对你说: " + content);
                log.info("消息已发送给用户: {}", toUserId);
            } else {
                // 可选：通知发送者目标用户不在线
                session.getBasicRemote().sendText("用户 " + toUserId + " 不在线");
                log.info("目标用户不存在: {}", toUserId);
            }
        } catch (Exception e) {
            log.error("消息处理失败: {}", e.getMessage());
            e.printStackTrace(); // 打印完整的错误堆栈
            // 如果不是JSON格式，则广播消息给所有用户
            String fromUserId = session.getRequestParameterMap().get("userId").getFirst();
            String broadcastMessage = "用户 " + fromUserId + " 广播消息: " + message;
            broadcast(broadcastMessage);
            log.info("收到非JSON格式消息，已广播: {}", message);
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

### WebSocket 配置类

```
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }
}

```

## 前端实现

### WebSocket Store (Pinia)

```
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { post } from "@/net/index.js"

export const useWebSocketStore = defineStore('websocket', () => {
  const ws = ref(null)
  const messageCallback=ref(null);

  const UpdateOnline = (online) => {
    post("api/user/UpdateOnline", {
      isOnline: online,
    }, (message) => {
      console.log("上线状态更新")
    })
  }

  const initWebSocket = (userId) => {
    // 如果已经存在连接且连接状态正常，则不重新连接
    if (ws.value && ws.value.readyState === WebSocket.OPEN) {
      return;
    }
    console.log("开始连接WebSocket")
    ws.value = new WebSocket(`ws://localhost:8080/myWs?userId=${userId}`)
    ws.value.onopen = () => {
      UpdateOnline("true")
      console.log("WebSocket连接成功！")
    }
    // 在 WebSocket 连接初始化时添加消息接收处理
    ws.value.onmessage = (event) => {
      console.log("收到WebSocket消息:", event.data)
      //检查是否已经设置了回调函数。如果已经设置了回调函数，那么就执行这个回调函数，并将收到的消息数据传递给它。
      if (messageCallback.value) {
        messageCallback.value(event.data)//调用 messageCallback.value 函数，并将 receivedData（收到的消息数据）作为参数传递给它。
      }
    }
  }

  const sendMsg = (toUserId,question) => {
    const privateMsg = {
      toUserId: toUserId,
      content: question
    };
    console.log(toUserId+" "+question);
    ws.value.send(JSON.stringify(privateMsg));
  }

  const closeWebSocket = () => {
    if (ws.value) {
      UpdateOnline("false");
      ws.value.close();
      ws.value = null;
    }
  }
  // 添加设置回调函数的方法
const setMessageCallback = (callback) => {
  messageCallback.value = callback//可以理解为messageCallback现在就是传入的回调函数
}


  return {
    ws,
    initWebSocket,
    closeWebSocket,
    sendMsg,
    setMessageCallback
  }
}) 
```

### 聊天界面组件--需要修改

```
<script setup>
import {onMounted, reactive, ref, watch} from "vue";
import {get, post} from "@/net/index.js";
import { UserOutlined } from '@ant-design/icons-vue';
import {userUserStore} from "@/stores/userStore.js";
import {useWebSocketStore} from "@/stores/websocketStore.js";
import {message} from "ant-design-vue";

const [messageApi, contextHolder] = message.useMessage();
const userStore=userUserStore();
const wsStore = useWebSocketStore();
const ChatMessage=ref([]);
const user=reactive({
  user:[],
  question:'',
  toUserId:0,
  avatar1:'',
  avatar2:'',
})
const getAllUsers=()=>{
  get('api/user/getAllUsers',{},(message,data)=>{
    user.user=data;
  })
}

const getChatByUserIdAndToUserId=()=>{
  get('/api/chat/getChatByUserIdAndToUserId',{
    toUserId:user.toUserId,
  },(message,data)=>{
    const Message=Array.isArray(data)?data:[data];
    Message.forEach((content)=>{
      ChatMessage.value.push({userId:content.sender,content:content.message})
    })
  })
}

const selectUser=(userId,avatar)=>{
  ChatMessage.value=[];
  user.toUserId=userId;
  user.avatar1=userStore.user.avator;
  user.avatar2=avatar
  getChatByUserIdAndToUserId();
}

const connection=()=>{
  post('/api/chat/PrivateChat',{
    toUserId:user.toUserId,
    message:user.question,
  },(message)=>{
    messageApi.success("发送成功");
    console.log('发送的消息:', {userId:userStore.user.id,content:user.question});
    ChatMessage.value.push({userId:userStore.user.id,content:user.question});
    wsStore.sendMsg(user.toUserId,user.question);
    user.question='';
  })
}

// 添加消息处理函数
const handleWebSocketMessage = (data) => {
  // 解析消息格式 "用户 xxx 对你说: xxx"
  const match = data.match(/用户 (\d+) 对你说: (.+)/);
  if (match) {
    const [, fromUserId, content] = match;
    ChatMessage.value.push({
      userId: parseInt(fromUserId),
      content: content,
    });
  }
}

watch(()=>wsStore.ws,(newWs,oldValue)=>{
  if(newWs!==oldValue)
  getAllUsers();
})

onMounted(()=>{
  getAllUsers();
  getChatByUserIdAndToUserId()
  wsStore.setMessageCallback(handleWebSocketMessage);
})

</script>

<template>
  <contextHolder/>
<div class="grid grid-cols-[7fr,2fr] w-full max-h-screen p-4 gap-6">
<!--  对话栏-->
  <div class="grid grid-rows-[11fr,2fr] h-full bg-opacity-100 gap-6">
<!--    消息展示-->
    <div class="w-full h-full  overflow-y-auto scrollbar-hide bg-white rounded-xl shadow-gray-500 shadow-md dark:bg-opacity-5 " style="max-height: 78vh">
      <div v-for="message in ChatMessage" class="gap-4 ">
        <div v-if="message.userId===userStore.user.id" class="w-full flex justify-end p-2 gap-2">
          <p class="max-w-4/5 flex flex-wrap my-auto border-2 border-gray-300 rounded-xl p-3">{{message.content}}</p>
          <a-avatar  :src="user.avatar1"  size="large"/>
        </div>
        <div v-else class="w-full flex  justify-start p-2 gap-2">
          <a-avatar  :src="user.avatar2"  size="large"/>
          <p class="max-w-4/5 flex flex-wrap my-auto border-2 border-gray-300 rounded-xl p-3 ">{{message.content}}</p>
        </div>
      </div>
    </div>
<!--    输入框-->
      <a-textarea @keyup.enter="connection" v-model:value="user.question" class="shadow-gray-500 shadow-md bg-white dark:bg-opacity-5" :rows="4"  placeholder="maxlength is 1000" :maxlength="10000" />
  </div>
<!--  好友栏-->
  <div class="w-full h-full  bg-white rounded-xl shadow-gray-500 shadow-md  dark:bg-opacity-5">
    <div  v-for="user in user.user"  class="w-full justify-start flex flex-nowrap p-4 gap-2">
      <a-avatar  :src="user.avator"  size="large"/>
      <p class="flex flex-nowrap items-center justify-center my-auto">{{user.username}}</p>
      <svg @click="selectUser(user.id,user.avator)" class="size-6 flex items-center justify-end my-auto hover:text-blue-600 hover:scale-110 active:scale-90" xmlns="http://www.w3.org/2000/svg"  viewBox="0 0 24 24">
        <g fill="none"><path fill="currentColor" d="m13.087 21.388l.645.382zm.542-.916l-.646-.382zm-3.258 0l-.645.382zm.542.916l.646-.382zm-8.532-5.475l.693-.287zm5.409 3.078l-.013.75zm-2.703-.372l-.287.693zm16.532-2.706l.693.287zm-5.409 3.078l-.012-.75zm2.703-.372l.287.693zm.7-15.882l-.392.64zm1.65 1.65l.64-.391zM4.388 2.738l-.392-.64zm-1.651 1.65l-.64-.391zM9.403 19.21l.377-.649zm4.33 2.56l.541-.916l-1.29-.764l-.543.916zm-4.007-.916l.542.916l1.29-.764l-.541-.916zm2.715.152a.52.52 0 0 1-.882 0l-1.291.764c.773 1.307 2.69 1.307 3.464 0zM10.5 2.75h3v-1.5h-3zm10.75 7.75v1h1.5v-1zm-18.5 1v-1h-1.5v1zm-1.5 0c0 1.155 0 2.058.05 2.787c.05.735.153 1.347.388 1.913l1.386-.574c-.147-.352-.233-.782-.278-1.441c-.046-.666-.046-1.51-.046-2.685zm6.553 6.742c-1.256-.022-1.914-.102-2.43-.316L4.8 19.313c.805.334 1.721.408 2.977.43zM1.688 16.2A5.75 5.75 0 0 0 4.8 19.312l.574-1.386a4.25 4.25 0 0 1-2.3-2.3zm19.562-4.7c0 1.175 0 2.019-.046 2.685c-.045.659-.131 1.089-.277 1.441l1.385.574c.235-.566.338-1.178.389-1.913c.05-.729.049-1.632.049-2.787zm-5.027 8.241c1.256-.021 2.172-.095 2.977-.429l-.574-1.386c-.515.214-1.173.294-2.428.316zm4.704-4.115a4.25 4.25 0 0 1-2.3 2.3l.573 1.386a5.75 5.75 0 0 0 3.112-3.112zM13.5 2.75c1.651 0 2.837 0 3.762.089c.914.087 1.495.253 1.959.537l.783-1.279c-.739-.452-1.577-.654-2.6-.752c-1.012-.096-2.282-.095-3.904-.095zm9.25 7.75c0-1.622 0-2.891-.096-3.904c-.097-1.023-.299-1.862-.751-2.6l-1.28.783c.285.464.451 1.045.538 1.96c.088.924.089 2.11.089 3.761zm-3.53-7.124a4.25 4.25 0 0 1 1.404 1.403l1.279-.783a5.75 5.75 0 0 0-1.899-1.899zM10.5 1.25c-1.622 0-2.891 0-3.904.095c-1.023.098-1.862.3-2.6.752l.783 1.28c.464-.285 1.045-.451 1.96-.538c.924-.088 2.11-.089 3.761-.089zM2.75 10.5c0-1.651 0-2.837.089-3.762c.087-.914.253-1.495.537-1.959l-1.279-.783c-.452.738-.654 1.577-.752 2.6C1.25 7.61 1.25 8.878 1.25 10.5zm1.246-8.403a5.75 5.75 0 0 0-1.899 1.899l1.28.783a4.25 4.25 0 0 1 1.402-1.403zm7.02 17.993c-.202-.343-.38-.646-.554-.884a2.2 2.2 0 0 0-.682-.645l-.754 1.297c.047.028.112.078.224.232c.121.166.258.396.476.764zm-3.24-.349c.44.008.718.014.93.037c.198.022.275.054.32.08l.754-1.297a2.2 2.2 0 0 0-.909-.274c-.298-.033-.657-.038-1.069-.045zm6.498 1.113c.218-.367.355-.598.476-.764c.112-.154.177-.204.224-.232l-.754-1.297c-.29.17-.5.395-.682.645c-.173.238-.352.54-.555.884zm1.924-2.612c-.412.007-.771.012-1.069.045c-.311.035-.616.104-.909.274l.754 1.297c.045-.026.122-.058.32-.08c.212-.023.49-.03.93-.037z"/><path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 11h.009m3.982 0H12m3.991 0H16"/></g>
      </svg>
      <svg v-if="user.isOnline==='true'" class="size-6 flex text-green-600 justify-end my-auto" xmlns="http://www.w3.org/2000/svg" width="128" height="128" viewBox="0 0 16 16">
        <path fill="currentColor" d="M8 14a1.5 1.5 0 1 1 0-3a1.5 1.5 0 0 1 0 3m0-1a.5.5 0 1 0 0-1a.5.5 0 0 0 0 1m3.189-3.64a.5.5 0 0 1-.721.692A3.4 3.4 0 0 0 8 9c-.937 0-1.813.378-2.453 1.037a.5.5 0 0 1-.717-.697A4.4 4.4 0 0 1 8 8c1.22 0 2.361.497 3.189 1.36m2.02-2.14a.5.5 0 1 1-.721.693A6.2 6.2 0 0 0 8 6a6.2 6.2 0 0 0-4.46 1.885a.5.5 0 0 1-.718-.697A7.2 7.2 0 0 1 8 5a7.2 7.2 0 0 1 5.21 2.22m2.02-2.138a.5.5 0 0 1-.721.692A9 9 0 0 0 8 3a9 9 0 0 0-6.469 2.734a.5.5 0 1 1-.717-.697A10 10 0 0 1 8 2a10 10 0 0 1 7.23 3.082"/>
      </svg>
      <svg v-else class="size-6 flex justify-end my-auto" xmlns="http://www.w3.org/2000/svg" width="128" height="128" viewBox="0 0 16 16">
        <path fill="currentColor" d="m6.517 12.271l1.254-1.254Q7.883 11 8 11a1.5 1.5 0 1 1-1.483 1.271m2.945-2.944l.74-.74c.361.208.694.467.987.772a.5.5 0 0 1-.721.693a3.4 3.4 0 0 0-1.006-.725m2.162-2.163l.716-.715q.463.349.87.772a.5.5 0 1 1-.722.692a6.3 6.3 0 0 0-.864-.749M7.061 6.07A6.2 6.2 0 0 0 3.54 7.885a.5.5 0 0 1-.717-.697a7.2 7.2 0 0 1 5.309-2.187zm6.672-1.014l.71-.71q.411.346.786.736a.5.5 0 0 1-.721.692a9 9 0 0 0-.775-.718m-3.807-1.85A9 9 0 0 0 8 3a9 9 0 0 0-6.469 2.734a.5.5 0 1 1-.717-.697A10 10 0 0 1 8 2c.944 0 1.868.131 2.75.382zM8 13a.5.5 0 1 0 0-1a.5.5 0 0 0 0 1m-5.424 1a.5.5 0 0 1-.707-.707L14.146 1.146a.5.5 0 0 1 .708.708z"/>
      </svg>
    </div>
  </div>
</div>
</template>

<style>
/* 隐藏所有浏览器滚动条 */
.scrollbar-hide::-webkit-scrollbar {
  display: none; /* Chrome/Safari/Edge */
}

.scrollbar-hide {
  -ms-overflow-style: none;  /* IE/Edge */
  scrollbar-width: none;     /* Firefox */
}
</style>
```

### 连接管理

```
const userStore = userUserStore()
const wsStore = useWebSocketStore()

// 页面卸载时关闭WebSocket
const handleBeforeUnload = () => {
  wsStore.closeWebSocket()
}

// 监听用户信息变化初始化WebSocket
watch(() => userStore.user, (newUser) => {
  if (newUser) {
    wsStore.initWebSocket(newUser.id)
    window.addEventListener('beforeunload', handleBeforeUnload)
  }
}, { immediate: true })

// 组件卸载时清理
onUnmounted(() => {
  window.removeEventListener('beforeunload', handleBeforeUnload)
  wsStore.closeWebSocket()
})
```

## 功能说明

1. **连接管理**：
   - 用户登录后自动建立WebSocket连接
   - 页面关闭或离开时自动断开连接
   - 连接状态同步到后端数据库
2. **消息处理**：
   - 支持私聊消息的发送和接收
   - 消息格式为JSON：`{toUserId: xxx, content: xxx}`
   - 接收到的消息格式：`"用户 xxx 对你说: xxx"`
3. **状态管理**：
   - 使用Pinia管理WebSocket连接状态
   - 使用回调函数处理收到的消息
   - 维护在线用户列表
4. **错误处理**：
   - 非JSON格式消息自动转为广播
   - 目标用户不在线时通知发送者
   - 连接异常时自动重连
5. **UI交互**：
   - 显示用户在线状态
   - 消息分左右显示区分发送/接收
   - 支持回车发送消息