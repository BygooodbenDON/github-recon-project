<script setup>
import { ref } from 'vue'; // 從 vue 裡面拿出「神奇盒子」工具

const searchText = ref(''); // 準備一個叫 searchText 的盒子，用來裝搜尋框寫的字
const userInfo = ref(null); // 準備一個叫 userInfo 的盒子，用來裝 GitHub 的資料，一開始是空的
const hasError = ref(false); // 準備一個叫 hasError 的開關，紀錄有沒有出錯，預設是「沒錯」 (false)

// 定義一個「點擊小喇叭」響起時要做的動作
const searchUser = async () => {
  if (!searchText.value) return; // 如果盒子是空的，就什麼都不做
  
  hasError.value = false; // 每次搜尋前，先把錯誤開關關掉
  userInfo.value = null; // 也把舊的資料盒子清空

  // 請「外送員」去 GitHub 拿資料
  const response = await fetch(`https://api.github.com/users/${searchText.value}`);
  
  // 檢查外送員有沒有順利拿到東西
  if (response.ok) {
    const data = await response.json();
    userInfo.value = data; // 拿到資料了，放進盒子裡
  } else {
    hasError.value = true; // 哎呀，外送員空手而回（可能找不到人），把錯誤開關打開
  }
};
</script>

<template>
  <div class="container">
    <h1>GitHub 偵察機 🚀</h1>
    
    <div class="search-box">
      <!-- 用「神奇盒子」同步輸入的文字 -->
      <input v-model="searchText" placeholder="輸入 GitHub 帳號..." />
      <!-- 點擊時吹響「小喇叭」，執行 searchUser -->
      <button @click="searchUser">開始偵察</button>
    </div>

    <!-- 這是「隱身開關」，只有當 userInfo 盒子有東西時才會出現 -->
    <div v-if="userInfo" class="result">
      <img :src="userInfo.avatar_url" width="100" />
      <h2>{{ userInfo.name || userInfo.login }}</h2>
      <p>{{ userInfo.bio }}</p>
    </div>

    <!-- 這是另一個「隱身開關」，只有當 hasError 是開著的時候才會出現 -->
    <div v-if="hasError" class="error-msg">
      <p>查無此人 🔍</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 50px 20px;
  text-align: center;
  font-family: sans-serif;
  background-color: #1a1a1a; /* 換成深色背景 */
  color: #ffffff; /* 文字變成白色 */
  min-height: 100vh;
}
.search-box {
  margin-bottom: 20px;
}
input {
  padding: 10px;
  font-size: 16px;
  border-radius: 4px;
  border: 1px solid #444;
  background-color: #333;
  color: white;
}
button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #42b883;
  color: white;
  border: none;
  cursor: pointer;
  margin-left: 10px;
  border-radius: 4px;
}
.result {
  border: 1px solid #444;
  padding: 20px;
  border-radius: 8px;
  background-color: #2a2a2a;
}
.result img {
  border-radius: 50%; /* 讓照片變圓圈圈 */
  border: 3px solid #42b883; /* 幫照片加個綠色邊框 */
}
.error-msg {
  color: #ff7875;
  background-color: #431414;
  border: 1px solid #a61d24;
  padding: 10px;
  border-radius: 4px;
  margin-top: 20px;
}
</style>
