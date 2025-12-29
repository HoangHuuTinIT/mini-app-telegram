<script setup lang="ts">
import { ref, onMounted } from 'vue';
import AppPage from '@/components/AppPage.vue';

const form = ref({
  name: '',
  age: '',
  job: ''
});

// 4. KHI MỞ WEB LÊN: ĐỌC DỮ LIỆU TỪ HASH (Sau dấu #)
onMounted(() => {
  // Hash sẽ có dạng: #tgWebAppData=...&name=Tin&age=20...
  // .slice(1) để bỏ dấu # ở đầu đi
  const hashString = window.location.hash.slice(1);
  const params = new URLSearchParams(hashString);

  form.value.name = params.get('name') || '';
  form.value.age = params.get('age') || '';
  form.value.job = params.get('job') || '';
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
