<script lang="ts" setup>
import { marked } from "marked";
import { ChatCompletionRequestMessage } from "openai";

import { ref, reactive, nextTick } from "vue";

// 使用的模型
const model: string = "gpt-3.5-turbo";

// 对话信息数组
const messages = reactive<ChatCompletionRequestMessage[]>([]);

// 输入内容
let inputVal = ref<string>("");

// 输入框是否禁用
let isDisabled = ref<boolean>(false);

// 消息列表
const msgList = ref<HTMLUListElement>();

// 发送消息
async function sendMsg() {
  // 如果输入框内容为空格则不发送消息
  if (inputVal.value.trim() === "") {
    inputVal.value = "";
    return;
  } else {
    // 消息数组推入用户消息
    messages.push({
      role: "user",
      content: inputVal.value,
    });
    // 消息数组推入gpt响应消息
    messages.push({
      role: "assistant",
      content: "",
    });
    // 输入框置空
    inputVal.value = "";

    // 输入框禁用
    isDisabled.value = true;

    // 滚动条滚动到最新消息
    nextTick(() => {
      msgList.value?.scrollTo({
        top: msgList.value!.scrollHeight,
        behavior: "smooth",
      });
    });

    // 使用fetch发送请求
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      body: JSON.stringify({
        model,
        messages,
        stream: true, // stream分流开启
      }),
      headers: {
        "Content-Type": "application/json",
        Authorization: "Bearer your-apikey",
      },
    });

    // 判断状态码是否为200
    if (response.status !== 200) {
      isDisabled.value = false;
      return;
    }

    // 定义一个文本解析器，用于解析二进制数据
    const decoder = new TextDecoder("utf-8");

    // 调用ReadableStream对象上的getReader方法，
    // 返回ReadableStreamDefaultReader对象
    const reader = response.body?.getReader();

    // 定义一个空文本字符串用于拼接数据流的内容
    let content = "";

    // 循环获取数据流
    while (true) {
      // ReadableStreamDefaultReader每调用一次read方法可以返回一个数据流
      // 包含一个done和value，
      // done是结束标识，当为true时则表示结束
      // value则为每一次的数据
      const { done, value } = await reader!.read();
      // 如果done为true 则停止本次循环
      if (done) break;

      // 解析数据
      const text = decoder.decode(value);

      // 判断解析出来的是否为<ERR>
      if (text === "<ERR>") break;

      // 将text以回车符分割
      const dataArr: string[] = text.split("\n\n");
      // 遍历分割出来的数组
      for (const data of dataArr) {
        // 每条数据为 data:{xxx},因此对其进行截取
        const jsonStr = data.slice(5, data.length);

        // 判断jsonStr是否为 [DONE] 或 ""
        if (jsonStr.includes("[DONE]") || jsonStr === "") break;

        // 转为json对象
        const json = JSON.parse(jsonStr);

        // 判断finish_reason是否为"stop", 若是,则表示最后一条数组
        if (json.choices[0].finish_reason === "stop") {
          isDisabled.value = false; // 输入框打开
          break;
        }

        if (json.choices[0].delta.hasOwnProperty("content")) {
          // 内容的拼接
          content += json.choices[0].delta.content;

          // markdown 转为 html
          messages[messages.length - 1].content = marked.parse(content);

          // 滚轮定位到指定位置
          msgList.value?.scrollTo({
            top: msgList.value!.scrollHeight,
            behavior: "smooth",
          });
        }
      }
    }
  }
}
</script>

<template>
  <div class="chatBody">
    <header class="header">
      <div>学习小助手</div>
    </header>
    <ul class="messageBody" ref="msgList">
      <li
        v-for="(msg, index) in messages"
        :key="index"
        :class="msg.role == 'assistant' ? 'assisClass' : 'userClass'"
      >
        <div class="chatMeg" v-show="msg.role == 'assistant'">
          <img class="assistant" src="../assets/syz.jpg" alt="gpt" />
          <div class="assisBubble markdown-body" v-html="msg.content"></div>
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
        :disabled="isDisabled"
        v-model="inputVal"
        placeholder="请输入问题..."
        @keyup.enter="sendMsg"
      />
      <el-button type="info" @click="sendMsg">发送</el-button>
    </footer>
  </div>
</template>

<style scoped lang="scss">
.chatBody {
  position: absolute;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  height: 100vh;
  width: 100vw;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto;
  background-color: #343541;

  .header {
    height: 8vh;
    width: 100vw;
    border-bottom: 1px solid #d6d6d6;
    color: #ffffff;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .messageBody {
    height: 90vh;
    width: 100vw;
    overflow-y: auto;
    padding: 10px 0;
    display: flex;
    flex-direction: column;
    align-items: center;

    li {
      list-style: none;
      width: 60vw;
      display: flex;
      padding: 20px 20px 20px 20px;

      .chatMeg {
        margin: 5px 0;
        display: flex;
        align-items: flex-start;

        img {
          height: 30px;
          width: 30px;
          margin: 0 10px;
          border-radius: 4px;
        }

        .assisBubble {
          padding: 5px;
          max-width: 55vw;
          word-break: hyphenate;
          position: relative;
        }

        .userBubble {
          padding: 5px;
          color: #ffffff;
          word-break: hyphenate;
          position: relative;
        }
      }
    }

    .assisClass {
      justify-content: start;
      background-color: #0d1117;
    }

    .userClass {
      justify-content: end;
      background-color: #444654;
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
    width: 40vw;
    padding: 10px;
    display: flex;

    .el-input {
      margin-right: 5px;
    }
  }
}
</style>
