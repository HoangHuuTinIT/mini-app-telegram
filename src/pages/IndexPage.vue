<script setup lang="ts">
import { onMounted, ref } from 'vue';
import AppPage from '@/components/AppPage.vue';

const userName = ref('Khách');
const userId = ref(0);
const rawDataDisplay = ref(''); // Biến này để debug xem URL nhận được gì

// HÀM TỰ XỬ LÝ: Tự đọc dữ liệu từ URL mà không cần chờ Telegram SDK
const manualParseInitData = () => {
  // 1. Lấy chuỗi Hash từ URL (cái phần sau dấu # mà Android gửi sang)
  const hash = window.location.hash.slice(1); // Bỏ dấu #
  rawDataDisplay.value = hash; // Hiển thị ra để kiểm tra

  // 2. Tìm tham số tgWebAppData
  const params = new URLSearchParams(hash);
  const tgWebAppData = params.get('tgWebAppData');

  if (tgWebAppData) {
    // 3. tgWebAppData lại là một chuỗi query string nữa, cần parse tiếp
    const dataParams = new URLSearchParams(tgWebAppData);
    const userString = dataParams.get('user');

    if (userString) {
      try {
        // 4. Parse JSON thông tin user
        const userObj = JSON.parse(userString);
        userName.value = userObj.first_name + ' ' + (userObj.last_name || '');
        userId.value = userObj.id;
        console.log("✅ Đã lấy được user:", userObj);
      } catch (e) {
        console.error("❌ Lỗi parse JSON user:", e);
      }
    }
  }
};

onMounted(() => {
  // GỌI HÀM TỰ XỬ LÝ NGAY KHI APP CHẠY
  manualParseInitData();
  // Thử báo cho Android biết là web đã sẵn sàng (nếu SDK load được)
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    (window as any).Telegram?.WebApp?.ready();

});

const handleCloseApp = () => {
  try {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const webApp = (window as any).Telegram?.WebApp;

    // Kiểm tra kỹ trước khi gọi lệnh close
    if (webApp && typeof webApp.close === 'function') {
      webApp.close();
    } else {
      alert("Không tìm thấy kết nối với Android (Telegram SDK missing).");
      // Fallback: Nếu không có SDK, thử gọi trực tiếp vào Interface bạn đã gắn
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      if ((window as any).TelegramWebviewProxy) {
          // eslint-disable-next-line @typescript-eslint/no-explicit-any
          (window as any).TelegramWebviewProxy.postEvent('web_app_close', '');
      }
    }
  } catch (e) {
    console.error("Lỗi khi đóng app:", e);
  }
};
</script>

<template>
  <AppPage title="Demo Tương Tác" :back="false">
    <div class="container">
      <div class="user-card">
        <h3>👋 Xin chào: {{ userName }}</h3>
        <p>User ID: <strong>{{ userId }}</strong></p>
        <p class="note">URL Hash nhận được:</p>
        <code class="debug-box">{{ rawDataDisplay.substring(0, 50) }}...</code>
      </div>

      <button class="btn-close" @click="handleCloseApp">
        ❌ Đóng Mini App
      </button>
    </div>
  </AppPage>
</template>

<style scoped>
.container {
  display: flex; flex-direction: column; align-items: center; gap: 20px; padding: 20px;
}
.user-card {
  background-color: var(--tg-theme-secondary-bg-color, #232e3c);
  padding: 20px; border-radius: 12px; width: 100%; word-break: break-all;
}
.btn-close {
  background-color: #ec3942; color: white; border: none; padding: 12px 24px;
  border-radius: 8px; font-weight: bold; cursor: pointer;
}
.debug-box {
    display: block; font-size: 10px; color: #aaa; margin-top: 5px;
}
</style>
