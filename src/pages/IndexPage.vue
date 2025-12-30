<script setup lang="ts">
import { onMounted, unref, computed, type CSSProperties } from 'vue';
import {
  viewport,
  themeParams,
  initData
} from '@tma.js/sdk-vue';
import AppPage from '@/components/AppPage.vue';

// --- HELPER: Hàm lấy giá trị từ Signal hoặc Ref an toàn ---
// FIX LỖI "Unexpected any": Thêm dòng ignore eslint ngay bên dưới
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const safeUnwrap = (val: any) => {
  if (typeof val === 'function') {
    return val();
  }
  return unref(val);
};

// --- 1. XỬ LÝ DỮ LIỆU VIEWPORT ---
// Lấy giá trị an toàn, nếu chưa có thì trả về 0
const vpHeight = computed(() => safeUnwrap(viewport.height) || 0);
const vpWidth = computed(() => safeUnwrap(viewport.width) || 0);
const vpExpanded = computed(() => !!safeUnwrap(viewport.isExpanded));

// Kiểm tra xem đã có dữ liệu chiều cao chưa
const isViewportReady = computed(() => {
  const h = safeUnwrap(viewport.height);
  return typeof h === 'number' && h > 0;
});

// --- 2. XỬ LÝ DỮ LIỆU THEME ---
// SỬA LỖI HIỂN THỊ FUNCTION: Dùng safeUnwrap để gọi hàm lấy chuỗi màu
const btnColorText = computed(() => {
  const color = safeUnwrap(themeParams.buttonColor);
  return color ? String(color) : '#31b545';
});

const bgColorText = computed(() => {
  const color = safeUnwrap(themeParams.bgColor);
  return color ? String(color) : '#ffffff';
});

// --- STYLE COMPUTED ---
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

onMounted(async () => {
  console.log("Vue App Mounted");

  // KÍCH HOẠT THỦ CÔNG (Force Mount)
  // Nếu Viewport chưa được mount bởi init.ts, ta thử mount lại ở đây
  if (!viewport.isMounted()) {
    console.log("Viewport chưa mount, đang thử mount...");
    try {
      await viewport.mount();
      console.log("Viewport mounted thành công!");
    } catch (e) {
      console.error("Lỗi mount viewport:", e);
    }
  }
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
        <div v-else class="loading">
          <p>Đang đợi Android trả lời...</p>
          <p style="font-size: 12px; color: orange">
             (Nếu treo ở đây: Kiểm tra file init.ts hoặc code Android trả về event)
          </p>
        </div>
      </div>

      <div class="card" :style="cardBorderStyle">
        <h4>🎨 Theme Info</h4>

        <div>
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
