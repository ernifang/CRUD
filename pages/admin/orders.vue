<template>
  <div class="flex">
    <Sidebar />
    <div class="ml-64 p-6 w-full">
      <h1 class="text-xl font-bold">📦 Order Management</h1>

      <div class="mb-4 flex items-center">
        <input
            v-model="searchQuery"
            type="text"
            placeholder="🔍 Search orders..."
            class="w-full p-3 border rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
        />
      </div>

      <div v-if="isLoading" class="flex justify-center items-center mt-10">
        <div class="loader"></div>
      </div>

      <div v-else class="bg-white shadow-lg rounded-lg overflow-hidden">
        <table class="min-w-full border-collapse">
          <thead>
          <tr class="bg-indigo-600 text-white">
            <th class="px-6 py-3 text-left">Order ID</th>
            <th class="px-6 py-3 text-left">Customer</th>
            <th class="px-6 py-3 text-left">Total</th>
            <th class="px-6 py-3 text-left">Status</th>
            <th class="px-6 py-3 text-left">Actions</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="order in filteredOrders" :key="order.id" class="border-b hover:bg-gray-100 transition">
            <td class="px-6 py-3">{{ order.id }}</td>
            <td class="px-6 py-3">{{ order.customer }}</td>
            <td class="px-6 py-3 font-bold">${{ order.total }}</td>
            <td class="px-6 py-3">
                <span class="status-label" :class="getStatusClass(order.status)">
                  {{ order.status }}
                </span>
            </td>
            <td class="px-6 py-3 space-x-2 flex">
              <button @click="openEditModal(order)" class="btn-edit">
                ✏️ Edit
              </button>
              <button @click="confirmDelete(order)" class="btn-delete">
                <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="white">
                  <path d="M6 19a2 2 0 002 2h8a2 2 0 002-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z"/>
                </svg>
                Delete
              </button>
            </td>
          </tr>
          </tbody>
        </table>
      </div>

      <div v-if="showEditModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">✏️ Edit Order</h2>
          <p>Update status order <strong>#{{ selectedOrder.id }}</strong></p>

          <div class="mt-4">
            <label class="block text-sm font-medium text-gray-700">Order Status:</label>
            <select v-model="editedStatus" class="w-full p-3 border rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
              <option value="Pending">Pending</option>
              <option value="Shipped">Shipped</option>
              <option value="Completed">Completed</option>
              <option value="Canceled">Canceled</option>
            </select>
          </div>

          <div class="mt-4 flex justify-between">
            <button @click="closeEditModal" class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400">Cancel</button>
            <button @click="updateOrder" class="px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600">
              Save Changes
            </button>
          </div>
        </div>
      </div>

      <div v-if="showModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">⚠️ Confirm Deletion</h2>
          <p>
            Are you sure you want to delete order <strong>#{{ selectedOrder.id }}</strong>?
          </p>
          <div class="mt-4 flex justify-between">
            <button @click="showModal = false" class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400">Cancel</button>
            <button @click="deleteOrder" class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-gray-600">
              Confirm Delete
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import Sidebar from '~/components/Sidebar.vue'
import axios from 'axios';

const API_URL = import.meta.env.VITE_BACKEND_URL;

const orders = ref([])
const searchQuery = ref('')
const isLoading = ref(true)
const showModal = ref(false)
const selectedOrder = ref(null)
const editedStatus = ref('')
const showEditModal = ref(false)

const openEditModal = (order) => {
  selectedOrder.value = order
  editedStatus.value = order.status
  showEditModal.value = true
}
const closeEditModal = () => {
  showEditModal.value = false
  selectedOrder.value = null
}
const updateOrder = async () => {
  if (!selectedOrder.value) return;

  try {
    const token = localStorage.getItem('token');
    if (!token) {
      window.location.href = '/login';
      return;
    }

    const response = await axios.put(`${API_URL}/orders/${selectedOrder.value.id}`,
        { status: editedStatus.value },
        { headers: { Authorization: `Bearer ${token}` } }
    );

    console.log(" Order Updated:", response.data);

    orders.value = orders.value.map(order =>
        order.id === selectedOrder.value.id ? { ...order, status: editedStatus.value } : order
    );

    closeEditModal();
  } catch (error) {
    console.error("Error updating order:", error);
  }
};


const fetchOrders = async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      window.location.href = "/login";
      return;
    }

    console.log("🔄 Fetching orders from:", `${API_URL}/orders`);

    const response = await axios.get(
        `${API_URL}/orders`,
        {
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "ngrok-skip-browser-warning": "true",
            "Origin": window.location.origin,
          },
        }
    );

    console.log(" Raw Response Data:", response.data);

    if (response.data.success) {
      if (Array.isArray(response.data.data)) {
        orders.value = response.data.data;
      } else if (typeof response.data.data === "object") {
        orders.value = [response.data.data];
      } else {
        console.warn(" Warning: Unexpected data format, using empty array.");
        orders.value = [];
      }
    } else {
      console.warn(" API response tidak sukses, reset ke array kosong.");
      orders.value = [];
    }

    console.log("Orders Loaded:", orders.value);
  } catch (error) {
    console.error("Error fetching Orders:", error);
    orders.value = [];
  } finally {
    isLoading.value = false;
  }
};







const confirmDelete = (order) => {
  selectedOrder.value = order
  showModal.value = true
}

const deleteOrder = async () => {
  if (!selectedOrder.value) return;

  try {
    const token = localStorage.getItem('token');
    if (!token) {
      window.location.href = '/login';
      return;
    }

    const response = await axios.delete(`${API_URL}/orders/${selectedOrder.value.id}`, {
      headers: { Authorization: `Bearer ${token}` },
    });

    console.log("🗑️ Order Deleted:", response.data);

    orders.value = orders.value.filter(order => order.id !== selectedOrder.value.id);
    showModal.value = false;
    selectedOrder.value = null;
  } catch (error) {
    console.error(" Error deleting order:", error);
  }
};


const filteredOrders = computed(() => {
  if (!Array.isArray(orders.value)) {
    console.warn("⚠️ Warning: orders.value bukan array, reset ke array kosong.");
    return [];
  }

  return orders.value.filter(order =>
      order.customer.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      order.id.toString().includes(searchQuery.value) ||
      order.status.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});




const getStatusClass = (status) => {
  return {
    'bg-yellow-400 text-black': status === 'Pending',
    'bg-blue-500 text-white': status === 'Shipped',
    'bg-green-500 text-white': status === 'Completed',
    'bg-red-500 text-white': status === 'Canceled',
  }
}

onMounted(fetchOrders)
</script>

<style scoped>
table {
  width: 100%;
  border-spacing: 0;
}

th,
td {
  text-align: left;
  padding: 12px;
}

th {
  background-color: #4f46e5;
  color: white;
}

td {
  border-bottom: 1px solid #ddd;
}

button {
  cursor: pointer;
}

.status-label {
  padding: 6px 12px;
  border-radius: 8px;
  font-weight: bold;
  display: inline-block;
}

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

.loader {
  border: 5px solid #f3f3f3;
  border-top: 5px solid #4f46e5;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

.btn-edit, .btn-delete {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 8px 14px;
  font-weight: bold;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
  font-size: 14px;
}

.btn-edit {
  background-color: #3b82f6;
  color: white;
}

.btn-edit:hover {
  background-color: #2563eb;
}

.btn-delete {
  background-color: #ef4444;
  color: white;
}

.btn-delete:hover {
  background-color: #dc2626;
}

.icon {
  width: 16px;
  height: 16px;
  fill: white;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
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
