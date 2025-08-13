---
title: 【小程序】uniapp引入野火对话
time: 2025-05-19 14:00:43
categories: 小程序
---

[官方文档](https://docs.wildfirechat.cn/web/integration.html)

## 连接

### 引入 sdk

- 下载[wx-chat](https://github.com/wildfirechat/wx-chat)项目包

- 把`wfc/proto/proto.min.js`替换为自己的文件，然后再把`wfc`文件目录引入到自己的项目下，随便放在哪个目录

- 将示例项目的`config.js`放在自己项目下

#### 遇到问题

引入后，wfc 中报错`btoa`有问题

查看到 btoa 是引用的`util/base64.min.js`，估计是打包的兼容性问题

但这个文件内容很少，因此重写一下`util/base64.min.js`的两个方法即可

```js
export const btoa = function (str) {
  const buffer = new ArrayBuffer(str.length);
  const dataView = new Uint8Array(buffer);
  for (let i = 0; i < str.length; i++) {
    dataView[i] = str.charCodeAt(i);
  }
  // 使用 uni-app 或微信小程序的 base64 编码方法
  return uni.arrayBufferToBase64(buffer); // uni-app 请用 uni.arrayBufferToBase64
};
export const atob = function (base64) {
  // 使用小程序 base64 转 ArrayBuffer
  const arrayBuffer = uni.base64ToArrayBuffer(base64); // uni-app 请用 uni.base64ToArrayBuffer
  const uint8Array = new Uint8Array(arrayBuffer);
  let str = "";
  for (let i = 0; i < uint8Array.length; i++) {
    str += String.fromCharCode(uint8Array[i]);
  }
  return str;
};
```

### 初始化

- 在`app.vue`的`onLaunch`中，进行 wfc 的初始化

```js
import wfc from "@/wfc/client/wfc";
// 初始化 wfc
wfc.init();
// 获取到当前用户的clientid，后续链接会用的
const clientId = wfc.getClientId("wx");
console.log("clientId", clientId);
Store.chatStore.setClientId(clientId);
return clientId;
```

- connect 链接

拿到当前用户的 userid 和 token，连接到 wfc

```js
wfc.connect(userId, token);
```

## 对话列表

其实 wx-chat 这个项目已经提供的很全面了，只是需要把小程序代码转为 uniapp 的代码，这步可以交由 ai 帮忙完成

因此这里只把所有代码贴出

```js
import { onLoad, onShow } from '@dcloudio/uni-app'
import { onMounted, onUnmounted, ref } from 'vue'
import EventType from '@/wfc/client/wfcEvent'
import ConnectionStatus from '@/wfc/client/connectionStatus'
import MessageConfig from '@/wfc/client/messageConfig'
import PersistFlag from '@/wfc/messages/persistFlag'
import wfc from '@/wfc/client/wfc'
import { timeFormat } from '@/wfc/util/time'
import Config from '@/config'
import sha1 from 'js-sha1'

interface ConversationUI {
  conversation: any
  ui: {
    title: string
    portrait: string
    lastMsgContent: string
    unread: number
    time: string
  }
}

const conversations = ref<ConversationUI[]>([])
const isLoading = ref(true)

// 事件处理函数
const onConnnectionStatusChange = (status: number) => {
  console.log('connectionStatus', status)
  if (status === ConnectionStatus.ConnectionStatusConnected) {
    showConversationList()
  }

  const logoutStatuses = [
    ConnectionStatus.ConnectionStatusLogout,
    ConnectionStatus.ConnectionStatusRejected,
    ConnectionStatus.ConnectionStatusKickedOff,
    ConnectionStatus.ConnectionStatusTokenIncorrect,
    ConnectionStatus.ConnectionStatusSecretKeyMismatch,
    ConnectionStatus.ConnectionStatusServerDown,
  ]

  if (logoutStatuses.includes(status)) {
    console.log('reLaunch login page, connectionStatus', status)
    uni.clearStorageSync()
  }
}

const onUserInfosUpdate = () => showConversationList()
const onGroupInfosUpdate = () => showConversationList()

const onReceiveMessage = (msg: any) => {
  console.log('receive msg', msg)
  if (
    MessageConfig.getMessageContentPersitFlag(msg.content.type) === PersistFlag.Persist_And_Count
  ) {
    showConversationList()
  }
}

const onSettingUpdate = () => showConversationList()
const onConversationInfoUpdate = () => showConversationList()

const calculateSign = (nonce: string, secretKey: string): { sign: string; timestamp: string } => {
  const timestamp = Date.now().toString()
  const data = `${nonce}|${secretKey}|${timestamp}`
  const sign = sha1(data)
  return { sign, timestamp }
}

const showConversationList = () => {
  const list = wfc.getConversationList([0, 1, 2], [0, 1])
  const clUi = list.map((item) => {
    item.ui = {
      title: item.title(),
      portrait: item.portrait() || Config.DEFAULT_USER_PORTRAIT,
      lastMsgContent: '',
      unread: item.unreadCount?.unread || 0,
      time: '',
    }

    if (item.lastMessage && item.lastMessage.messageContent) {
      let digest = item.lastMessage.messageContent.digest(item.lastMessage)
      digest = digest.replace(/\n/g, ' ')
      item.ui.lastMsgContent = digest
      item.ui.time = timeFormat(item.lastMessage.timestamp)
    }

    return item
  })

  console.log('cl', clUi)
  conversations.value = clUi
  isLoading.value = false
}

// 页面跳转
const chatTo = (e: any) => {
  const item = e.currentTarget.dataset.item
  wfc.clearConversationUnreadStatus(item.conversation)
  delete item.unread
  uni.navigateTo({
    url: `/pagesChat/pages/ChatPage/index?conversation=${JSON.stringify(item.conversation)}`,
  })
}

const onConversationLongTap = (e: any) => {
  const conversationInfo = e.currentTarget.dataset.item
  const menuItems = ['清空会话', '删除会话']
  if (conversationInfo.isTop) {
    menuItems.push('取消置顶')
  } else {
    menuItems.push('会话置顶')
  }

  uni.showActionSheet({
    itemList: menuItems,
    success(res) {
      const item = menuItems[res.tapIndex]
      switch (item) {
        case '清空会话':
          wfc.clearMessages(conversationInfo.conversation)
          showConversationList()
          break
        case '删除会话':
          wfc.removeConversation(conversationInfo.conversation, true)
          showConversationList()
          break
        case '会话置顶':
          wfc.setConversationTop(conversationInfo.conversation, true)
          break
        case '取消置顶':
          wfc.setConversationTop(conversationInfo.conversation, false)
          break
      }
    },
    fail(res) {
      console.log(res.errMsg)
    },
  })
}

// 生命周期
onMounted(() => {
  wfc.eventEmitter.on(EventType.ConnectionStatusChanged, onConnnectionStatusChange)
  wfc.eventEmitter.on(EventType.UserInfosUpdate, onUserInfosUpdate)
  wfc.eventEmitter.on(EventType.GroupInfosUpdate, onGroupInfosUpdate)
  wfc.eventEmitter.on(EventType.ReceiveMessage, onReceiveMessage)
  wfc.eventEmitter.on(EventType.SettingUpdate, onSettingUpdate)
  wfc.eventEmitter.on(EventType.ConversationInfoUpdate, onConversationInfoUpdate)
})

onUnmounted(() => {
  wfc.eventEmitter.removeListener(EventType.ConnectionStatusChanged, onConnnectionStatusChange)
  wfc.eventEmitter.removeListener(EventType.UserInfosUpdate, onUserInfosUpdate)
  wfc.eventEmitter.removeListener(EventType.GroupInfosUpdate, onGroupInfosUpdate)
  wfc.eventEmitter.removeListener(EventType.ReceiveMessage, onReceiveMessage)
  wfc.eventEmitter.removeListener(EventType.SettingUpdate, onSettingUpdate)
})

onShow(() => {
  const userId = wx.getStorageSync('userId')
  const token = wx.getStorageSync('token')
  if (userId && token) {
    wfc.connect(userId, token)
    wfc.onForeground()
    if (wfc.getConnectionStatus() === ConnectionStatus.ConnectionStatusConnected) {
      showConversationList()
    }
  }
})

// 加载头像错误处理
const loadPortraitError = (e: Event) => {
  console.log('load conversation target portrait error', e)
}

```

## 对话详情

对话详情会比对话页面复杂些，因为有各种消息类型需要处理，而且在例子中涉及文件较多，但都可以用 ai 处理转为 uniapp 语法

需要把`pages/chat`下的`ui.js`和`msg-type`目录拷贝到对应的页面下，当然，也可以放到其他文件，只需要改一下引入即可

示例项目中，会涉及到 chatinput、chatitem、chatword、chatloading、、chattime 等多个文件，需要将对应的文件一个一个转为符合自己需求的文件即可

```js
import { ref, onMounted, onUnmounted } from 'vue'
import UI from './ui'
import { onLoad, onShow } from '@dcloudio/uni-app'
import VoiceManager from './msg-type/voice-manager'
import wfc from '@/wfc/client/wfc'
import ConversationInfo from '@/wfc/model/conversationInfo'
import TextMessageContent from '@/wfc/messages/textMessageContent'
import WelcomeMgsMessageContent from '@/wfc/messages/welcomeMgsMessageContent'
import CopyTextMessageContent from '@/wfc/messages/copyTextMessageContent'
import EventType from '@/wfc/client/wfcEvent'
import Conversation from '@/wfc/model/conversation'
import ImageMessageContent from '@/wfc/messages/imageMessageContent'
import SoundMessageContent from '@/wfc/messages/soundMessageContent'
import MessageStatus from '@/wfc/messages/messageStatus'
import NotificationMessageContent from '@/wfc/messages/notification/notificationMessageContent'
import { eq, gt, numberValue } from '@/wfc/util/longUtil'
import { timeFormat } from '@/wfc/util/time'
import avenginekitproxy from '@/wfc/av/engine/avenginekitproxy'
import ConversationType from '@/wfc/model/conversationType'
import CallStartMessageContent from '@/wfc/av/messages/callStartMessageContent'
import ConferenceInviteMessageContent from '@/wfc/av/messages/conferenceInviteMessageContent'
import StreamingTextGeneratingMessageContent from '@/wfc/messages/streamingTextGeneratingMessageContent'
import StreamingTextGeneratedMessageContent from '@/wfc/messages/streamingTextGeneratedMessageContent'
import MessageContentType from '@/wfc/messages/messageContentType'
import ChatInput from './components/ChatInput.vue'
import ChatItem from './components/ChatItem.vue'

// 定义响应式数据
const conversation = ref<Conversation>(new Conversation())
const lastTimestamp = ref<number>(0)
const textMessage = ref('')
const chatItems = ref<any[]>([])
const latestPlayVoicePath = ref('')
const chatStatue = ref('open')
const isLoadAll = ref(false)
const extraArr = ref([
  {
    picName: 'choose_picture',
    description: '照片',
  },
  {
    picName: 'take_photos',
    description: '拍摄',
  },
  {
    picName: 'take_photos',
    description: '音频通话',
  },
  {
    picName: 'take_photos',
    description: '视频通话',
  },
])
const scrollTopVal = ref(0)
const pageHeight = ref(0)

let UIInstance: UI
let voiceManager: VoiceManager

function _isDisplayMessage(message) {
  // return [PersistFlag.Persist, PersistFlag.Persist_And_Count].indexOf(MessageConfig.getMessageContentPersitFlag(message.messageContent.type)) > -1;
  return (
    message.messageId !== 0 ||
    message.messageContent.type === MessageContentType.Streaming_Text_Generating
  )
}
function onReceiveMessage(msg: any) {
  if (!msg.conversation.equal(conversation.value) || !_isDisplayMessage(msg)) {
    return
  }

  wfc.clearConversationUnreadStatus(conversation.value)

  const msgIndex = chatItems.value.findIndex((m) => {
    return (
      m.messageId === msg.messageId ||
      (gt(m.messageUid, 0) && eq(m.messageUid, msg.messageUid)) ||
      (m.messageContent.type === MessageContentType.Streaming_Text_Generating &&
        (msg.messageContent.type === MessageContentType.Streaming_Text_Generating ||
          msg.messageContent.type === MessageContentType.Streaming_Text_Generated) &&
        m.messageContent.streamId === msg.messageContent.streamId)
    )
  })

  const newUIMsg = messagesToUiMessages([msg])[0]
  if (msgIndex >= 0) {
    chatItems.value[msgIndex] = newUIMsg
  } else {
    chatItems.value.push(newUIMsg)
  }

  scrollTopVal.value = chatItems.value.length * 999
}

function onSendMessage(msg: any) {
  if (!msg.conversation.equal(conversation.value)) return
  showMessageList()
}

function onMessageStatusUpdate(msg: any) {
  if (!msg.conversation.equal(conversation.value)) return
  showMessageList()
}

function onRecallMessage(operator: string, messageUid: number) {
  const msg = wfc.getMessageByUid(messageUid)
  if (!msg.conversation.equal(conversation.value)) return
  showMessageList()
}
onShow(() => {
  wfc.onForeground()
  showMessageList()
})
onLoad((options) => {
  console.log('------------chat onLoad----------', options)

  const conversationJson = JSON.parse(options.conversation)
  pageHeight.value = uni.getSystemInfoSync().windowHeight
  conversation.value = Object.assign(new Conversation(), conversationJson)
  const conversationInfo = new ConversationInfo()
  conversationInfo.conversation = conversation.value
  console.log(
    '------------chat onLoad----------',
    options,
    conversation.value,
    conversationInfo.conversation,
  )

  uni.setNavigationBarTitle({
    title: conversationInfo.title() || '',
  })

  UIInstance = new UI(getCurrentInstance())
  voiceManager = new VoiceManager(getCurrentInstance())

  // 注册事件监听
  wfc.eventEmitter.on(EventType.ReceiveMessage, onReceiveMessage)
  wfc.eventEmitter.on(EventType.SendMessage, onSendMessage)
  wfc.eventEmitter.on(EventType.MessageStatusUpdate, onMessageStatusUpdate)
  wfc.eventEmitter.on(EventType.RecallMessage, onRecallMessage)
})

onUnmounted(() => {
  voiceManager.stopAllVoicePlay(true)

  wfc.eventEmitter.off(EventType.ReceiveMessage, onReceiveMessage)
  wfc.eventEmitter.off(EventType.SendMessage, onSendMessage)
  wfc.eventEmitter.off(EventType.MessageStatusUpdate, onMessageStatusUpdate)
  wfc.eventEmitter.off(EventType.RecallMessage, onRecallMessage)
})

function onSendMessageEvent(e) {
  const content = e.detail.value
  const textMsgContent = new TextMessageContent(content)
  sendMessage(textMsgContent)
}
function onSendCopyMessageEvent(msg) {
  const textMsgContent = new CopyTextMessageContent(msg)
  sendMessage(textMsgContent)
}
function sendMessage(messageContent: any) {
  wfc.sendConversationMessage(
    conversation.value,
    messageContent,
    null,
    () => {
      showMessageList()
    },
    (progress, total) => {
      console.log('upload progress', progress, total)
      showMessageList()
    },
    () => {
      showMessageList()
    },
    () => {
      showMessageList()
    },
  )
}
function loadOldMessages() {
  console.log('loadOldMessages')
  const messages = chatItems.value
  const beforeUid = messages.length > 0 ? messages[0].messageUid : 0
  const beforeId = messages.length > 0 ? messages[0].messageId : 0
  wfc.getMessagesV2(
    conversation.value,
    beforeId,
    true,
    20,
    '',
    (msgs) => {
      if (msgs.length > 0) {
        let uiMsgs = messagesToUiMessages(msgs)
        uiMsgs = uiMsgs.concat(messages)
        chatItems.value = uiMsgs
        console.log('chatItems.value', uiMsgs)
      } else {
        wfc.loadRemoteConversationMessages(
          conversation.value,
          [],
          beforeUid,
          50,
          (msgs) => {
            if (msgs.length > 0) {
              let uiMsgs = messagesToUiMessages(msgs)
              if (uiMsgs.length === 0) {
                isLoadAll.value = true
              }
              uiMsgs = uiMsgs.concat(messages)
              chatItems.value = uiMsgs
              console.log('chatItems.value', uiMsgs)
            } else {
              isLoadAll.value = true
            }
          },
          (errorCode) => {
            console.log('load remote message error', errorCode)
          },
        )
      }
    },
    (err) => {
      console.log('load remote message error', err)
    },
  )
}
function showMessageList() {
  wfc.getMessagesV2(
    conversation.value,
    0,
    true,
    20,
    '',
    (messages) => {
      console.log('getMessagesV2', messages)
      if (messages.length < 20) {
        loadOldMessages()
        if (messages.length === 0) {
          return
        }
      }

      const uiMsgs = messagesToUiMessages(messages)
      chatItems.value = uiMsgs
      scrollTopVal.value = uiMsgs.length * 999
    },
    (err) => {
      console.error('getMessagesV2 error', conversation.value, err)
    },
  )
}

function messagesToUiMessages(messages: any[]) {
  return messages.map((m) => {
    const item = {
      showTime: numberValue(m.timestamp) - lastTimestamp.value > 2 * 60 * 1000,
      time: timeFormat(m.timestamp),
      headUrl: wfc.getUserInfo(m.from).portrait,
      isMy: m.direction === 0,
      isPlaying: false,
      sendStatus: 'sending' | 'success' | 'failed',
      content: '',
      type: '',
    }

    switch (m.status) {
      case MessageStatus.Sending:
        item.sendStatus = 'sending'
        break
      case MessageStatus.Sent:
        item.sendStatus = 'success'
        break
      case MessageStatus.SendFailure:
        item.sendStatus = 'failed'
        break
    }

    if (m.messageContent instanceof TextMessageContent) {
      item.content = m.messageContent.content
      item.type = 'text'
    } else if (m.messageContent instanceof WelcomeMgsMessageContent) {
      item.content = m.messageContent.content
      item.type = 'welcome-msg'
    } else if (m.messageContent instanceof CopyTextMessageContent) {
      item.content = m.messageContent.content
      item.type = 'copy-text'
    } else if (m.messageContent instanceof ImageMessageContent) {
      item.content = m.messageContent.localPath || m.messageContent.remotePath
      item.type = 'image'
    } else if (m.messageContent instanceof SoundMessageContent) {
      item.content = m.messageContent.localPath || m.messageContent.remotePath
      item.type = 'voice'
    }

    lastTimestamp.value = numberValue(m.timestamp)
    m.ui = item
    return m
  })
}
```

## 自定义消息类型

[自定义消息类型](https://docs.wildfirechat.cn/base_knowledge/custom_message_content.html?h=%E6%B6%88%E6%81%AF%E7%B1%BB%E5%9E%8B)，官方文档里面写了对应的文档

这里给出一个基于 text 消息类型衍生的增加复制按钮的消息类型 CopyTextMessage

因为这个消息类型下，只需要增加一个复制按钮即可，其他的和 text 消息类型一样，所以这个例子很简单

### 消息类型定义

在`wfc/message`目录下，新建一个文件`CopyTextMessageContent.ts`作为消息类型定义的文件

```js
/*
 * Copyright (c) 2020 WildFireChat. All rights reserved.
 */

import MessageContent from "./messageContent";
import MessageContentType from "./messageContentType";
import wfc from "../client/wfc";
import QuoteInfo from "../model/quoteInfo";

export default class copyTextMessageContent extends MessageContent {
  content;
  quoteInfo;

  constructor(content, mentionedType = 0, mentionedTargets = []) {
    super(MessageContentType.Copy_Text, mentionedType, mentionedTargets);
    this.content = content;
  }

  digest() {
    return this.content;
  }

  encode() {
    const payload = super.encode();
    payload.searchableContent = this.content;
    if (this.quoteInfo) {
      const obj = {
        quote: this.quoteInfo.encode(),
      };
      // JSON.parse 和 JSON.stringify 不能处理java long
      const orgStr = JSON.stringify(obj);
      const str = orgStr.replace(/"u":"([0-9]+)"/, '"u":$1');

      payload.binaryContent = wfc.utf8_to_b64(str);
    }
    return payload;
  }

  decode(payload) {
    super.decode(payload);
    this.content = payload.searchableContent;
    if (payload.binaryContent && payload.binaryContent.length > 0) {
      // JSON.parse 和 JSON.stringify 不能处理java long
      let quoteInfoStr = wfc.b64_to_utf8(payload.binaryContent);
      // FIXME node 环境，decodeURIComponent 方法，有时候会在最后添加上@字符，目前尚未找到原因，先规避
      quoteInfoStr = quoteInfoStr.substring(
        0,
        quoteInfoStr.lastIndexOf("}") + 1
      );
      quoteInfoStr = quoteInfoStr.replace(/"u":([0-9]+),/, '"u":"$1",');
      const obj = JSON.parse(quoteInfoStr).quote;

      this.quoteInfo = new QuoteInfo();
      this.quoteInfo.decode(obj);
    }
  }

  setQuoteInfo(quoteInfo) {
    this.quoteInfo = quoteInfo;
  }
}
```

### 声明消息类型

- `wfc/message/messageContentType`

  增加消息类型对应的枚举

  `static Copy_Text = 17`

- `wfc/client/messageConfig`

  增加消息类型列表，在`MessageContents`变量中添加

```diff
+ import CopyTextMessageContent from '../messages/copyTextMessageContent'
static MessageContents = [
...
+ {
+ name: 'copy-text',
+ flag: PersistFlag.Persist_And_Count,
+ type: MessageContentType.CopyTextMessageContent,
+ contentClazz: WelcomeMgsMessageContent,
+ },
...
]
```

- `wfc/model/favItem`

  `fromMessage`修改内容返回，这里主要用于消息列表的最新消息显示

```diff
+ import CopyTextMessageContent from '../messages/copyTextMessageContent'
switch (message.messageContent.type) {
case MessageContentType.Text:
    const textMessageContent = message.messageContent
    favItem.title = textMessageContent.content
    break
+ case MessageContentType.Copy_Text:
+    const CopyTextMessageContent = message.messageContent
    favItem.title = CopyTextMessageContent.content
    break
    ...
}
```

### 使用

- 格式化消息列表

  在聊天内容列表，增加对自定义消息类型的处理

```diff
function messagesToUiMessages(messages: any[]) {
    ...
    if (m.messageContent instanceof TextMessageContent) {
    item.content = m.messageContent.content;
    item.type = "text";
+         } else if (m.messageContent instanceof CopyTextMessageContent) {
+          item.content = m.messageContent.content;
+          item.type = "copy-text";
+        }
    ...
}
```

- 发送自定义消息类型

当需要发送对应类型的消息文本时，需要引入对应的消息类型 class 声明，并创建一个实例传给 sendMessage

```js
import CopyTextMessageContent from "@/wfc/messages/copyTextMessageContent";
function onSendCopyMessageEvent(msg) {
  const textMsgContent = new CopyTextMessageContent(msg);
  sendMessage(textMsgContent);
}
```

- 自定义消息类型显示

野火通过type来区分不同的消息类型，因此在`chatWord`文件中，需要加上自定义消息类型的ui代码

``` html
<!-- 复制文本消息 -->
<block v-if="type === 'copy-text'">
  <view
    class="message-content"
    :class="{ isMyWordStyle: isMy, isOtherWordStyle: !isMy }"
    :style="{
          wordWrap: 'break-word',
          borderRadius: isMy ? '16rpx 0px 16rpx 16rpx' : '0px 16rpx 16rpx 16rpx',
        }"
    @click="handleTextClick"
    :data-index="index"
  >
    <text class="inline" v-for="text in content" :key="text"> {{ text }} </text>
    <image
      :src="copyIcon"
      class="inline-block ml-16rpx w-40rpx h-40rpx align-top"
      @click="onCopy"
    />
  </view>
</block>
```
