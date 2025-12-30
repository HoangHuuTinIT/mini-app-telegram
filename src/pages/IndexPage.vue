<script setup lang="ts">
import { onMounted, unref, computed } from 'vue';
import {
  viewport,
  themeParams,
  initData
} from '@tma.js/sdk-vue';
import AppPage from '@/components/AppPage.vue';

// --- 1. XỬ LÝ DỮ LIỆU VIEWPORT ---
// Lấy giá trị thô ra để dùng trong Template
const vpHeight = computed(() => unref(viewport.height));
const vpWidth = computed(() => unref(viewport.width));
const vpExpanded = computed(() => unref(viewport.isExpanded));
// Tạo biến check xem viewport đã sẵn sàng chưa (thay cho v-if="viewport")
const isViewportReady = computed(() => typeof unref(viewport.height) === 'number');

// --- 2. XỬ LÝ DỮ LIỆU THEME (Tạo object style sẵn ở đây) ---
// Việc tạo style object ở đây giúp Template không bị lỗi TS2345
const buttonStyle = computed(() => {
  const color = unref(themeParams.buttonColor) || '#31b545';
  return {
    backgroundColor: color,
    borderColor: color
  };
});

const bgStyle = computed(() => {
  const color = unref(themeParams.bgColor) || '#ffffff';
  return { background: color };
});

const btnColorText = computed(() => unref(themeParams.buttonColor) || '#31b545');
const bgColorText = computed(() => unref(themeParams.bgColor) || '#ffffff');


// --- 3. HÀM GỬI DATA ---
const sendToAndroid = () => {
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const proxy = (window as any).TelegramWebviewProxy;

  if (proxy) {
    // SỬA LỖI USER: Ép kiểu as any để bỏ qua lỗi kiểm tra type khắt khe của TS
    // Vì ta biết chắc chắn runtime nó sẽ chạy được
    const rawUser = unref(initData.user);
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const userSafe = rawUser as any;

    proxy.postEvent('send_data_back_to_android', JSON.stringify({
      name: userSafe?.firstName || 'User',
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
        <div v-if="isViewportReady">
          <p>Height: <b>{{ vpHeight }}px</b></p>
          <p>Width: <b>{{ vpWidth }}px</b></p>
          <p>Expanded: <b>{{ vpExpanded ? 'Yes' : 'No' }}</b></p>
        </div>
        <div v-else class="loading">Đang đợi Android trả lời...</div>
      </div>

      <div class="card" :style="{ borderColor: btnColorText }">
        <h4>🎨 Theme Info</h4>

        <div v-if="btnColorText">
          <p>Bg Color:
            <span :style="bgStyle">
              {{ bgColorText }}
            </span>
          </p>
          <p>Button Color:
            <span :style="{ background: btnColorText }">
              {{ btnColorText }}
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
