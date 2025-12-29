<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue';
import AppPage from '@/components/AppPage.vue';

// 1. ĐỊNH NGHĨA KIỂU DỮ LIỆU (INTERFACE)
// Giúp TypeScript hiểu "Món ăn" gồm những thông tin gì
interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

// 2. Danh sách món ăn (Gán kiểu Product[])
const products = ref<Product[]>([
  { id: 1, name: 'Burger Bò', price: 5.0, image: '🍔' },
  { id: 2, name: 'Khoai tây chiên', price: 2.5, image: '🍟' },
  { id: 3, name: 'Nước ngọt', price: 1.5, image: '🥤' },
  { id: 4, name: 'Combo Hủy Diệt', price: 8.0, image: '🍱' },
]);

// 3. Giỏ hàng
const cart = ref<{ [key: number]: number }>({}); // Lưu { id: quantity }

// Tính tổng tiền
const totalPrice = computed(() => {
  let total = 0;
  for (const [id, qty] of Object.entries(cart.value)) {
    const product = products.value.find(p => p.id === Number(id));
    if (product) total += product.price * qty;
  }
  return total;
});

// 4. Hàm giao tiếp với Android (Bridge)
const callAndroid = (eventType: string, eventData: object) => {
  try {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const proxy = (window as any).TelegramWebviewProxy;
    if (proxy) {
      proxy.postEvent(eventType, JSON.stringify(eventData));
    } else {
      console.log("Web mode:", eventType, eventData);
    }
  } catch (e) { console.error(e); }
};

// 5. Logic Thêm/Bớt món
// --- SỬA LỖI TẠI ĐÂY: Thay 'any' bằng 'Product' ---
const updateCart = (product: Product, change: number) => {
  const currentQty = cart.value[product.id] || 0;
  const newQty = currentQty + change;

  // Gọi Android rung nhẹ 1 cái cho sướng tay
  callAndroid('web_app_trigger_haptic_feedback', { type: 'impact', impact_style: 'light' });

  if (newQty <= 0) {
    delete cart.value[product.id];
  } else {
    cart.value[product.id] = newQty;
  }
};

// 6. Theo dõi tổng tiền để cập nhật Main Button (Native)
watch(totalPrice, (newPrice) => {
  if (newPrice > 0) {
    // Có tiền -> Hiện nút Main Button
    callAndroid('web_app_setup_main_button', {
      is_visible: true,
      text: `THANH TOÁN ($${newPrice.toFixed(2)})`,
      color: '#31b545' // Màu xanh lá
    });
  } else {
    // Hết tiền -> Ẩn nút
    callAndroid('web_app_setup_main_button', { is_visible: false });
  }
});

onMounted(() => {
  // Mới vào thì ẩn nút đi
  callAndroid('web_app_setup_main_button', { is_visible: false });
});

// Xử lý sự kiện Android gửi về (khi bấm Main Button)
// eslint-disable-next-line @typescript-eslint/no-explicit-any
(window as any).Telegram = {
    WebView: {
        receiveEvent: (eventType: string) => {
            if (eventType === 'main_button_pressed') {
                alert("Đơn hàng đã được gửi! Tổng: $" + totalPrice.value);
            }
        }
    }
};
</script>

<template>
  <AppPage title="Durger King Fake" :back="false">
    <div class="menu-container">

      <div class="product-item" v-for="p in products" :key="p.id">
        <div class="icon">{{ p.image }}</div>
        <div class="info">
          <h3>{{ p.name }}</h3>
          <p>${{ p.price.toFixed(2) }}</p>
        </div>

        <div class="controls" v-if="cart[p.id]">
          <button class="btn-circle remove" @click="updateCart(p, -1)">-</button>
          <span>{{ cart[p.id] }}</span>
          <button class="btn-circle add" @click="updateCart(p, 1)">+</button>
        </div>
        <button class="btn-add" v-else @click="updateCart(p, 1)">ADD</button>
      </div>

    </div>
    <div style="height: 80px;"></div>
  </AppPage>
</template>

<style scoped>
.menu-container { padding: 15px; }
.product-item {
  display: flex; align-items: center;
  background: white; border-radius: 12px;
  padding: 10px; margin-bottom: 10px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.05);
}
.icon { font-size: 40px; margin-right: 15px; }
.info { flex: 1; }
.info h3 { margin: 0; font-size: 16px; color: #333; }
.info p { margin: 5px 0 0; color: #888; }

.btn-add {
  background: #f0f0f0; border: none; padding: 8px 20px;
  border-radius: 6px; color: #2481cc; font-weight: bold; cursor: pointer;
}
.controls {
  display: flex; align-items: center; gap: 10px;
}
.btn-circle {
  width: 30px; height: 30px; border-radius: 50%; border: none;
  font-weight: bold; cursor: pointer; display: flex; align-items: center; justify-content: center;
}
.add { background: #31b545; color: white; }
.remove { background: #e64d44; color: white; }
</style>
