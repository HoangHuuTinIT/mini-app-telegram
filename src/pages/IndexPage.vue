<script setup lang="ts">
import { onMounted, ref } from 'vue';
import AppPage from '@/components/AppPage.vue';

// 1. Khai báo biến để hứng dữ liệu
const userName = ref('Khách');
const userId = ref(0);

// 2. Hàm lấy thông tin từ App Android gửi sang
onMounted(() => {
  // Bùa chú: Dòng dưới bảo ESLint "Đừng bắt lỗi any ở dòng tiếp theo nữa"
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const webApp = (window as any).Telegram?.WebApp;

  if (webApp && webApp.initDataUnsafe?.user) {
    userName.value = webApp.initDataUnsafe.user.first_name + ' ' + webApp.initDataUnsafe.user.last_name;
    userId.value = webApp.initDataUnsafe.user.id;

    // Báo cho Android biết là Web đã load xong
    webApp.ready();
  }
});

// 3. Hàm gọi ngược về Android: Yêu cầu đóng App
const handleCloseApp = () => {
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  (window as any).Telegram?.WebApp.close();
};
</script>

<template>
  <AppPage title="Demo Tương Tác" :back="false">
    <div class="container">
      <div class="user-card">
        <h3>👋 Xin chào: {{ userName }}</h3>
        <p>User ID: <strong>{{ userId }}</strong></p>
        <p class="note">(Dữ liệu này do Android App gửi sang)</p>
      </div>

      <button class="btn-close" @click="handleCloseApp">
        ❌ Đóng Mini App
      </button>

      <p class="explain">
        Khi bấm nút trên, Web sẽ gọi lệnh <code>close()</code>,
        Android sẽ bắt được và đóng Activity.
      </p>
    </div>
  </AppPage>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  text-align: center;
  padding: 20px 0;
}

.user-card {
  background-color: var(--tg-theme-secondary-bg-color, #232e3c);
  padding: 20px;
  border-radius: 12px;
  width: 100%;
}

.btn-close {
  background-color: var(--tg-theme-destructive-text-color, #ec3942);
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

.btn-close:active {
  opacity: 0.8;
}

.note, .explain {
  color: var(--tg-theme-hint-color, #7d8b99);
  font-size: 14px;
}
</style>
