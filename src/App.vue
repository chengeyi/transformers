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
</template>

<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { ref, onMounted, reactive } from 'vue';
import { pipeline, env } from '@xenova/transformers';

// 定義問答對庫
const faq = reactive([
  { question: "你好嗎?", answer: "我很好，謝謝你的關心！" },
  { question: "今天天氣如何?", answer: "今天是個晴朗的好天氣。" },
  { question: "你的名字是什麼?", answer: "我是你的智能助手。" },
]);

// 計算餘弦相似度函數
const cosineSimilarity = (vec1, vec2) => {
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
      // const result = await generator(inputText.value,{ max_length: 50, temperature: 0.7 });
      // generatedText.value = result[0]?.generated_text || '生成文本失敗';
      const result = await generator(inputText.value);
      inputVector.value = result.data;


      let idxArr = [];
      for (const { question, answer } of faq) {
        const questionFeatures = await generator(question);
        const questionVector = questionFeatures.data;
        const similarity = cosineSimilarity(inputVector.value, questionVector);

        if (similarity > bestMatch.similarity) {
          idxArr.push(similarity)
        }
      }
      console.log(idxArr)
      const maxSimilarity = Math.max(...idxArr);
      const bestMatchIndex = idxArr.indexOf(maxSimilarity);
      generatedText.value = faq[bestMatchIndex].answer;

    }
  } catch (error) {
    console.error('生成文本時出錯:', error);
  }
};







const documents = ref([
  "會員查詢",
  "名單管理",
  "今天的新聞關於最近的經濟學會議。",
  "行銷活動",
  "活動集點",
  "輪盤遊戲",
  "首頁橫幅廣告",
  "首頁菜單",
  "首頁置底廣告",
  "消息推播",
  "優惠券菜單",
  "優惠券設定",
  "首頁繳費",
  "繳費主頁項目",
  "繳費維護設定",
  "繳費交易紀錄",
  "繳費注意事項",
  "門市交易",
  "交易記錄",
]);
const query = ref("");
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


const generateMenu = async() => {
  const generator = await loadModel();
  documentVectors.value = await Promise.all(documents.value.map(doc => generator(doc)));
  // 將查詢和文檔轉換為向量
  const queryVector = await generator(query.value);

  // 計算查詢與每篇文檔的相似度
  const similarities = documentVectors.value.map(docVec => {
    return cosineSimilarity(queryVector, docVec[0]);  // 取向量的第一維
  });
  // 找出最相似的文檔
  const bestMatchIndex = similarities.indexOf(Math.max(...similarities));
  console.log('最相似的文檔是：', documents.value[bestMatchIndex]);
};
const documentVectors = ref([]);

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
</style>
