<template>
  <view class="chat-container">
	  
    <!-- 聊天记录区域 -->
    <scroll-view class="messages" scroll-y="true" ref="scrollView">
      <view v-for="(message, index) in chatMessages" :key="index" class="message">
        {{ message }}
      </view>
    </scroll-view>

    <!-- 固定在底部的输入消息和发送按钮区域 -->
    <view class="input-container">
      <textarea type="text" v-model="newMessage" placeholder="输入消息, 默认为人体损伤鉴定系统. 上传文件定制模型." auto-height="true" maxlength="-1"/>
      <button @click="sendMessage">发送</button>
	  <button @click="selectFile">上传文件</button> <!-- 添加上传文件按钮 -->
    </view>
  </view>
</template>

<script>
// import io from '@hyoga/uni-socket.io';
// import io from '@/js_sdk/hyoga-uni-socket_io/uni-socket.io.js';
export default {
  data() {
    return {
      newMessage: '', // 新消息的内容
      chatMessages: [], // 存储聊天记录
	  chatSocket: null,
    };
  },
  mounted() {
	  let outer = this;
	  this.chatSocket = new WebSocket("ws://127.0.0.1:8000/ws/chat/lobby/");
	  this.chatSocket.onmessage = function(e) {
		  const data = JSON.parse(e.data);
		  console.log(data.message);
		  outer.chatMessages.push(data.message);
	  };
	  this.chatSocket.onclose = function(e) {
		  console.error('Chat socket closed unexpectedly');
	  };
  },
  methods: {
    sendMessage() {
      if (this.newMessage.trim() !== '') {
		console.log(this.chatSocket);
        this.chatMessages.push("YOU: " + this.newMessage);
		console.log(this.chatMessages);
		let outer = this;
		this.chatSocket.send(JSON.stringify({
			'message': outer.newMessage,
			'type': 'string'
		}));
		this.newMessage = '';
      }
    },
	// 选择文件的方法
	selectFile() {
	  let input = document.createElement('input');
	  input.type = 'file';
	  input.onchange = e => {
		let file = e.target.files[0];
		this.sendFile(file);
	  };
	  input.click();
	},

	// 发送文件的方法
	sendFile(file) {
	  let reader = new FileReader();
	  reader.onload = e => {
	    let fileData = e.target.result;
	    // 创建包含文件数据的对象
	    let messageObject = {
	      'message': fileData,
		  'type': 'file'
	    };
	    // 将对象转换为 JSON 字符串并发送
	    this.chatSocket.send(JSON.stringify(messageObject));
	  };
	  reader.readAsDataURL(file); // 将文件读取为 Base64
	}
  }
};
</script>

<style>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages {
  flex-grow: 1;
  overflow-y: auto;
  padding: 10px;
  background-color: #f0f0f0; /* 根据需要调整背景色 */
}

.message {
  margin-bottom: 10px;
  padding: 10px;
  background-color: #fff;
  border-radius: 10px;
}

.input-container {
  display: flex;
  padding: 10px;
  background-color: #ffffff; /* 输入区背景色 */
}

.input-container textarea {
  flex-grow: 1;
  margin-right: 10px;
  padding: 0px;
  border: 1px solid #ccc;
  border-radius: 4px;
  height: 100%;
  padding: 15px !important;
}

.input-container button {
  padding: 5px 15px;
  background-color: #1e90ff;
  color: #ffffff;
  border: none;
  border-radius: 4px;
  margin: 1px;
}
</style>