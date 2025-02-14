<template>
  <div class="flex">
    <Sidebar />
    <div class="ml-64 p-6 w-full">
      <h1 class="text-xl font-bold">👥 User Management</h1>

      <div class="mb-4 flex items-center">
        <input
            v-model="searchQuery"
            type="text"
            placeholder="🔍 Search users..."
            class="w-full p-3 border rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
        />
      </div>

      <div class="bg-white shadow-lg rounded-lg overflow-hidden">
        <table class="min-w-full border-collapse">
          <thead>
          <tr class="bg-indigo-600 text-white">
            <th class="px-6 py-3 text-left">ID</th>
            <th class="px-6 py-3 text-left">Username</th>
            <th class="px-6 py-3 text-left">Email</th>
            <th class="px-6 py-3 text-left">Role</th>
            <th class="px-6 py-3 text-left">Actions</th>
          </tr>
          </thead>
          <tbody>
          <tr
              v-for="user in filteredUsers"
              :key="user.id"
              class="border-b hover:bg-gray-100 transition"
          >
            <td class="px-6 py-3">{{ user.id }}</td>
            <td class="px-6 py-3">{{ user.username }}</td>
            <td class="px-6 py-3">{{ user.email }}</td>
            <td class="px-6 py-3 capitalize">{{ user.role }}</td>
            <td class="px-6 py-3 space-x-2 flex">
              <button @click="openEditModal(user)" class="btn-edit">
                ✏️ Edit
              </button>
              <button @click="confirmDelete(user)" class="btn-delete">
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

      <div
          v-if="errorMessage"
          class="mt-4 bg-red-100 text-red-600 p-3 rounded-md"
      >
        {{ errorMessage }}
      </div>

      <div v-if="showModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">
            ⚠️ Confirm Deletion
          </h2>
          <p>
            Are you sure you want to delete user
            <strong>{{ selectedUser.username }}</strong>?
          </p>
          <div class="mt-4 flex justify-between">
            <button
                @click="showModal = false"
                class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400"
            >
              Cancel
            </button>
            <button
                @click="deleteUser"
                class="px-4 py-2 bg-red-500 text-black rounded-lg hover:bg-gray-400"
            >
              Confirm Delete
            </button>
          </div>
        </div>
      </div>

      <div v-if="showEditModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">
            ✏️ Edit User Role
          </h2>
          <p>
            Edit role untuk user
            <strong>{{ selectedUser.username }}</strong>:
          </p>
          <div class="mt-4">
            <input
                v-model="editedRole"
                type="text"
                placeholder="Enter new role"
                class="w-full p-3 border rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
            />
          </div>
          <div class="mt-4 flex justify-between">
            <button
                @click="closeEditModal"
                class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400"
            >
              Cancel
            </button>
            <button
                @click="updateUserRole"
                class="px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600"
            >
              Save Changes
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
import axios from "axios";
const API_URL = import.meta.env.VITE_BACKEND_URL;

const users = ref([])
const searchQuery = ref('')
const errorMessage = ref('')
const showModal = ref(false)
const showEditModal = ref(false)
const selectedUser = ref(null)
const editedRole = ref('')

const fetchUsers = async () => {
  try {
    const token = localStorage.getItem("token");
    if (!token) {
      window.location.href = "/login";
      return;
    }

    console.log("🔄 Fetching users from:", `${API_URL}/users`);

    const response = await axios.get(`${API_URL}/users`, {
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json",
        "ngrok-skip-browser-warning": "true",
        "Origin": window.location.origin,
      },
    });

    console.log("🟢 Raw Response Data:", response.data);

    if (response.data.success) {
      if (Array.isArray(response.data.data)) {
        users.value = response.data.data;
      } else if (typeof response.data.data === "object") {
        users.value = [response.data.data];
      } else {
        console.warn("⚠️ Warning: Unexpected data format, using empty array.");
        users.value = [];
      }
    } else {
      console.warn("⚠️ API response tidak sukses, reset ke array kosong.");
      users.value = [];
    }

    console.log("Users Loaded:", users.value);
  } catch (error) {
    console.error("Error fetching users:", error);
    errorMessage.value = "Failed to fetch users";
    users.value = [];
  }
};



const confirmDelete = (user) => {
  selectedUser.value = user
  showModal.value = true
}


const deleteUser = async () => {
  if (!selectedUser.value) return;

  try {
    const token = localStorage.getItem('token');
    if (!token) {
      window.location.href = '/login';
      return;
    }

    const response = await axios.delete(`${API_URL}/users/${selectedUser.value.id}`, {
      headers: { Authorization: `Bearer ${token}` },
    });

    console.log("🗑️ User Deleted:", response.data);

    users.value = users.value.filter(user => user.id !== selectedUser.value.id);
    showModal.value = false;
    selectedUser.value = null;
  } catch (error) {
    console.error("Error deleting order:", error);
  }
}

const openEditModal = (user) => {
  selectedUser.value = user
  editedRole.value = user.role
  showEditModal.value = true
}

const closeEditModal = () => {
  showEditModal.value = false
  selectedUser.value = null
  editedRole.value = ''
}

const updateUserRole = async () => {
  if (!selectedUser.value) return

  try {
    const token = localStorage.getItem('token')
    if (!token) {
      window.location.href = '/login'
      return
    }

    const response = await fetch(
        `${API_URL}/users/${selectedUser.value.id}`,
        {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
            Authorization: `Bearer ${token}`,
          },
          body: JSON.stringify({role: editedRole.value}),
        }
    )

    if (!response.ok) {
      throw new Error('Failed to update user role')
    }

    users.value = users.value.map((user) =>
        user.id === selectedUser.value.id ? {...user, role: editedRole.value} : user
    )
    closeEditModal()
  } catch (error) {
    errorMessage.value = error.message
  }
}

const filteredUsers = computed(() => {
  return users.value.filter(
      (user) =>
          user.username.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
          user.email.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

onMounted(fetchUsers)
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
