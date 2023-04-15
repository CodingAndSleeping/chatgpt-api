<script lang="ts" setup>
import { AxiosResponse } from "axios";
import "github-markdown-css"
import { marked } from "marked";

import {
  ChatCompletionRequestMessage,
  ChatCompletionRequestMessageRoleEnum,
  Configuration,
  CreateChatCompletionResponse,
  OpenAIApi,
} from "openai";

import { ref, reactive, watch, nextTick } from "vue";

const configuration: Configuration = new Configuration({
  apiKey: "your-key",
});
const openai: OpenAIApi = new OpenAIApi(configuration);

// 使用的模型
const model: string = "gpt-3.5-turbo";

// 对话信息数组
const messages = reactive<ChatCompletionRequestMessage[]>([

{
  role:"user",
  content:"请用ptython写一段 css程序"
},
{
  role:"assistant",
  content:'<p>以下是一个简单的 Python 程序，它接受用户输入的数字并计算其平方：</p> <pre><code class="language-python">num = int(input(&#39;请输入一个数字：&#39;)) square = num ** 2 print(&#39;该数字的平方为：&#39;, square) </code></pre> <p>以下是稍微复杂一些的 Python 程序示例，它从用户输入的几个字符串中返回最长的单词：</p> <pre><code class="language-python">str_input = input(&#39;请输入一些字符串，以空格分隔：&#39;) str_list = str_input.split() max_word = &#39;&#39; for word in str_list: if len(word) &gt; len(max_word): max_word = word print(&#39;最长的单词是：&#39;, max_word) </code></pre> <p>该程序将输入字符串分割成一个单词列表，并遍历该列表以查找最长的单词。最后，它打印出该单词。</p>'
}
]);



// 输入内容
let inputVal = ref<string>("");

// 发送消息
async function sendMsg() {
  // 如果输入框内容为空格则不发送消息
  if (inputVal.value.trim() === "") {
    inputVal.value = "";
    return;
  } else {
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
      content: marked.parse(
        response.data.choices[0].message?.content as string
      ),
    });
  }
}

const msgList = ref<HTMLElement | any>();

// 侦听消息数组变化，滚动条滚动到最后一条消息处
watch(messages, () => {
  nextTick(() => {
    msgList.value?.scrollTo({
      top: msgList.value?.lastElementChild?.offsetTop,
      behavior: "smooth",
    });
  });
});

</script>

<template>
  <div class="chatBody">
    <header class="header">
      <el-icon>
        <ArrowLeft />
      </el-icon>
      <div>学习小助手</div>
      <el-icon>
        <MoreFilled />
      </el-icon>
    </header>
    <ul class="messageBody" ref="msgList">
      <li
        v-for="(msg, index) in messages"
        :key="index"
        :class="msg.role == 'assistant' ? 'assisClass' : 'userClass'"
      >
        <div class="chatMeg" v-show="msg.role == 'assistant'">
          <img class="assistant" src="../assets/syz.jpg" alt="gpt" />
          <div class="assisBubble markdown-body" v-html="msg.content" ></div>
        </div>

        <div class="chatMeg" v-show="msg.role == 'user'">
          <div class="userBubble">
            {{ msg.content }}
          </div>
          <img src="../assets/lzt.jpg" alt="user" />
        </div>
      </li>
    </ul>
    <footer class="footer">
      <el-input
        v-model="inputVal"
        placeholder="请输入..."
        @keyup.enter="sendMsg"
      />
      <el-button type="success" @click="sendMsg">发送</el-button>
    </footer>
  </div>
</template>

<style scoped lang="scss">
.chatBody {
  position: absolute;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 90vh;
  width: 95vw;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  background-color: #ffffff;


  .header {
    height: 8%;
    border-bottom: 1px solid #d6d6d6;
    color: #000000;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 15px;
  }

  .messageBody {
    height: 90%;
    width: 100%;
    overflow-y: auto;
    margin: 0;
    padding: 10px 0;


    li {
      list-style: none;
      // width: 100%;s
      display: flex;
      padding: 20px 40px 20px 20px;
      border-bottom: 0.5px solid #ffffff;

      .chatMeg {
        margin: 5px 0;
        display: flex;
        align-items: center;
        align-items: flex-start;

        img {
          height: 30px;
          width: 30px;
          margin: 0 10px;
          border-radius: 4px;
        }
        img.assistant {
          margin-top: 10px;
        }

        .assisBubble {
          padding: 5px;

          font-size: 14px;
          // color: #000000;
          word-break: hyphenate;
          position: relative;
        }

        .userBubble {

          padding: 5px;
          font-size: 14px;
          color: #ffffff;
          word-break: hyphenate;
          position: relative;
        }

      }
    }

    .assisClass {
      justify-content: start;
      background-color: #ffffff;

    }

    .userClass {
      justify-content: end;
      background-color: #343541;

    }
  }

  .messageBody::-webkit-scrollbar {
    width: 3px;
  }

  .messageBody:hover::-webkit-scrollbar-thumb {
    background-color: #bababa;
    border-radius: 6px;
  }

  .footer {
    padding: 10px;
    display: flex;
    border-top: 1px solid #d6d6d6;
    .el-input {
      margin-right: 5px;
    }
  }
}
</style>
