<template>
  <div>
    <a href="https://vite.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <HelloWorld msg="Vite + Vue" />


  <div id="home">
    <section>
      <h1>Text Generation</h1>
      <!-- @input="generateText" -->
      <input v-model="inputText" placeholder="輸入文本..." />
      <p>生成的文本：{{ generatedText }}</p>
      <button @click="generateText">sdsdsd</button>
    </section>
  </div>



  <div id="home">
    <section>
      <h1>選單</h1>
      <!-- @input="generateText" -->
      <input v-model="query" placeholder="輸入文本..." />

      <button @click="generateMenu">選單</button>
    </section>
  </div>

  <div id="home">
    <section>
      <h1>翻譯</h1>
      <!-- @input="generateText" -->
      <input v-model="translatorWord" placeholder="輸入文本..." />

      <button @click="translator">選單</button>
    </section>
  </div>


  <!-- <div id="home">
    <section>
      <h1> 加載問答模型</h1>
      <input v-model="questionWord" placeholder="輸入文本..." />

      <button @click="answerer">問答</button>
    </section>
  </div> -->


  <div id="home">
    <section>
      <h1>客服機器人</h1>
      <!-- <input v-model="distilgpt2Word" placeholder="輸入文本..." />

      <button @click="distilgpt2">問答</button> -->
    </section>
  </div>


  <div class="chat-container">
    <div class="chat-box" ref="chatBox">
      <div v-for="(message, index) in messages" :key="index"
        :class="['message', message.isUser ? 'user-message' : 'response-message']">
        <div v-if="!message.isTyping" class="message-content"> {{ message.text }} </div>
        <div v-else class="typing-indicator">
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>
    </div>
    <div class="input-area">
      <input v-model="userMessage" type="text" placeholder="輸入訊息..." @keyup.enter="sendMessage" class="input-box" />
      <button @click="sendMessage" class="send-button">送出</button>
    </div>
  </div>
</template>

<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { ref, onMounted, reactive, nextTick } from 'vue';
import { pipeline, env } from '@xenova/transformers';

// 定義問答對庫
const faq = reactive([
  { question: "今天天氣如何?", answer: "今天是個晴朗的好天氣。" },
  // { question: "你好嗎?", answer: "我很好，謝謝你的關心！" },
  { question: "你的名字是什麼?", answer: "我是你的智能助手。" },
  { question: "如何新增用戶?", answer: "請至用戶管理頁面，新增人員，新增完畢臨時的密碼將會寄送至信箱，登入時再進行新密碼設定並登入" },
  { question: "如何修改使用者權限?", answer: "請至用戶管理頁面，選擇欲修改之人員，重新選擇權限，設定完成務必請該使用者重新登入，已使頁面權限生效" },
  { question: "如何修改密碼?", answer: "請至用戶管理頁面，選擇欲修改之人員，再行重新發送Email" },
  { question: "甚麼是角色管理?", answer: "角色管理是可自行新增角色，並設定此角色所能使用的頁面及權限" },
]);

// 計算餘弦相似度函數
const cosineSimilarity = (vec1, vec2) => {
  // console.log(vec1)
  // console.log(vec2)
  const dotProduct = vec1.reduce((sum, v, i) => sum + v * vec2[i], 0);
  const magnitude1 = Math.sqrt(vec1.reduce((sum, v) => sum + v ** 2, 0));
  const magnitude2 = Math.sqrt(vec2.reduce((sum, v) => sum + v ** 2, 0));
  return dotProduct / (magnitude1 * magnitude2);
};

// 參數設置
const inputText = ref('你好嗎?');
const generatedText = ref('');
env.HF_ACCESS_TOKEN = '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';
// 加載模型並生成文本
const loadModel = async () => {
  try {
    // text-generation
    // const generator = await pipeline('text-classification',);
    // 'feature-extraction', 'Xenova/all-MiniLM-L6-v2'
    // 'question-answering', 'Xenova/distilbert-base-uncased-distilled-squad'
    const generator = await pipeline('feature-extraction', 'Xenova/all-MiniLM-L6-v2', {
      quantized: false,
    });
    return generator;
  } catch (error) {
    console.error('模型加載失敗', error);
  }
};

const inputVector = ref('');
// 計算與問答庫中問題的相似度
const bestMatch = reactive({ answer: "抱歉，我不知道如何回答這個問題。", similarity: 0 });
const generateText = async () => {
  try {
    if (inputText.value) {
      const generator = await loadModel();
      console.log(generator)
      const result = await generator(inputText.value);
      inputVector.value = result.data;


      let idxArr = [];
      for (const { question, answer } of faq) {
        const questionFeatures = await generator(question);
        const questionVector = questionFeatures.data;
        const similarity = cosineSimilarity(inputVector.value, questionVector);
        // if (similarity > bestMatch.similarity) {
        idxArr.push(similarity)
        // }
      }

      // 將 NaN 替換為負無窮大
      const adjustedArr = idxArr.map(val => isNaN(val) ? -Infinity : val);
      console.log(adjustedArr)
      const maxSimilarity = Math.max(...adjustedArr);
      const bestMatchIndex = idxArr.indexOf(maxSimilarity);
      generatedText.value = faq[bestMatchIndex].answer;


      // const faqVectors = await Promise.all(faq.map(item => generator(item.question)));
      // const queryVector = await generator(inputText.value);

      // const similarities = faqVectors.map(faqVec => cosineSimilarity(queryVector.data, faqVec[0].data));
      // console.log('similarities', similarities)
      // const adjustedArr = similarities.map(val => isNaN(val) ? -Infinity : val);
      // const bestMatchIndex = similarities.indexOf(Math.max(...adjustedArr));
      // generatedText.value = faq[bestMatchIndex].answer;

    }
  } catch (error) {
    console.error('生成文本時出錯:', error);
    generatedText.value = "抱歉，我不知道如何回答這個問題。"
  }
};







const documents = ref([
  "會員查詢功能用於查詢所有會員的基本資訊。",
  "名單管理功能用於管理會員或客戶名單。",
  "行銷活動模組幫助用戶創建和管理行銷相關活動。",
  "活動集點功能記錄用戶參與活動的積分。",
  "輪盤遊戲功能提供抽獎的娛樂方式。",
  "首頁橫幅廣告幫助提升特定活動的曝光率。",
  "首頁菜單管理整個網站的導航功能。",
  "首頁置底廣告展示特定促銷或消息。",
  "消息推播功能用於通知用戶即時消息。",
  "優惠券菜單用於展示可用優惠券。",
  "優惠券設定功能幫助管理優惠券的條件。",
  "首頁繳費幫助用戶快速完成支付操作。",
  "繳費主頁項目展示所有可繳費的項目。",
  "繳費維護設定管理支付相關的設置。",
  "繳費交易紀錄記錄所有的支付歷史。",
  "繳費注意事項提醒用戶支付時的特別事項。",
  "門市交易功能記錄線下門市的交易。",
  "交易記錄保存所有交易的詳細資料。",
]);
const query = ref("交易記錄");
// // 將查詢和文檔轉換為向量
// const queryVector = await generator(query.value);

// 計算餘弦相似度函數
// const cosineSimilarity = (vec1, vec2) => {
//   const dotProduct = vec1.reduce((sum, v, i) => sum + v * vec2[i], 0);
//   const magnitude1 = Math.sqrt(vec1.reduce((sum, v) => sum + v ** 2, 0));
//   const magnitude2 = Math.sqrt(vec2.reduce((sum, v) => sum + v ** 2, 0));
//   return dotProduct / (magnitude1 * magnitude2);
// };

// // 計算查詢與每篇文檔的相似度
// const similarities = documentVectors.map(docVec => {
//   return cosineSimilarity(queryVector, docVec[0]);  // 取向量的第一維
// });
// // 找出最相似的文檔
// const bestMatchIndex = similarities.indexOf(Math.max(...similarities));
// console.log('最相似的文檔是：', documents.value[bestMatchIndex]);


const generateMenu = async () => {
  const generator = await loadModel();
  documentVectors.value = await Promise.all(documents.value.map(doc => generator(doc)));
  console.log('documentVectors', documentVectors.value)
  // 將查詢和文檔轉換為向量
  const queryVector = await generator(query.value);
  // 計算查詢與每篇文檔的相似度
  const similarities = documentVectors.value.map(docVec => {
    console.log(docVec)
    return cosineSimilarity(queryVector.data, docVec[0].data);  // 取向量的第一維
  });
  console.log('similarities', similarities)
  // 找出最相似的文檔
  const bestMatchIndex = similarities.indexOf(Math.max(...similarities));
  console.log('最相似的文檔是：', documents.value[bestMatchIndex]);
};
const documentVectors = ref([]);


const translatorWord = ref('Welcome to our store!');
const translator = async () => {
  const translator = await pipeline('translation', 'Xenova/nllb-200-distilled-600M');
  console.log(translator)
  // 翻譯英文到繁體中文
  const result = await translator(translatorWord.value, {
    src_lang: 'eng_Latn',
    tgt_lang: 'zho_Hant'
  });

  console.log('翻譯結果:', result[0].translation_text);
};


const questionWord = ref('誰設計了巴黎鐵塔？')
const answerer = async () => {
  const context = '巴黎鐵塔是位於法國巴黎香榭麗舍公園的一座鍛鐵格子塔。它以工程師古斯塔夫·艾菲爾的名字命名，他的公司設計並建造了這座塔。';

  // 確保 `question` 是字串
  const question = '誰設計了巴黎鐵塔？';

  // 載入模型
  const answerers = await pipeline('question-answering', 'Xenova/distilbert-base-uncased-distilled-squad');
  console.log('Question:', question);  // 確保輸出的問題是字串
  console.log('Context:', context);    // 確保上下文是字串

  try {
    // 使用 pipeline 處理問題和上下文
    const result = await answerers({
      question: question,  // 問題
      context: context     // 上下文
    });

    // 輸出答案
    console.log(result);
  } catch (error) {
    console.error('Error:', error);
  }
}



const distilgpt2Word = ref('我喜歡跟我的狗玩，')
const distilgpt2 = async () => {
  const generator = await pipeline('text-generation', 'Xenova/distilgpt2');
  const output = await generator(distilgpt2Word.value, {
    temperature: 2,
    max_new_tokens: 10,
    repetition_penalty: 1.5,
    no_repeat_ngram_size: 2,
    num_beams: 2,
    num_return_sequences: 2,
  });
  console.log(output)
}











// 定義問答對庫
const dataQA = reactive([
  // { question: "你好嗎?", answer: "我很好，謝謝你的關心！" },
  // { question: "今天天氣如何?", answer: "今天是個晴朗的好天氣。" },
  // { question: "你的名字是什麼?", answer: "我是你的智能助手。" },
  { question: "如何新增用戶?", answer: "請至用戶管理頁面，新增人員，新增完畢臨時的密碼將會寄送至信箱，登入時再進行新密碼設定並登入" },
  // { question: "加人進來?", answer: "請至用戶管理頁面，新增人員，新增完畢臨時的密碼將會寄送至信箱，登入時再進行新密碼設定並登入" },
  { question: "如何權限", answer: "請至用戶管理頁面，選擇欲修改之人員，重新選擇權限，設定完成務必請該使用者重新登入，已使頁面權限生效" },
  { question: "如何改密碼?", answer: "請至用戶管理頁面，選擇欲修改之人員，再行重新發送Email" },
  // { question: "甚麼是角色管理?", answer: "角色管理是可自行新增角色，並設定此角色所能使用的頁面及權限" },
]);
const messages = reactive([
  { text: "你好，我是你的助手，有什麼可以幫助您的嗎？", isUser: false, isTyping: false },
]);
const userMessage = ref("");
const chatBox = ref(null);

const sendMessage = async () => {
  const generator = await loadModel();


  if (!userMessage.value.trim()) return;
  messages.push({ text: userMessage.value, isUser: true, isTyping: false });
  const result = await generator(userMessage.value);
  console.log(result.data)
  inputVector.value = result.data;
  let idxArr = [];
  for (const { question, answer } of dataQA) {
    const questionFeatures = await generator(question);
    console.log(questionFeatures.data)
    const questionVector = questionFeatures.data;
    const similarity = cosineSimilarity(inputVector.value, questionVector);
    idxArr.push(similarity)
  }

  // 將 NaN 替換為負無窮大
  console.log(idxArr)
  const adjustedArr = idxArr.map(val => isNaN(val) ? -Infinity : val);
  const maxSimilarity = Math.max(...adjustedArr);
  const bestMatchIndex = idxArr.indexOf(maxSimilarity);
  generatedText.value = dataQA[bestMatchIndex].answer;

  userMessage.value = "";

  // 模擬對方的回覆
  messages.push({ text: "", isUser: false, isTyping: true });
  setTimeout(() => {
    messages[messages.length - 1] = {
      text: generatedText.value,
      isUser: false,
      isTyping: false,
    };
  }, 2000); // 模擬 2 秒延遲

  nextTick(() => {
    scrollToBottom();
  });
};

const scrollToBottom = () => {
  const chat = chatBox.value;
  chat.scrollTop = chat.scrollHeight;
};
onMounted(() => {
  console.log('模型已準備就緒');
  console.log(pipeline)
});
</script>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}











.chat-container {
  display: flex;
  flex-direction: column;
  max-width: 600px;
  height: 600px;
  margin: 0 auto;
  border: 1px solid #e4e4e4;
  border-radius: 10px;
  overflow: hidden;
  background: #f9f9f9;
}

.chat-box {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
  background: #fff;
}

.message {
  display: flex;
  margin-bottom: 10px;
}

.response-message {
  justify-content: flex-start;
}

.user-message {
  justify-content: flex-end;
}

.message-content {
  max-width: 60%;
  padding: 10px 15px;
  border-radius: 15px;
  font-size: 14px;
  line-height: 1.5;
}

.user-message .message-content {
  background: #e6f7ff;
  color: #333;
  border-bottom-right-radius: 0;
}

.response-message .message-content {
  background: #ffebcc;
  color: #333;
  border-bottom-left-radius: 0;
}

.typing-indicator {
  display: flex;
  justify-content: flex-start;
  gap: 5px;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  background-color: #999;
  border-radius: 50%;
  animation: typing 1.5s infinite;
}

.typing-indicator span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes typing {
  0% {
    transform: translateY(0);
    opacity: 0.5;
  }

  50% {
    transform: translateY(-5px);
    opacity: 1;
  }

  100% {
    transform: translateY(0);
    opacity: 0.5;
  }
}

.input-area {
  display: flex;
  padding: 10px;
  background: #f0f0f0;
  border-top: 1px solid #e4e4e4;
}

.input-box {
  flex: 1;
  padding: 8px 10px;
  font-size: 14px;
  border: 1px solid #ccc;
  border-radius: 5px;
  outline: none;
  margin-right: 10px;
}

.input-box:focus {
  border-color: #007bff;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}

.send-button {
  padding: 8px 15px;
  font-size: 14px;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.send-button:hover {
  background-color: #0056b3;
}
</style>
