<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue';
import AppPage from '@/components/AppPage.vue';

// 1. ĐỊNH NGHĨA DANH SÁCH MÓN ĂN (DATA GIẢ LẬP)
interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

const products = ref<Product[]>([
  { id: 1, name: 'Burger Bò', price: 5.0, image: '🍔' },
  { id: 2, name: 'Khoai tây chiên', price: 2.5, image: '🍟' },
  { id: 3, name: 'Nước ngọt', price: 1.5, image: '🥤' },
  { id: 4, name: 'Combo Hủy Diệt', price: 8.0, image: '🍱' },
  { id: 5, name: 'Pizza', price: 12.0, image: '🍕' },
  { id: 6, name: 'Hotdog', price: 3.5, image: '🌭' },
]);

// 2. QUẢN LÝ GIỎ HÀNG
const cart = ref<{ [key: number]: number }>({}); // Lưu { id: số_lượng }

// Tính tổng tiền
const totalPrice = computed(() => {
  let total = 0;
  for (const [id, qty] of Object.entries(cart.value)) {
    const product = products.value.find(p => p.id === Number(id));
    if (product) total += product.price * qty;
  }
  return total;
});

// 3. HÀM TƯƠNG TÁC THÔNG MINH (BRIDGE)
// Hàm này tự động chọn cách nói chuyện tùy vào việc App đang chạy ở đâu
const telegramInterface = {
  // Hàm hiển thị/ẩn nút Main Button (Nút to ở đáy)
  updateMainButton: (isVisible: boolean, text: string) => {
    // CÁCH 1: Chạy trên Telegram Thật (Ưu tiên)
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    if ((window as any).Telegram?.WebApp?.MainButton) {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      const mainBtn = (window as any).Telegram.WebApp.MainButton;
      if (isVisible) {
        mainBtn.setText(text);
        mainBtn.show();
        mainBtn.enable();
      } else {
        mainBtn.hide();
      }
    }
    // CÁCH 2: Chạy trên App Android Tự Chế (Dự phòng)
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    else if ((window as any).TelegramWebviewProxy) {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      (window as any).TelegramWebviewProxy.postEvent('web_app_setup_main_button', JSON.stringify({
        is_visible: isVisible,
        text: text,
        color: '#31b545'
      }));
    }
  },

  // Hàm rung phản hồi (Haptic Feedback)
  hapticFeedback: () => {
    // Telegram Thật
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    if ((window as any).Telegram?.WebApp?.HapticFeedback) {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      (window as any).Telegram.WebApp.HapticFeedback.impactOccurred('light');
    }
    // Android Tự Chế
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    else if ((window as any).TelegramWebviewProxy) {
        // eslint-disable-next-line @typescript-eslint/no-explicit-any
      (window as any).TelegramWebviewProxy.postEvent('web_app_trigger_haptic_feedback', JSON.stringify({
        type: 'impact',
        impact_style: 'light'
      }));
    }
  },

  // Hàm gửi dữ liệu khi thanh toán
  sendData: (data: object) => {
    const jsonString = JSON.stringify(data);

    // Telegram Thật
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    if ((window as any).Telegram?.WebApp) {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      (window as any).Telegram.WebApp.sendData(jsonString);
    }
    // Android Tự Chế
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    else if ((window as any).TelegramWebviewProxy) {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      (window as any).TelegramWebviewProxy.postEvent('send_data_back_to_android', jsonString);
    }
    else {
      alert("Đang test trên Chrome: " + jsonString);
    }
  }
};

// 4. LOGIC THÊM BỚT MÓN
const updateCart = (product: Product, change: number) => {
  const currentQty = cart.value[product.id] || 0;
  const newQty = currentQty + change;

  // Rung nhẹ cái cho sướng tay
  telegramInterface.hapticFeedback();

  if (newQty <= 0) {
    delete cart.value[product.id];
  } else {
    cart.value[product.id] = newQty;
  }
};

// 5. THEO DÕI TỔNG TIỀN ĐỂ HIỆN NÚT THANH TOÁN
watch(totalPrice, (newPrice) => {
  if (newPrice > 0) {
    telegramInterface.updateMainButton(true, `THANH TOÁN ($${newPrice.toFixed(2)})`);
  } else {
    telegramInterface.updateMainButton(false, '');
  }
});

// 6. XỬ LÝ KHI BẤM NÚT MAIN BUTTON
const handleMainButtonClick = () => {
  telegramInterface.sendData({
    cart: cart.value,
    total: totalPrice.value,
    message: "Đã chốt đơn!"
  });
};

onMounted(() => {
  // Ẩn nút lúc đầu
  telegramInterface.updateMainButton(false, '');

  // Đăng ký sự kiện click cho Telegram Thật
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  if ((window as any).Telegram?.WebApp) {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    (window as any).Telegram.WebApp.ready(); // Báo cáo đã sẵn sàng
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    (window as any).Telegram.WebApp.onEvent('mainButtonClicked', handleMainButtonClick);
  }
});
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
          <span class="qty">{{ cart[p.id] }}</span>
          <button class="btn-circle add" @click="updateCart(p, 1)">+</button>
        </div>
        <button class="btn-add" v-else @click="updateCart(p, 1)">ADD</button>
      </div>

      <div style="height: 100px;"></div>

    </div>
  </AppPage>
</template>

<style scoped>
.menu-container {
  padding: 15px;
  background-color: var(--tg-theme-bg-color, #fff); /* Tự động theo theme Telegram */
  color: var(--tg-theme-text-color, #000);
}

.product-item {
  display: flex;
  align-items: center;
  background: var(--tg-theme-secondary-bg-color, #f5f5f5);
  border-radius: 15px;
  padding: 10px;
  margin-bottom: 15px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.05);
}

.icon {
  font-size: 40px;
  margin-right: 15px;
}

.info {
  flex: 1;
}
.info h3 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
}
.info p {
  margin: 5px 0 0;
  color: var(--tg-theme-hint-color, #888);
}

.btn-add {
  background: var(--tg-theme-button-color, #31b545);
  color: var(--tg-theme-button-text-color, #fff);
  border: none;
  padding: 8px 25px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
}

.controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.qty {
  font-weight: bold;
  min-width: 20px;
  text-align: center;
}

.btn-circle {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: none;
  font-weight: bold;
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}

.add {
  background: #31b545;
}

.remove {
  background: #e64d44;
}
</style>
