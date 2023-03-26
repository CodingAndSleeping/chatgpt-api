<script lang="ts" setup>
// import {ElButton} from 'element-plus';
import { AxiosResponse } from "axios";
import {
  ChatCompletionRequestMessage,
  ChatCompletionRequestMessageRoleEnum,
  ChatCompletionResponseMessage,
  Configuration,
  CreateChatCompletionResponse,
  OpenAIApi,
} from "openai";
import { ref, reactive } from "vue";

const configuration: Configuration = new Configuration({
  organization: "org-SPkH5Jw420EOxS7VHFDSPiev",
  apiKey: "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",
});
const openai: OpenAIApi = new OpenAIApi(configuration);

// 使用的模型
const model: string = "gpt-3.5-turbo";

// 对话信息数组
const messages = reactive<ChatCompletionRequestMessage[]>([
  {
    role: "user",
    content: "你好",
  },

  {
    role: "assistant",
    content: "你好，请问有什么需要帮您的吗？",
  },
]);

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
    <ul class="message">
      <li v-for="(msg, index) in messages" :key="index">
        <div
          v-show="msg.role == 'assistant'"
          :class="msg.role == 'assistant' ? 'assisClass' : 'userClass'"
        >
          <img src="../assets/robot.png" alt="gpt" />
          <span>{{ msg.content }}</span>
        </div>

        <div
          v-show="msg.role == 'user'"
          :class="msg.role == 'assistant' ? 'assisClass' : 'userClass'"
        >
          <span>{{ msg.content }}</span>
          <img src="../assets/头像.jpg" alt="user" />
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
  display: flex;
  flex-direction: column;

  justify-content: space-between;
  height: 700px;
  width: 500px;
  margin: auto;
  padding: 20px;

  border: 1px solid red;

  .message {
    margin: 0;
    padding: 0;
    li {
      list-style: none;
      div {
        margin: 5px 0;
        display: flex;
        align-items: center;
      }

      img {
        height: 20px;
        width: 20px;
		padding: 0 5px;
      }

	  .assisClass{
		justify-content: start;
	  }

	  .userClass{
		justify-content: end;
	  }
    }
  }

  .input {
    display: flex;
  }
}
</style>
