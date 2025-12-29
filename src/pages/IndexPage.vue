<script setup lang="ts">
import { onMounted, ref } from 'vue';
import AppPage from '@/components/AppPage.vue';

const userName = ref('Khách');
const userId = ref(0);
const rawDataDisplay = ref('');

// 1. Hàm parse dữ liệu từ URL (Giữ nguyên của bạn)
const manualParseInitData = () => {
  const hash = window.location.hash.slice(1);
  rawDataDisplay.value = hash;
  const params = new URLSearchParams(hash);
  const tgWebAppData = params.get('tgWebAppData');

  if (tgWebAppData) {
    const dataParams = new URLSearchParams(tgWebAppData);
    const userString = dataParams.get('user');
    if (userString) {
      try {
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
  manualParseInitData();
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  (window as any).Telegram?.WebApp?.ready();
});

// 2. Hàm đóng App (Giữ nguyên của bạn)
const handleCloseApp = () => {
  try {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const webApp = (window as any).Telegram?.WebApp;
    if (webApp && typeof webApp.close === 'function') {
      webApp.close();
    } else {
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

// 3. HÀM MỚI: Mở Link Google
const handleOpenGoogle = () => {
    try {
        const urlToOpen = 'https://www.google.com';

        // Cách 1: Dùng SDK chuẩn (nếu có)
        // eslint-disable-next-line @typescript-eslint/no-explicit-any
        const webApp = (window as any).Telegram?.WebApp;

        if (webApp && typeof webApp.openLink === 'function') {
            webApp.openLink(urlToOpen);
        } else {
            // Cách 2: Gọi thủ công qua Bridge (Dành cho App Android tự code)
            const payload = JSON.stringify({ url: urlToOpen });

            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            if ((window as any).TelegramWebviewProxy) {
                // eslint-disable-next-line @typescript-eslint/no-explicit-any
                (window as any).TelegramWebviewProxy.postEvent('web_app_open_link', payload);
            } else {
                alert("Không tìm thấy kết nối với Android Host!");
            }
        }
    } catch (e) {
        console.error("Lỗi mở link:", e);
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

      <button class="btn-link" @click="handleOpenGoogle">
        🌐 Mở Google (Test Link)
      </button>

      <button class="btn-close" @click="handleCloseApp">
        ❌ Đóng Mini App
      </button>

    </div>
  </AppPage>
</template>

<style scoped>
.container {
  display: flex; flex-direction: column; align-items: center; gap: 15px; padding: 20px;
}
.user-card {
  background-color: var(--tg-theme-secondary-bg-color, #232e3c);
  padding: 20px; border-radius: 12px; width: 100%; word-break: break-all;
}

/* Style cho nút đóng (Đỏ) */
.btn-close {
  background-color: var(--tg-theme-destructive-text-color, #ec3942);
  color: white; border: none; padding: 12px 24px;
  border-radius: 8px; font-weight: bold; cursor: pointer;
  width: 100%;
}

/* Style cho nút mở link (Xanh) - Mới thêm */
.btn-link {
  background-color: var(--tg-theme-button-color, #3390ec); /* Màu xanh Telegram */
  color: var(--tg-theme-button-text-color, white);
  border: none; padding: 12px 24px;
  border-radius: 8px; font-weight: bold; cursor: pointer;
  width: 100%;
}

.debug-box {
    display: block; font-size: 10px; color: #aaa; margin-top: 5px;
}
</style>
