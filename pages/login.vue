<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-indigo-500 to-blue-600">
    <div class="bg-white shadow-lg rounded-lg p-8 w-full max-w-md transition-transform hover:scale-105">
      <h2 class="text-2xl font-bold text-center text-gray-800 mb-6">🔑 Welcome Back!</h2>

      <div v-if="errorMessage" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative mb-4">
        <strong class="font-bold">Oops! </strong>
        <span class="block sm:inline">{{ errorMessage }}</span>
      </div>

      <form @submit.prevent="login" class="space-y-4">
        <div class="relative">
          <input v-model="username" type="text" placeholder="Username" class="input-field" required />
          <span class="icon">👤</span>
        </div>

        <div class="relative">
          <input v-model="password" type="password" placeholder="Password" class="input-field" required />
          <span class="icon">🔒</span>
        </div>

        <button type="submit" class="btn-primary">🚀 Login</button>
      </form>

      <p class="text-center text-gray-500 mt-4">
        Don't have an account?
        <a href="/register" class="text-indigo-600 hover:underline">Register here</a>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

import axios from 'axios';

const API_URL = import.meta.env.VITE_BACKEND_URL;
const router = useRouter()
const username = ref('')
const password = ref('')
const errorMessage = ref('')

const login = async () => {
  errorMessage.value = '';

  try {
    console.log("🟢 Attempting login with:", username.value, password.value);

    const response = await axios.post(
        `${API_URL}/login`,
        { username: username.value, password: password.value },
        {
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "ngrok-skip-browser-warning": "true",
            "Origin": window.location.origin,
          },
        }
    );

    console.log(" Raw Response:", response.data);

    if (response.data.token) {
      console.log(" Token received:", response.data.token);

      localStorage.setItem('token', response.data.token);
      localStorage.setItem('username', response.data.username);
      localStorage.setItem('user_id', response.data.id);
      localStorage.setItem('email', response.data.email);
      localStorage.setItem('role', response.data.role);

      if (response.data.role === 'admin') {
        await router.push('/admin/dashboard');
      } else {
       await router.push('/dashbord');
      }
    } else {
      console.warn("⚠️ Login failed. Response:", response.data);
      errorMessage.value = response.data.error || 'Invalid username or password!';
    }
  } catch (error) {
    console.error(" Error during login:", error.response?.data || error);
    errorMessage.value = error.response?.data?.error || 'Something went wrong. Please try again!';
  }
};
</script>

<style scoped>
.input-field {
  width: 100%;
  padding: 12px;
  padding-left: 40px;
  border: 2px solid #ddd;
  border-radius: 8px;
  outline: none;
  transition: all 0.3s ease-in-out;
  position: relative;
}

.input-field:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 8px rgba(79, 70, 229, 0.4);
  transform: scale(1.05);
}

.icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 18px;
  color: #4f46e5;
}

.btn-primary {
  width: 100%;
  background: linear-gradient(90deg, #4f46e5, #4338ca);
  color: white;
  padding: 12px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
  text-transform: uppercase;
}

.btn-primary:hover {
  background: linear-gradient(90deg, #4338ca, #3730a3);
  transform: scale(1.05);
}
</style>
