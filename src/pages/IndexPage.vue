<script setup lang="ts">
import { onMounted, unref, computed, type CSSProperties } from 'vue';
import {
  viewport,
  themeParams,
  initData
} from '@tma.js/sdk-vue';
import AppPage from '@/components/AppPage.vue';

// --- 1. XỬ LÝ DỮ LIỆU VIEWPORT ---
// Khai báo rõ kiểu trả về là number hoặc boolean
const vpHeight = computed((): number | undefined => unref(viewport.height));
const vpWidth = computed((): number | undefined => unref(viewport.width));
const vpExpanded = computed((): boolean => !!unref(viewport.isExpanded));

// FIX LỖI TS2774: Khai báo rõ ràng đây là boolean
const isViewportReady = computed((): boolean => {
  return typeof unref(viewport.height) === 'number';
});

// --- 2. XỬ LÝ DỮ LIỆU THEME ---
// FIX: Khai báo rõ ràng trả về string
const btnColorText = computed((): string => unref(themeParams.buttonColor) || '#31b545');
const bgColorText = computed((): string => unref(themeParams.bgColor) || '#ffffff');

// FIX LỖI TS2345:
// 1. Khai báo kiểu trả về là : CSSProperties
// 2. Dùng .value khi gọi các computed khác (btnColorText.value) để TS không bị nhầm lẫn
const cardBorderStyle = computed((): CSSProperties => {
  return { borderColor: btnColorText.value };
});

const bgSpanStyle = computed((): CSSProperties => {
  return { background: bgColorText.value };
});

const btnSpanStyle = computed((): CSSProperties => {
  return { background: btnColorText.value };
});

// --- 3. HÀM GỬI DATA ---
const sendToAndroid = () => {
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const proxy = (window as any).TelegramWebviewProxy;

  if (proxy) {
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

      <div class="card" :style="cardBorderStyle">
        <h4>🎨 Theme Info</h4>

        <div v-if="btnColorText">
          <p>Bg Color:
            <span :style="bgSpanStyle">
              {{ bgColorText }}
            </span>
          </p>
          <p>Button Color:
            <span :style="btnSpanStyle">
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
