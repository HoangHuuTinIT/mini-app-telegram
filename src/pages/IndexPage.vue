<script setup lang="ts">
import { ref, onMounted } from 'vue';
import AppPage from '@/components/AppPage.vue';

const form = ref({
  name: '',
  age: '',
  job: '',
  token: '',
  userId: ''
});

// 4. KHI MỞ WEB LÊN: ĐỌC DỮ LIỆU TỪ HASH (Sau dấu #)
onMounted(() => {
  try {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const proxy = (window as any).TelegramWebviewProxy;

    if (proxy && proxy.getStartupParams) {
      // 1. Gọi hàm getStartupParams() của Android
      // Hàm này chạy đồng bộ và trả về chuỗi JSON ngay lập tức
      const jsonString = proxy.getStartupParams();

      console.log("Dữ liệu nhận từ Android:", jsonString);

      // 2. Parse JSON ra để dùng
      const data = JSON.parse(jsonString);

      form.value.name = data.name || '';
      form.value.age = data.age || '';
      form.value.job = data.job || '';
      form.value.token = data.token || '';
      form.value.userId = data.userId || '';

    } else {
      console.warn("Không tìm thấy Android Bridge (Đang chạy trên trình duyệt thường?)");
    }
  } catch (e) {
    console.error("Lỗi lấy dữ liệu khởi tạo:", e);
  }
});

// 5. KHI BẤM XÁC NHẬN (Giữ nguyên như cũ)
const sendBackToAndroid = () => {
  const dataToSend = {
    name: form.value.name,
    age: form.value.age,
    job: form.value.job
  };

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const proxy = (window as any).TelegramWebviewProxy;
  if (proxy) {
    proxy.postEvent('send_data_back_to_android', JSON.stringify(dataToSend));
  } else {
    // Chế độ test trên trình duyệt PC
    alert("Gửi về Android: " + JSON.stringify(dataToSend));
  }
};
</script>

<template>
  <AppPage title="Sửa thông tin" :back="false">
    <div class="container">
      <h3>Dữ liệu từ Android:</h3>

      <div class="form-group">
        <label>Tên:</label>
        <input v-model="form.name" />
      </div>

      <div class="form-group">
        <label>Tuổi:</label>
        <input v-model="form.age" />
      </div>

      <div class="form-group">
        <label>Nghề:</label>
        <input v-model="form.job" />
      </div>
      <p>Token nhận được: {{ form.token.substring(0, 10) }}...</p>
      <button class="btn-confirm" @click="sendBackToAndroid">
        XÁC NHẬN & QUAY VỀ
      </button>
    </div>
  </AppPage>
</template>

<style scoped>
.container { padding: 20px; }
.form-group { margin-bottom: 15px; }
label { display: block; font-weight: bold; margin-bottom: 5px; }
input {
  width: 100%; padding: 10px;
  border: 1px solid #ccc; border-radius: 8px; font-size: 16px;
}
.btn-confirm {
  width: 100%; padding: 15px; margin-top: 20px;
  background-color: #31b545; color: white; border: none;
  border-radius: 10px; font-size: 18px; font-weight: bold;
}
</style>
