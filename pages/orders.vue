<template>
  <div class="flex">
    <Sidebar />
    <div class="ml-64 p-6 w-full">
      <h1 class="text-2xl font-bold mb-4">📦 My Orders</h1>

      <div v-if="isLoading" class="text-center text-gray-500">Loading orders...</div>

      <div v-else-if="orders.length" class="bg-white shadow-lg rounded-lg overflow-hidden">
        <table class="min-w-full border-collapse">
          <thead>
          <tr class="bg-indigo-600 text-white">
            <th class="px-6 py-3 text-left">Customer</th>
            <th class="px-6 py-3 text-left">Product ID</th>
            <th class="px-6 py-3 text-left">Product Name</th>
            <th class="px-6 py-3 text-left">Total</th>
            <th class="px-6 py-3 text-left">Status</th>
            <th class="px-6 py-3 text-left">Created At</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="order in orders" :key="order.id" class="border-b hover:bg-gray-100 transition">
            <td class="px-6 py-3">{{ order.customer }}</td>
            <td class="px-6 py-3">{{ order.product.id }}</td>
            <td class="px-6 py-3">{{ order.product.name }}</td>
            <td class="px-6 py-3 font-bold">${{ order.total }}</td>
            <td class="px-6 py-3">
              <span :class="statusClass(order.status)">{{ order.status }}</span>
            </td>
            <td class="px-6 py-3">{{ formatDate(order.created_at) }}</td>
          </tr>
          </tbody>

        </table>
      </div>

      <div v-else class="text-center text-gray-500">You have no orders yet.</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Sidebar from '@/components/Sidebar.vue'
import axios from 'axios'

const API_URL = import.meta.env.VITE_BACKEND_URL;
const orders = ref([]);
const isLoading = ref(true);

const getHeaders = () => {
  const token = localStorage.getItem('token');
  return {
    "Content-Type": "application/json",
    "Accept": "application/json",
    "ngrok-skip-browser-warning": "true",
    "Origin": window.location.origin,
    ...(token ? {Authorization: `Bearer ${token}`} : {}),
  };
};

const fetchOrders = async () => {
  try {
    const userId = localStorage.getItem('user_id');

    if (!userId) {
      console.error(" User ID not found");
      return;
    }

    const response = await axios.get(`${API_URL}/orders`, { headers: getHeaders() });

    console.log("🟢 API Response:", response.data);

    if (response.data.success && Array.isArray(response.data.data)) {
      orders.value = response.data.data.filter(order => order.user_id == userId);
      console.log(" Orders Loaded:", orders.value);
    } else {
      console.error(" Unexpected API response:", response.data);
    }

  } catch (error) {
    console.error(" Fetch Orders Error:", error.response?.data || error);
  } finally {
    isLoading.value = false;
  }
};


const statusClass = (status) => {
  switch (status.toLowerCase()) {
    case 'pending':
      return 'text-yellow-500 font-bold';
    case 'completed':
      return 'text-green-500 font-bold';
    case 'canceled':
      return 'text-red-500 font-bold';
    default:
      return 'text-gray-500';
  }
};

const formatDate = (dateString) => {
  const date = new Date(dateString);
  return date.toLocaleString();
};

onMounted(fetchOrders);
</script>

<style scoped>
.text-yellow-500 {
  color: #eab308;
}

.text-green-500 {
  color: #10b981;
}

.text-red-500 {
  color: #ef4444;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 12px;
  text-align: left;
}

th {
  background-color: #4f46e5;
  color: white;
}

tr:hover {
  background-color: #f3f4f6;
}

.text-gray-500 {
  color: #6b7280;
}
</style>
