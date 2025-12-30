<script setup lang="ts">
import { onMounted, unref, computed } from 'vue';
import {
  viewport,
  themeParams,
  initData
} from '@tma.js/sdk-vue';
import AppPage from '@/components/AppPage.vue';

// --- 1. XỬ LÝ DỮ LIỆU USER AN TOÀN ---
// Tạo một computed để lấy dữ liệu User ra khỏi vỏ bọc Ref
const userData = computed(() => unref(initData.user));

// --- 2. XỬ LÝ DỮ LIỆU VIEWPORT (Mở gói ra số/boolean) ---
const vpHeight = computed(() => unref(viewport.height));
const vpWidth = computed(() => unref(viewport.width));
const vpExpanded = computed(() => unref(viewport.isExpanded));

// --- 3. XỬ LÝ DỮ LIỆU THEME (Mở gói ra chuỗi màu) ---
// Nếu không có màu thì trả về màu mặc định để tránh lỗi undefined
const btnColor = computed(() => unref(themeParams.buttonColor) || '#31b545');
const bgColor = computed(() => unref(themeParams.bgColor) || '#ffffff');

// Hàm gửi dữ liệu về Android
const sendToAndroid = () => {
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const proxy = (window as any).TelegramWebviewProxy;

  if (proxy) {
    // Lúc này userData.value chắc chắn là Object User hoặc undefined
    const user = userData.value;

    proxy.postEvent('send_data_back_to_android', JSON.stringify({
      name: user?.firstName || 'User',
      action: 'Click Button Vue',
      timestamp: Date.now()
    }));
  } else {
    alert("Không tìm thấy Android Bridge! Đang chạy trên Web thường?");
  }
};

onMounted(() => {
  console.log("Vue App Mounted");
});
</script>

<template>
  <AppPage title="Android Bridge Test" :back="false">
    <div class="status-container">
      <h3>Kết nối Android Host</h3>

      <div class="card">
        <h4>📱 Viewport Info</h4>
        <div v-if="viewport">
          <p>Height: <b>{{ vpHeight }}px</b></p>
          <p>Width: <b>{{ vpWidth }}px</b></p>
          <p>Expanded: <b>{{ vpExpanded ? 'Yes' : 'No' }}</b></p>
        </div>
        <div v-else class="loading">Đang đợi Android trả lời...</div>
      </div>

      <div class="card" :style="{ borderColor: btnColor }">
        <h4>🎨 Theme Info</h4>
        <div v-if="themeParams">
          <p>Bg Color:
            <span :style="{ background: bgColor }">
              {{ bgColor }}
            </span>
          </p>
          <p>Button Color:
            <span :style="{ background: btnColor }">
              {{ btnColor }}
            </span>
          </p>
        </div>
        <div v-else class="loading">Đang đợi Android trả lời...</div>
      </div>

      <button class="btn-android" @click="sendToAndroid">
        Gửi Data & Đóng App Android
      </button>

    </div>
  </AppPage>
</template>

<style scoped>
.status-container { padding: 15px; }
.card {
  background: #f5f5f5; border-radius: 10px; padding: 15px; margin-bottom: 15px;
  border-left: 5px solid #ccc; box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.loading { color: #888; font-style: italic; }
.btn-android {
  width: 100%; padding: 15px; background-color: #31b545; color: white;
  border: none; border-radius: 8px; font-weight: bold; font-size: 16px; cursor: pointer;
}
.btn-android:active { opacity: 0.8; }
span {
  display: inline-block; padding: 2px 6px; border-radius: 4px;
  border: 1px solid #ddd; font-family: monospace;
}
</style>
