<template>
  <div class="min-h-screen flex bg-gray-100">
    <Sidebar />

    <div class="flex-1 p-6 ml-64">
      <nav class="bg-white p-4 rounded-lg shadow-md flex justify-between items-center">
        <h1 class="text-2xl font-bold text-gray-800">🚀 Welcome, <span class="text-indigo-600">{{ username }}</span>!</h1>
        <button @click="logout" class="px-4 py-2 bg-red-500 text-white rounded-md hover:bg-red-600 transition">
          🚪 Logout
        </button>
      </nav>

      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 mt-6">

        <div class="dashboard-card">
          <div class="icon bg-blue-500">
            📦
          </div>
          <div>
            <h2 class="text-3xl font-bold">{{ totalOrders }}</h2>
            <p class="text-gray-600">Total Orders</p>
          </div>
        </div>

        <div class="dashboard-card">
          <div class="icon bg-yellow-500">
            ⏳
          </div>
          <div>
            <h2 class="text-3xl font-bold">{{ pendingOrders }}</h2>
            <p class="text-gray-600">Pending Orders</p>
          </div>
        </div>

        <div class="dashboard-card">
          <div class="icon bg-green-500">
            ✅
          </div>
          <div>
            <h2 class="text-3xl font-bold">{{ completedOrders }}</h2>
            <p class="text-gray-600">Completed Orders</p>
          </div>
        </div>
      </div>

      <div class="bg-white p-6 rounded-lg shadow-lg mt-6">
        <h2 class="text-xl font-bold text-gray-800 mb-4">🔍 Recent Activity</h2>
        <ul v-if="recentOrders.length">
          <li v-for="order in recentOrders" :key="order.id" class="flex justify-between p-2 border-b last:border-none">
            <span>{{ order.customer }} ordered <strong>{{ order.product_name }}</strong></span>
            <span :class="statusClass(order.status)">{{ order.status }}</span>
          </li>
        </ul>
        <p v-else class="text-gray-500">No recent orders.</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Sidebar from '~/components/Sidebar.vue'
import { useRouter } from 'vue-router'
import axios from 'axios';

const API_URL = import.meta.env.VITE_BACKEND_URL;
const router = useRouter();
const username = ref('');
const totalOrders = ref(0);
const pendingOrders = ref(0);
const completedOrders = ref(0);
const recentOrders = ref([]);

const fetchUserData = () => {
  const storedUsername = localStorage.getItem('username');
  if (!storedUsername) {
    router.push('/login');
    return;
  }
  username.value = storedUsername;
};

const fetchOrders = async () => {
  try {
    const userId = localStorage.getItem('user_id');
    if (!userId) {
      console.error("User ID not found");
      return;
    }

    const response = await axios.get(`${API_URL}/orders`, { headers: getHeaders() });
    const userOrders = response.data.data.filter(order => order.user_id == userId);

    totalOrders.value = userOrders.length;
    pendingOrders.value = userOrders.filter(order => order.status.toLowerCase() === 'pending').length;
    completedOrders.value = userOrders.filter(order => order.status.toLowerCase() === 'completed').length;
    recentOrders.value = userOrders.slice(0, 5);

    console.log("🟢 Orders Loaded:", userOrders);
  } catch (error) {
    console.error("Error fetching orders:", error);
  }
};

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

const logout = () => {
  localStorage.removeItem('token');
  router.push('/login');
};

onMounted(() => {
  fetchUserData();
  fetchOrders();
});
</script>

<style scoped>
.dashboard-card {
  display: flex;
  align-items: center;
  gap: 15px;
  background: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease-in-out;
}

.dashboard-card:hover {
  transform: translateY(-5px) scale(1.05);
}

.icon {
  width: 50px;
  height: 50px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.8rem;
  color: white;
}

.text-yellow-500 {
  color: #eab308;
}

.text-green-500 {
  color: #10b981;
}

.text-red-500 {
  color: #ef4444;
}
</style>
