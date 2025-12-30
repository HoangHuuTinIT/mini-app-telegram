<script setup lang="ts">
import { onMounted, unref, computed } from 'vue';
import {
  viewport,
  themeParams,
  initData
} from '@tma.js/sdk-vue';
import AppPage from '@/components/AppPage.vue';

// --- 1. XỬ LÝ DỮ LIỆU VIEWPORT ---
const vpHeight = computed(() => unref(viewport.height));
const vpWidth = computed(() => unref(viewport.width));
const vpExpanded = computed(() => unref(viewport.isExpanded));

// Sửa lỗi TS2774: Kiểm tra kỹ hơn hoặc dùng !! để ép kiểu boolean rõ ràng
// viewport.height trả về number, check !== undefined an toàn hơn
const isViewportReady = computed(() => unref(viewport.height) !== undefined);

// --- 2. XỬ LÝ DỮ LIỆU THEME ---
// Helper lấy mã màu string raw
const btnColorText = computed(() => unref(themeParams.buttonColor) || '#31b545');
const bgColorText = computed(() => unref(themeParams.bgColor) || '#ffffff');

// FIX LỖI TS2345: Tạo các Style Object Computed riêng biệt
// Thay vì viết inline object trong template, ta tạo object hoàn chỉnh ở đây
// Lúc này unref() sẽ lấy giá trị string ra, TS sẽ hiểu đây là object CSS hợp lệ.

// Style cho Button (Card Background)
const btnStyle = computed(() => {
  const color = unref(btnColorText);
  return {
    backgroundColor: color,
    borderColor: color
  };
});

// Style cho Background text
const bgSpanStyle = computed(() => {
  return { background: unref(bgColorText) };
});

// Style cho Button text span
const btnSpanStyle = computed(() => {
  return { background: unref(btnColorText) };
});

// Style cho Border Card
const cardBorderStyle = computed(() => {
  return { borderColor: unref(btnColorText) };
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
