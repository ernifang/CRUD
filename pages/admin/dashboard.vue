<template>
  <div class="min-h-screen flex bg-gray-100">
    <Sidebar />

    <div class="flex-1 p-6 ml-64">

      <nav class="bg-white p-4 rounded-lg shadow-md flex justify-between items-center">
        <h1 class="text-2xl font-bold text-gray-800">📊 Dashboard Overview</h1>
        <div class="flex items-center space-x-4">
          <span v-if="username" class="text-gray-600">Hello, <strong>{{ username }}</strong> 👋</span>
        </div>
      </nav>

      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6 mt-6">
        <div class="dashboard-card">
          <div class="icon bg-blue-500">👤</div>
          <div>
            <p class="text-gray-500">Total Users</p>
            <h2 class="text-2xl font-bold">{{ totalUsers }}</h2>
          </div>
        </div>
        <div class="dashboard-card">
          <div class="icon bg-green-500">💰</div>
          <div>
            <p class="text-gray-500">Total Revenue</p>
            <h2 class="text-2xl font-bold">${{ totalRevenue }}</h2>
          </div>
        </div>
        <div class="dashboard-card">
          <div class="icon bg-purple-500">🛍</div>
          <div>
            <p class="text-gray-500">Products Sold</p>
            <h2 class="text-2xl font-bold">{{ totalProductsSold }}</h2>
          </div>
        </div>
      </div>

      <div class="bg-white p-6 rounded-lg shadow-lg mt-6">
        <h2 class="text-xl font-bold text-gray-800 mb-4">📜 Recent Activity</h2>
        <ul v-if="activities.length">
          <li v-for="(activity, index) in activities" :key="index" class="activity-item">
            <span class="text-gray-600 text-sm">{{ activity.time }}</span>
            <span class="ml-4 font-medium">{{ activity.message }}</span>
          </li>
        </ul>
        <p v-else class="text-gray-500">No recent activity.</p>
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
const totalUsers = ref(0);
const totalRevenue = ref(0);
const totalProductsSold = ref(0);
const activities = ref([]);

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


const fetchUserData = () => {
  const storedUsername = localStorage.getItem('username');
  if (!storedUsername) {
    router.push('/login');
    return;
  }
  username.value = storedUsername;
};


const fetchUsers = async () => {
  try {
    const response = await axios.get(`${API_URL}/users`, { headers: getHeaders() });
    totalUsers.value = response.data.data.length;
    console.log("Total Users Loaded:", totalUsers.value);
  } catch (error) {
    console.error("Error fetching users:", error);
  }
};

const fetchRevenueAndSales = async () => {
  try {
    const response = await axios.get(`${API_URL}/orders`, {headers: getHeaders()});

    totalRevenue.value = response.data.data.reduce((sum, order) => sum + order.total, 0);
    totalProductsSold.value = response.data.data.length;

    console.log("Revenue & Sales Loaded:", {revenue: totalRevenue.value, sales: totalProductsSold.value});
  } catch (error) {
    console.error("Error fetching orders:", error);
  }
};

const fetchRecentActivities = async () => {
  try {
    const response = await axios.get(`${API_URL}/orders`, {headers: getHeaders()});

    activities.value = response.data.data
        .slice(-5)
        .map(order => ({
          time: new Date(order.created_at).toLocaleString(),
          message: `New order by ${order.customer}`
        }));

    console.log("Recent Activities Loaded:", activities.value);
  } catch (error) {
    console.error("Error fetching activities:", error);
  }
};

onMounted(() => {
  fetchUserData();
  fetchUsers();
  fetchRevenueAndSales();
  fetchRecentActivities();
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
  position: relative;
  overflow: hidden;
}

.dashboard-card:hover {
  transform: translateY(-5px) scale(1.05);
  box-shadow: 0px 6px 15px rgba(0, 0, 0, 0.2);
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

.bg-blue-500 {
  background-color: #3b82f6;
}

.bg-green-500 {
  background-color: #10b981;
}

.bg-purple-500 {
  background-color: #8b5cf6;
}

.activity-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #e5e7eb;
}

.activity-item:last-child {
  border-bottom: none;
}
</style>
