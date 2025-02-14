<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-500 to-indigo-700">
    <div class="bg-white shadow-lg rounded-lg p-8 w-full max-w-md transition-transform hover:scale-105">
      <h2 class="text-2xl font-bold text-center text-gray-800 mb-6">🚀 Create an Account</h2>
      <form @submit.prevent="register" class="space-y-4">

        <div class="relative">
          <input v-model="username" type="text" placeholder="Username" class="input-field" required />
          <span class="icon">👤</span>
        </div>

        <div class="relative">
          <input v-model="email" type="email" placeholder="Email" class="input-field" required />
          <span class="icon">📧</span>
        </div>

        <div class="relative">
          <input v-model="password" type="password" placeholder="Password" class="input-field" required />
          <span class="icon">🔒</span>
        </div>

        <button type="submit" class="btn-primary">🚀 Register Now</button>
      </form>

      <p class="text-center text-gray-500 mt-4">
        Already have an account?
        <a href="/login" class="text-indigo-600 hover:underline">Login here</a>
      </p>
    </div>

    <div v-if="showSuccessModal" class="modal-overlay">
      <div class="modal-content">
        <h2 class="text-lg font-bold text-gray-800 mb-4">✅ Registration Successful!</h2>
        <p>Welcome, <strong>{{ username }}</strong>! You will be redirected to the dashboard shortly.</p>
        <button @click="redirectToDashboard" class="btn-modal">Go to Dashboard</button>
      </div>
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
const email = ref('')
const password = ref('')
const showSuccessModal = ref(false)
const errorMessage = ref('')

const register = async () => {
  try {
    console.log("🟢 Attempting registration with:", username.value, email.value, password.value);

    const response = await axios.post(
        `${API_URL}/register`,
        { username: username.value, email: email.value, password: password.value, role: 'user' },
        {
          headers: {
            "Content-Type": "application/json",
            "Accept": "application/json",

          },
        }
    );

    console.log("🔵 Raw Response:", response.data);

    if (response.data.token) {
      console.log("Registration Successful. Token received:", response.data.token);


      localStorage.setItem('token', response.data.token);
      localStorage.setItem('username', response.data.username);
      localStorage.setItem('email', response.data.email);
      localStorage.setItem('user_id', response.data.id);
      localStorage.setItem('role', response.data.role);


      showSuccessModal.value = true;
      setTimeout(() => {
        router.push('/dashbord');
      }, 2000);
    } else {
      console.warn("⚠️ Registration failed. Response:", response.data);
      errorMessage.value = response.data.error || 'Registration failed!';
    }
  } catch (error) {
    console.error("Error during registration:", error.response?.data || error);
    errorMessage.value = error.response?.data?.error || 'Something went wrong. Please try again!';
  }
};

const redirectToDashboard = () => {
  showSuccessModal.value = false
  router.push('/dashbord')
}
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
  color: #6366f1;
}

.btn-primary {
  width: 100%;
  background: linear-gradient(90deg, #6366f1, #4f46e5);
  color: white;
  padding: 12px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
  text-transform: uppercase;
}

.btn-primary:hover {
  background: linear-gradient(90deg, #4f46e5, #4338ca);
  transform: scale(1.05);
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
