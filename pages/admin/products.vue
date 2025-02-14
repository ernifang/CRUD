<template>
  <div class="flex">
    <Sidebar />
    <div class="ml-64 p-6 w-full">
      <h1 class="text-2xl font-bold mb-4">🛒 Product Management</h1>

      <div class="mb-4 flex items-center">
        <input
            v-model="searchQuery"
            type="text"
            placeholder="🔍 Search products..."
            class="w-full p-3 border rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
        />
      </div>

      <button @click="openAddModal" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-md mb-4">
        ➕ Add Product
      </button>

      <div class="bg-white shadow-lg rounded-lg overflow-hidden">
        <table class="min-w-full border-collapse">
          <thead>
          <tr class="bg-indigo-600 text-white">
            <th class="px-6 py-3 text-left">Product ID</th>
            <th class="px-6 py-3 text-left">Name</th>
            <th class="px-6 py-3 text-left">Price</th>
            <th class="px-6 py-3 text-left">Stock</th>
            <th class="px-6 py-3 text-left">Actions</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="product in filteredProducts" :key="product.id" class="border-b hover:bg-gray-100 transition">
            <td class="px-6 py-3">{{ product.id }}</td>
            <td class="px-6 py-3">{{ product.name }}</td>
            <td class="px-6 py-3 font-bold">${{ product.price }}</td>
            <td class="px-6 py-3">{{ product.stock }}</td>
            <td class="px-6 py-3 space-x-2 flex">
              <button @click="openEditModal(product)" class="btn-edit">
                ✏️ Edit
              </button>
              <button @click="confirmDelete(product)" class="btn-delete">
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

      <div v-if="showModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">{{ isEditing ? "✏️ Edit Product" : "➕ Add Product" }}</h2>

          <label class="block text-sm font-medium text-gray-700">Product Name:</label>
          <input v-model="productData.name" type="text" class="w-full p-2 border rounded-md mb-2" />

          <label class="block text-sm font-medium text-gray-700">Price:</label>
          <input v-model="productData.price" type="number" class="w-full p-2 border rounded-md mb-2" />

          <label class="block text-sm font-medium text-gray-700">Stock:</label>
          <input v-model="productData.stock" type="number" class="w-full p-2 border rounded-md mb-2" />

          <div class="mt-4 flex justify-between">
            <button @click="closeModal" class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400">Cancel</button>
            <button @click="isEditing ? updateProduct() : addProduct()" class="px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600">
              {{ isEditing ? "Save Changes" : "Add Product" }}
            </button>
          </div>
        </div>
      </div>

      <div v-if="showDeleteModal" class="modal-overlay">
        <div class="modal-content">
          <h2 class="text-lg font-bold text-gray-800 mb-4">⚠️ Confirm Deletion</h2>
          <p>
            Are you sure you want to delete <strong>{{ selectedProduct?.name }}</strong>?
          </p>
          <div class="mt-4 flex justify-between">
            <button @click="closeDeleteModal" class="px-4 py-2 bg-gray-300 rounded-lg hover:bg-gray-400">Cancel</button>
            <button @click="deleteProduct" class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-gray-600">
              Confirm Delete
            </button>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script setup>
import {computed, onMounted, ref} from 'vue'
import Sidebar from '~/components/Sidebar.vue'
import axios from 'axios';

const API_URL = import.meta.env.VITE_BACKEND_URL;

const products = ref([])
const showModal = ref(false)
const showDeleteModal = ref(false)
const isEditing = ref(false)
const selectedProduct = ref(null)
const searchQuery = ref('')
const isLoading = ref(true)

const productData = ref({
  name: '',
  price: '',
  stock: ''
})

const filteredProducts = computed(() => {
  return products.value.filter(product =>
      product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})
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
    console.log(" Products Loaded:", response.data);
    products.value = response.data;
  } catch (error) {
    console.error(" Fetch Error:", error);
  } finally {
    isLoading.value = false;
  }
};


const openAddModal = () => {
  isEditing.value = false
  productData.value = {name: '', price: '', stock: ''}
  showModal.value = true
}

const openEditModal = (product) => {
  isEditing.value = true
  selectedProduct.value = product
  productData.value = {...product}
  showModal.value = true
}

const closeModal = () => {
  showModal.value = false
}


const closeDeleteModal = () => {
  showDeleteModal.value = false
}

const confirmDelete = (product) => {
  selectedProduct.value = product
  showDeleteModal.value = true
}

const deleteProduct = async () => {
  if (!selectedProduct.value) return;
  try {
    await axios.delete(`${API_URL}/products/${selectedProduct.value.id}`);
    products.value = products.value.filter(product => product.id !== selectedProduct.value.id);
    closeDeleteModal();
  } catch (error) {
    console.error(" Error deleting product:", error);
  }
};

const addProduct = async () => {
  try {
    await axios.post(`${API_URL}/products`, productData.value);
    closeModal();
    await fetchProducts();
  } catch (error) {
    console.error("Error adding product:", error);
  }
};

const updateProduct = async () => {
  if (!selectedProduct.value) return;
  try {
    await axios.put(`${API_URL}/products/${selectedProduct.value.id}`, productData.value);
    closeModal();
    await fetchProducts();
  } catch (error) {
    console.error(" Error updating product:", error);
  }
};

onMounted(fetchProducts)
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
