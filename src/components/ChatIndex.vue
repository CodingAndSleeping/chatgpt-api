<script lang="ts" setup>
import { AxiosResponse } from "axios";
import {
  ChatCompletionRequestMessage,
  ChatCompletionRequestMessageRoleEnum,
  Configuration,
  CreateChatCompletionResponse,
  OpenAIApi,
} from "openai";
import { ref, reactive } from "vue";

const configuration: Configuration = new Configuration({
  organization: "your-code",
  apiKey: "your-key",
});
const openai: OpenAIApi = new OpenAIApi(configuration);

// 使用的模型
const model: string = "gpt-3.5-turbo";

// 对话信息数组
const messages = reactive<ChatCompletionRequestMessage[]>([]);

// 输入内容
let inputVal = ref<string>("");

// 发送消息
async function sendMsg() {
  messages.push({
    role: "user",
    content: inputVal.value,
  });
  inputVal.value = "";
  const response: AxiosResponse<CreateChatCompletionResponse> =
    await openai.createChatCompletion({
      model,
      messages,
    });

  messages.push({
    role: response.data.choices[0].message
      ?.role as ChatCompletionRequestMessageRoleEnum,
    content: response.data.choices[0].message?.content as string,
  });
}
</script>

<template>
  <div class="chatBody">
    <div class="header">
      <el-icon><ArrowLeft /></el-icon>
      <div>chatgpt</div>
      <el-icon><MoreFilled /></el-icon>
    </div>
    <ul class="message">
      <li
        v-for="(msg, index) in messages"
        :key="index"
        :class="msg.role == 'assistant' ? 'assisClass' : 'userClass'"
      >
        <div class="chatMeg" v-show="msg.role == 'assistant'">
          <img src="../assets/robot.png" alt="gpt" />
          <div :class="msg.role == 'assistant' ? 'assisBubble' : 'userBubble'">
            {{ msg.content }}
          </div>
        </div>

        <div class="chatMeg" v-show="msg.role == 'user'">
          <div :class="msg.role == 'assistant' ? 'assisBubble' : 'userBubble'">
            {{ msg.content }}
          </div>
          <img src="../assets/user.png" alt="user" />
        </div>
      </li>
    </ul>
    <div class="input">
      <el-input
        v-model="inputVal"
        placeholder="请输入..."
        @keyup.enter="sendMsg"
      />

      <el-button type="primary" @click="sendMsg">发送</el-button>
    </div>
  </div>
</template>

<style scoped lang="scss">
.chatBody {
  position: absolute;
  display: flex;
  flex-direction: column;

  justify-content: space-between;
  height: 700px;
  width: 500px;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  background-color: #f5f5f5;

  .header {
    height: 8%;
    border-bottom: 1px solid #d6d6d6;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 15px;
  }

  .message {
    height: 90%;
    width: 100%;
    overflow-y: auto;
    margin: 0;
    padding: 10px 0;

    li {
      list-style: none;
      width: 100%;
      display: flex;

      .chatMeg {
        margin: 5px 0;
        display: flex;
        align-items: center;
        align-items: flex-start;
        img {
          height: 30px;
          width: 30px;
          padding: 0 5px;
        }

        .assisBubble {
          max-width: 300px;
          padding: 5px;
          background-color: #ffffff;
          border-radius: 5px;
          font-size: 14px;

          color: #000000;
          word-break: hyphenate;
          position: relative;
        }
        .userBubble {
          max-width: 300px;
          padding: 5px;
          background-color: #89d961;
          border-radius: 5px;
          font-size: 14px;

          color: #000000;
          word-break: hyphenate;
          position: relative;
        }

        .assisBubble::before {
          content: "";
          position: absolute;
          width: 0;
          height: 0;
          left: -10px;
          top: 10px;
          border: 5px solid;
          border-color: transparent #ffffff transparent transparent;
        }

        .userBubble::before {
          content: "";
          position: absolute;
          width: 0;
          height: 0;
          right: -10px;
          top: 10px;
          border: 5px solid;
          border-color: transparent transparent transparent #89d961;
        }
      }
    }

    .assisClass {
      justify-content: start;
    }

    .userClass {
      justify-content: end;
    }
  }

  .input {
    margin: 10px;
    display: flex;
  }
}
</style>
