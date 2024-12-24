<script setup>
import { ref } from 'vue';

// 存储选择的文件
const selectedFile = ref(null);
// 本地Peer ID，用于标识本地端，初始为空
const localPeerId = ref('');
// 远程接收方的Peer ID，用于建立连接，初始为空
const remotePeerId = ref('');
// 用于本地创建WebRTC连接的RTCPeerConnection实例
const localPeerConnection = ref(null);
// 用于远程接收方创建WebRTC连接的RTCPeerConnection实例
const remotePeerConnection = ref(null);
// 用于文件传输的数据通道实例
const dataChannel = ref(null);
// 文件读取流对象
const fileReader = ref(null);

// 生成随机的Peer ID（简单示例，可替换为更可靠方式）
const generatePeerId = () => {
  const randomId = Math.random().toString(36).substring(2, 10);
  localPeerId.value = randomId;
};

// 处理文件选择事件
const handleFileSelect = (e) => {
  selectedFile.value = e.target.files[0];
};

// 创建本地WebRTC对等连接及相关设置
const createLocalPeerConnection = () => {
  localPeerConnection.value = new RTCPeerConnection();
  // 创建数据通道用于传输文件数据
  dataChannel.value = localPeerConnection.value.createDataChannel('file-transfer');
  dataChannel.value.onopen = () => {
    console.log('数据通道已打开，可以开始传输文件');
  };
  dataChannel.value.onmessage = (event) => {
    // 这里可处理接收方发来的数据（若有双向交互需求），暂简单打印示例
    console.log('从数据通道接收到消息：', event.data);
  };
  dataChannel.value.onclose = () => {
    console.log('数据通道已关闭');
  };

  // 处理ICE候选事件（用于网络穿透等），这里简单打印示例，实际要发送给接收方
  localPeerConnection.value.onicecandidate = (event) => {
    if (event.candidate) {
      console.log('本地ICE候选：', event.candidate);
      // 实际应用中要通过信令服务器将此候选发送给接收方
    }
  };
};

// 创建远程WebRTC对等连接及相关设置（接收方相关）
const createRemotePeerConnection = () => {
  remotePeerConnection.value = new RTCPeerConnection();
  remotePeerConnection.value.onicecandidate = (event) => {
    if (event.candidate) {
      console.log('远程ICE候选：', event.candidate);
      // 实际应用中要通过信令服务器将此候选发送给本地端
    }
  };
  remotePeerConnection.value.ondatachannel = (event) => {
    dataChannel.value = event.channel;
    dataChannel.value.onopen = () => {
      console.log('数据通道已打开，可以接收文件');
    };
    dataChannel.value.onmessage = (event) => {
      console.log('接收到文件数据：', event.data);
      // 这里接收到数据后可进一步处理保存等操作
    };
    dataChannel.value.onclose = () => {
      console.log('数据通道已关闭');
    };
  };
};

// 开始传输文件，创建连接、生成Offer等操作
const startTransfer = () => {
  generatePeerId();
  createLocalPeerConnection();
  localPeerConnection.value.createOffer()
      .then((offer) => localPeerConnection.value.setLocalDescription(offer))
      .then(() => {
        console.log('本地Offer已生成并设置，需发送给接收方（通过信令服务器）：', localPeerConnection.value.localDescription);
        // 实际应用中要通过信令服务器将Offer发送给接收方
      });
};

// 连接接收方，根据接收的Offer创建Answer等操作
const connectToReceiver = () => {
  createRemotePeerConnection();
  const offer = {
    // 这里简单模拟接收方传来的Offer，实际要从信令服务器获取真实Offer
    type: 'offer',
    sdp: '模拟的Offer示例'
  };
  remotePeerConnection.value.setRemoteDescription(offer)
      .then(() => remotePeerConnection.value.createAnswer())
      .then((answer) => remotePeerConnection.value.setLocalDescription(answer))
      .then(() => {
        console.log('远程Answer已生成并设置，需发送给本地端（通过信令服务器）：', remotePeerConnection.value.localDescription);
        // 实际应用中要通过信令服务器将Answer发送给本地端
      });
};
</script>

<template>
  <div>
    <h1>基于原生WebRTC的文件传输</h1>
    <input type="file" @change="handleFileSelect" />
    <button @click="startTransfer" :disabled="!selectedFile">开始传输</button>
    <div v-if="localPeerId">
      <p>你的本地Peer ID：{{localPeerId}}</p>
      <p>请让接收方输入此Peer ID开启接收：</p>
      <input type="text" v-model="remotePeerId" placeholder="接收方输入 Peer ID" />
      <button @click="connectToReceiver" :disabled="!remotePeerId">连接接收方</button>
    </div>
  </div>
</template>

<style scoped>

</style>