<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';
import AppPage from '@/components/AppPage.vue';

// --- 1. ĐỊNH NGHĨA DỮ LIỆU ---
interface Product {
  id: number;
  name: string;
  price: number;
  icon: string; // Dùng Emoji cho nhanh, bạn có thể thay bằng link ảnh
}

const products = ref<Product[]>([
  { id: 1, name: 'Burger', price: 4.99, icon: '🍔' },
  { id: 2, name: 'Fries', price: 1.49, icon: '🍟' },
  { id: 3, name: 'Taco', price: 3.99, icon: '🌮' },
  { id: 4, name: 'Hotdog', price: 3.49, icon: '🌭' },
  { id: 5, name: 'Pizza', price: 7.99, icon: '🍕' },
  { id: 6, name: 'Donut', price: 1.49, icon: '🍩' },
  { id: 7, name: 'Coke', price: 1.49, icon: '🥤' },
  { id: 8, name: 'Icecream', price: 5.99, icon: '🍦' },
]);

// Trạng thái giỏ hàng: { id_món: số_lượng }
const cart = ref<{ [key: number]: number }>({});
const currentView = ref<'menu' | 'order'>('menu'); // Quản lý màn hình
const userInfo = ref<string>('');

// --- 2. LOGIC TÍNH TOÁN ---
const totalPrice = computed(() => {
  let total = 0;
  for (const [id, qty] of Object.entries(cart.value)) {
    const product = products.value.find(p => p.id === Number(id));
    if (product) total += product.price * qty;
  }
  return total;
});

// --- 3. KẾT NỐI VỚI TELEGRAM (QUAN TRỌNG) ---
// Lấy đối tượng WebApp từ window
// eslint-disable-next-line @typescript-eslint/no-explicit-any
const tg = (window as any).Telegram?.WebApp;

onMounted(() => {

  // Báo cho Telegram biết App đã sẵn sàng
  tg?.ready();
  tg?.expand(); // Mở full màn hình

  // Lấy thông tin User từ Telegram thật
  if (tg?.initDataUnsafe?.user) {
    const u = tg.initDataUnsafe.user;
    userInfo.value = `${u.first_name} ${u.last_name || ''} (ID: ${u.id})`;
  } else {
    userInfo.value = "Chưa lấy được info (Đang chạy Local?)";
  }

  // Cấu hình nút Main Button (Nút xanh to ở dưới)
  tg?.MainButton.setParams({
    text: 'VIEW ORDER',
    color: '#31b545', // Màu xanh Durger King
    text_color: '#ffffff'
  });

  // Lắng nghe sự kiện bấm nút Main Button
  tg?.MainButton.onClick(() => {
    if (currentView.value === 'menu') {
      // Đang ở Menu -> Chuyển sang Order
      currentView.value = 'order';
      tg.BackButton.show(); // Hiện nút Back
    } else {
      // Đang ở Order -> Gửi đơn hàng & Đóng App
      const dataToSend = {
        cart: cart.value,
        total: totalPrice.value,
        user: tg.initDataUnsafe?.user
      };

      // Gửi dữ liệu về cho Bot (hoặc Android App sau này)
      tg.sendData(JSON.stringify(dataToSend));
    }
  });

  // Lắng nghe nút Back (trên thanh tiêu đề)
  tg?.BackButton.onClick(() => {
    currentView.value = 'menu';
    tg.BackButton.hide();
  });
});

// --- 4. THEO DÕI GIỎ HÀNG ĐỂ CẬP NHẬT NÚT ---
watch([totalPrice, currentView], ([newPrice, view]) => {
  if (newPrice > 0) {
    tg?.MainButton.show();
    if (view === 'menu') {
      tg?.MainButton.setText('VIEW ORDER');
    } else {
      tg?.MainButton.setText(`PAY $${newPrice.toFixed(2)}`);
    }
  } else {
    tg?.MainButton.hide();
  }
});

// Logic thêm bớt món
const updateCart = (product: Product, change: number) => {
  const currentQty = cart.value[product.id] || 0;
  const newQty = currentQty + change;

  // Rung nhẹ điện thoại (Haptic Feedback) chuẩn Telegram
  tg?.HapticFeedback.impactOccurred('medium');

  if (newQty <= 0) delete cart.value[product.id];
  else cart.value[product.id] = newQty;
};
</script>

<template>
  <AppPage :title="currentView === 'menu' ? 'Durger King' : 'Thanh toán'" :back="false">

    <div class="header-info">
      Chào, <b>{{ userInfo }}</b>
    </div>

    <div v-if="currentView === 'menu'" class="menu-grid">
      <div class="product-card" v-for="p in products" :key="p.id">
        <div class="p-icon">{{ p.icon }}</div>
        <div class="p-name">{{ p.name }}</div>
        <div class="p-price">${{ p.price }}</div>

        <div v-if="cart[p.id]" class="controls">
          <button class="btn-circle remove" @click="updateCart(p, -1)">-</button>
          <span class="qty">{{ cart[p.id] }}</span>
          <button class="btn-circle add" @click="updateCart(p, 1)">+</button>
        </div>
        <button v-else class="btn-add" @click="updateCart(p, 1)">ADD</button>
      </div>
    </div>

    <div v-else class="order-summary">
      <div class="order-img">🍔🍟🥤</div>
      <h3>Đơn hàng của bạn</h3>
      <div class="order-list">
        <div v-for="(qty, id) in cart" :key="id" class="order-item">
          <span>{{ products.find(p => p.id == Number(id))?.name }} x {{ qty }}</span>
          <b>${{ (products.find(p => p.id == Number(id))!.price * qty).toFixed(2) }}</b>
        </div>
      </div>
      <div class="total-line">
        <span>Tổng cộng:</span>
        <span class="total-price">${{ totalPrice.toFixed(2) }}</span>
      </div>
    </div>

    <div style="height: 100px;"></div> </AppPage>
</template>

<style scoped>
/* CSS giả lập giao diện Durger King */
.header-info {
  background: #f5f5f5; padding: 10px; text-align: center;
  font-size: 14px; color: #555;
}

.menu-grid {
  display: grid;
  grid-template-columns: 1fr 1fr; /* 2 cột */
  gap: 15px; padding: 15px;
}

.product-card {
  background: white; border-radius: 12px;
  padding: 15px; text-align: center;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.p-icon { font-size: 50px; margin-bottom: 5px; }
.p-name { font-weight: bold; margin-bottom: 2px; }
.p-price { color: #888; font-size: 14px; margin-bottom: 10px; }

.btn-add {
  background: #f1a037; color: white; border: none;
  width: 100%; padding: 8px; border-radius: 6px; font-weight: bold;
}

.controls {
  display: flex; justify-content: space-between; align-items: center;
}
.btn-circle {
  width: 28px; height: 28px; border-radius: 50%; border: none;
  color: white; font-weight: bold; font-size: 16px;
  display: flex; align-items: center; justify-content: center;
}
.add { background: #f1a037; }
.remove { background: #e64d44; }
.qty { font-weight: bold; }

/* CSS cho màn hình Order */
.order-summary { padding: 30px; text-align: center; }
.order-img { font-size: 60px; margin-bottom: 20px; }
.order-list { margin-top: 20px; text-align: left; }
.order-item {
  display: flex; justify-content: space-between;
  padding: 10px 0; border-bottom: 1px solid #eee;
}
.total-line {
  display: flex; justify-content: space-between;
  margin-top: 20px; font-size: 18px; font-weight: bold; color: #31b545;
}
</style>
