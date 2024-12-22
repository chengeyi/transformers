
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
</template>

<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { ref, onMounted } from 'vue';
import { pipeline } from '@xenova/transformers';

// 參數設置
const inputText = ref('');
const generatedText = ref('');

// 加載模型並生成文本
const loadModel = async () => {
  try {
    const generator = await pipeline('text-generation');
    return generator;
  } catch (error) {
    console.error('模型加載失敗', error);
  }
};

const generateText = async () => {
  try {
    if (inputText.value) {
      const generator = await loadModel();
      const result = await generator(inputText.value,{ max_length: 50, temperature: 0.7 });
      console.log(result)
      generatedText.value = result[0]?.generated_text || '生成文本失敗';
    }
  } catch (error) {
    console.error('生成文本時出錯:', error);}
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
</style>
