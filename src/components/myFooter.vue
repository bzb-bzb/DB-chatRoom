<template>
  <div>
    <el-input v-model="input" placeholder="请输入内容" @keyup.enter="sendMessage">
      <template #suffix>
        <div class="input-suffix">
          <!-- <el-button @click="showEmojiPicker">😊</el-button> -->
          <!-- <button class="emoji-button" ref="emojiButton" @click="toggleEmojiPicker">😊</button> -->
          <img src="@/assets/emoji.jpg" class="emoji-image" ref="emojiButton" @click="toggleEmojiPicker"/>
          <!-- <emoji-picker @select-emoji="addEmoji" /> -->
          <image-upload />
        </div>
      </template>
      <template #append>
        <el-button @click="sendMessage">发送</el-button> 
      </template>
    </el-input>
  </div>
</template>


<script>
import store from "../store/index.js";
// import EmojiPicker from './EmojiPicker.vue';
import ImageUpload from './ImageUpload.vue';
import { EmojiButton } from '@joeattardi/emoji-button';

export default {
  name: 'myFooter',
  components: {
    // EmojiPicker,
    ImageUpload
  },
  data() {
    return {
      input: ''
    };
  },
  mounted() {
      this.emojiPicker = new EmojiButton({
          position: 'bottom-start',
          zIndex: 9999,
      });
      this.emojiPicker.on('emoji', selection => {
          this.input += selection.emoji;
      });
  },
  methods: {
    toggleEmojiPicker() {
        document.body.style.overflow = this.emojiPicker.isOpen ? '' : 'hidden';
        this.emojiPicker.togglePicker(this.$refs.emojiButton);
    },
    
    // showEmojiPicker() {
    //   const picker = new EmojiButton();
      
    //   picker.on('emoji', selection => {
    //     // 将选择的表情符号添加到输入框中
    //     this.input += selection.emoji;
    //   });

    //   picker.togglePicker();
    // },

    async sendMessageTo() {
      try {
        // 从 store.state 中提取 receiverID 和 senderID
        const receiverID = store.state.targetInfomation.id;
        const senderID = store.state.myInformation.id;
        const token = store.state.token; // 假设 token 存储在 Vuex 的 store 中,在测试的时候直接拿一个字符串
        // 获取当前时间
        const currentTime = new Date().toLocaleString();

        // 构建请求体
        const messageData = {
          receiverID: receiverID,
          senderID: senderID,
          time:currentTime,
          messageType: '', // 这里为空字符串
          content: this.input, // 获取用户输入的消息内容
        };

        // 发送 POST 请求到后端 API
        const response = await axios.post(`http://localhost:8080/api/Message/SendMessage?token=${token}`, messageData);
        console.log('Message sent successfully:', response.data);

      } catch (error) {
        console.error('Error sending message:', error);
      }
    },

    sendMessage() {
      //控制台打印用户输入信息
      console.log('用户输入的信息为：', this.input);

      // 获取当前时间
      const currentTime = new Date().toLocaleString();

      if(this.input){
        var msg={
          targetName: store.state.targetInfomation.name,
          targetId:store.state.targetInfomation.id,
          list:{
            is_me: true,//用来判断是聊天对象发送的消息还是我发送的消息
            time: currentTime,//发送信息的时间
            message: this.input,
            messageType: '', // 这里为空字符串
          }
        }
        store.commit('addMessage', msg);
      }
      else{
        this.$notify({
          title: '提示',
          message: '输入不能为空',
          type: 'info',
          position: 'bottom-right', // 将通知显示在右下角
        });
      }
      //调用后端API发送信息，同时需要调用API来监听并接收信息

      this.sendMessageTo();

      //需要将输入框进行清空
      this.input='';
      console.log('清空输入框,输入框内容为：',this.input);
    },
    addEmoji(emoji) {
      this.input += emoji;
    }
  }
}
</script>

<style>
.input-suffix {
  display: flex;
  align-items: center; /* 确保子元素在垂直方向上居中对齐 */
  gap: 10px; /* 控制两个组件之间的间距 */
}
.emoji-image {
  width: 16px; /* 根据需要调整图片尺寸 */
  height: 16px;
}
</style>
