<script setup lang="ts">
import AppPage from '@/components/AppPage.vue';
// Không còn logic giao tiếp nào ở đây nữa
</script>

<template>
  <AppPage title="Mini App Đơn Giản" :back="false">
    <div class="container">
      <div class="card">
        <h1>🚀 Đã mở thành công!</h1>
        <p>Đây là giao diện Web (Vue.js) đang chạy bên trong Android WebView.</p>
        <p class="info">Chưa có kết nối dữ liệu nào được thiết lập.</p>
      </div>
    </div>
  </AppPage>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  height: 80vh;
}
.card {
  background-color: white;
  color: black;
  padding: 30px;
  border-radius: 15px;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
h1 { color: #2481cc; margin-bottom: 10px; }
.info { color: #666; font-size: 14px; margin-top: 20px; font-style: italic;}
</style>
