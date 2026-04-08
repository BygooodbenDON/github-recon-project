<script setup>
import { ref } from 'vue'; // 從 vue 裡面拿出「神奇盒子」工具

const searchText = ref(''); // 準備一個叫 searchText 的盒子，用來裝搜尋框寫的字
const userInfo = ref(null); // 準備一個叫 userInfo 的盒子，用來裝 GitHub 的名片資料
const userRepos = ref([]); // 準備一個叫 userRepos 的盒子，用來裝一整袋的作品倉庫
const hasError = ref(false); // 準備一個叫 hasError 的開關，紀錄有沒有出錯
const isLoading = ref(false); // 準備一個叫 isLoading 的開關，紀錄是不是「正在外送中」

// 定義一個「點擊小喇叭」響起時要做的動作
const searchUser = async () => {
  if (!searchText.value) return; 
  
  // 準備動作：開啟外送中開關，關掉錯誤開關，清空舊盒子
  isLoading.value = true;
  hasError.value = false;
  userInfo.value = null;
  userRepos.value = [];

  try {
    // 關卡一：請外送員拿個人名片資料
    const userResponse = await fetch(`https://api.github.com/users/${searchText.value}`);
    
    if (userResponse.ok) {
      const userData = await userResponse.json();
      userInfo.value = userData;

      // 關卡二：再請外送員跑一趟拿作品清單 (只拿前 5 個最酷的作品)
      const reposResponse = await fetch(`https://api.github.com/users/${searchText.value}/repos?sort=updated&per_page=5`);
      const reposData = await reposResponse.json();
      userRepos.value = reposData;
    } else {
      hasError.value = true; // 找不到人
    }
  } catch (e) {
    hasError.value = true; // 連線出問題
  } finally {
    isLoading.value = false; // 無論成功失敗，外送員都回來了，關閉外送中開關
  }
};
</script>

<template>
  <div class="container">
    <h1>GitHub 偵察機 🚀</h1>
    
    <div class="search-box">
      <input v-model="searchText" @keyup.enter="searchUser" placeholder="輸入 GitHub 帳號..." />
      <button @click="searchUser">開始偵察</button>
    </div>

    <!-- 正在外送中（讀取中）的提示 -->
    <div v-if="isLoading" class="loading">
      <div class="spinner"></div>
      <p>外送員正在飛奔 GitHub 倉庫中...</p>
    </div>

    <!-- 偵察結果 -->
    <div v-if="userInfo && !isLoading" class="result">
      <div class="profile">
        <img :src="userInfo.avatar_url" width="120" />
        <div class="info">
          <h2>{{ userInfo.name || userInfo.login }}</h2>
          <p class="bio">{{ userInfo.bio || '這個人很神秘，什麼都沒寫。' }}</p>
          
          <!-- 戰績勳章（數據） -->
          <div class="stats">
            <div class="stat-item">
              <span class="label">作品數</span>
              <span class="value">{{ userInfo.public_repos }}</span>
            </div>
            <div class="stat-item">
              <span class="label">粉絲</span>
              <span class="value">{{ userInfo.followers }}</span>
            </div>
            <div class="stat-item">
              <span class="label">追蹤中</span>
              <span class="value">{{ userInfo.following }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 作品清單：排排站指令 -->
      <div class="repos-section">
        <h3>近期作品倉庫 📦</h3>
        <div class="repos-list">
          <a v-for="repo in userRepos" :key="repo.id" :href="repo.html_url" target="_blank" class="repo-card">
            <h4>{{ repo.name }}</h4>
            <p>{{ repo.description || '無描述' }}</p>
            <span class="star">⭐ {{ repo.stargazers_count }}</span>
          </a>
        </div>
      </div>
    </div>

    <!-- 查無此人 -->
    <div v-if="hasError && !isLoading" class="error-msg">
      <p>查無此人 🔍</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 50px 20px;
  text-align: center;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #1a1a1a;
  color: #ffffff;
  min-height: 100vh;
}

.search-box {
  margin-bottom: 40px;
}

input {
  padding: 12px 20px;
  font-size: 16px;
  border-radius: 30px;
  border: 1px solid #444;
  background-color: #333;
  color: white;
  width: 250px;
  outline: none;
  transition: 0.3s;
}

input:focus {
  border-color: #42b883;
  width: 300px;
}

button {
  padding: 12px 25px;
  font-size: 16px;
  background-color: #42b883;
  color: white;
  border: none;
  cursor: pointer;
  margin-left: -45px;
  border-radius: 30px;
  font-weight: bold;
}

.loading {
  margin-top: 50px;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #333;
  border-top: 4px solid #42b883;
  border-radius: 50%;
  margin: 0 auto 20px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.result {
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.profile {
  display: flex;
  align-items: center;
  gap: 30px;
  background-color: #2a2a2a;
  padding: 30px;
  border-radius: 15px;
  text-align: left;
  margin-bottom: 30px;
}

.profile img {
  border-radius: 50%;
  border: 4px solid #42b883;
  box-shadow: 0 0 20px rgba(66, 184, 131, 0.3);
}

.info h2 {
  margin: 0 0 10px 0;
  font-size: 28px;
}

.bio {
  color: #aaa;
  margin-bottom: 20px;
}

.stats {
  display: flex;
  gap: 20px;
}

.stat-item {
  background-color: #333;
  padding: 10px 15px;
  border-radius: 8px;
  text-align: center;
  flex: 1;
}

.stat-item .label {
  display: block;
  font-size: 12px;
  color: #888;
  margin-bottom: 5px;
}

.stat-item .value {
  font-size: 18px;
  font-weight: bold;
  color: #42b883;
}

.repos-section {
  text-align: left;
}

.repos-list {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-top: 15px;
}

.repo-card {
  background-color: #2a2a2a;
  padding: 20px;
  border-radius: 10px;
  text-decoration: none;
  color: white;
  border: 1px solid #444;
  transition: 0.3s;
}

.repo-card:hover {
  border-color: #42b883;
  background-color: #333;
  transform: translateY(-5px);
}

.repo-card h4 {
  margin: 0 0 10px 0;
  color: #42b883;
}

.repo-card p {
  font-size: 14px;
  color: #aaa;
  margin-bottom: 15px;
  height: 40px;
  overflow: hidden;
}

.star {
  font-size: 12px;
  color: #ffca28;
}

.error-msg {
  margin-top: 50px;
  color: #ff7875;
  font-size: 20px;
}

@media (max-width: 600px) {
  .profile {
    flex-direction: column;
    text-align: center;
  }
  .stats {
    justify-content: center;
  }
  .repos-list {
    grid-template-columns: 1fr;
  }
}
</style>
