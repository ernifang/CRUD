<template>
  <div class="flex">
    <Sidebar />
    <div class="ml-64 p-6 w-full">
      <h1 class="text-2xl font-bold mb-4">🛍 Available Products</h1>

      <div class="mb-4 flex items-center">
        <input
            v-model="searchQuery"
            type="text"
            placeholder="🔍 Search products..."
            class="w-full p-3 border rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
        />
      </div>

      <div class="bg-white shadow-lg rounded-lg overflow-hidden">
        <table class="min-w-full border-collapse">
          <thead>
          <tr class="bg-indigo-600 text-white">
<!--
            <th class="px-6 py-3 text-left">Product ID</th>
-->
            <th class="px-6 py-3 text-left">Name</th>
            <th class="px-6 py-3 text-left">Price</th>
            <th class="px-6 py-3 text-left">Stock</th>
            <th class="px-6 py-3 text-left">Actions</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="product in filteredProducts" :key="product.id" class="border-b hover:bg-gray-100 transition">
<!--
            <td class="px-6 py-3">{{ product.id }}</td>-->
            <td class="px-6 py-3">{{ product.name }}</td>
            <td class="px-6 py-3 font-bold">${{ product.price }}</td>
            <td class="px-6 py-3">{{ product.stock }}</td>
            <td class="px-6 py-3">
              <button @click="addToCart(product)" class="btn-add">
                🛒 Add to Cart
              </button>
            </td>
          </tr>
          </tbody>
        </table>
      </div>

      <div v-if="showOrderModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">✅ Order Placed</h2>
          <p>Your order for <strong>{{ selectedProduct?.name }}</strong> has been placed successfully.</p>
          <button @click="closeOrderModal" class="btn-modal">OK</button>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import Sidebar from '@/components/Sidebar.vue'
import axios from 'axios'

const API_URL = import.meta.env.VITE_BACKEND_URL;

const products = ref([]);
const searchQuery = ref('');
const isLoading = ref(true);
const showOrderModal = ref(false);
const selectedProduct = ref(null);

const filteredProducts = computed(() => {
  return products.value.filter(product =>
      product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
});

const getHeaders = () => {
  const token = localStorage.getItem('token');
  return {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "ngrok-skip-browser-warning": "true",
    "Origin": window.location.origin,
    ...(token ? { Authorization: `Bearer ${token}` } : {}),
  };
};

const fetchProducts = async () => {
  try {
    const response = await axios.get(`${API_URL}/products`, { headers: getHeaders() });
    console.log("🟢 Products Loaded:", response.data);
    products.value = response.data;
  } catch (error) {
    console.error("  Fetch Error:", error);
  } finally {
    isLoading.value = false;
  }
};

const addToCart = async (product) => {
  try {
    selectedProduct.value = product;

    const userId = localStorage.getItem('user_id');
    const username = localStorage.getItem('username');


    if (!userId || !username) {
      console.error("  User ID or Username not found");
      return;
    }

    const orderData = {
      user_id: parseInt(userId),
      product_id: product.id,
      customer: username,
      total: parseFloat(product.price),
      status: "Pending",
      created_at: new Date().toISOString(),
    };

    console.log("📦 Sending Order Data:", orderData);

    const response = await axios.post(`${API_URL}/orders`, orderData, { headers: getHeaders() });

    console.log("  Order Created:", response.data);

    showOrderModal.value = true;
  } catch (error) {
    console.error("  Error adding order:", error.response?.data || error);
  }
};


const closeOrderModal = () => {
  showOrderModal.value = false;
};

onMounted(fetchProducts);
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  text-align: center;
  animation: fadeIn 0.3s ease-in-out;
}

.btn-add {
  background-color: #10b981;
  color: white;
  padding: 8px 14px;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
  font-size: 14px;
}

.btn-add:hover {
  background-color: #059669;
}


.btn-modal {
  margin-top: 20px;
  background: #4f46e5;
  color: white;
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease-in-out;
}

.btn-modal:hover {
  background: #4338ca;
  transform: scale(1.05);



}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
