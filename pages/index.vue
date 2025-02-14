<template>
  <div class="container">
    <div class="content">
      <h1 class="title">
        🚀 Welcome to <span class="highlight">Our Website</span>
      </h1>
      <p class="subtitle">Manage your products, orders, and users easily.</p>

      <button @click="goToLogin" class="login-btn">
        🔑 Login
      </button>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const goToLogin = () => {
  router.push('/login')
}
</script>

<style scoped>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  min-width: 100vw;
  overflow: hidden;
  background: linear-gradient(135deg, #3b82f6, #6b21a8);
  text-align: center;
  color: white;
  font-family: 'Poppins', sans-serif;
}

.content {
  max-width: 600px;
  animation: fadeIn 1s ease-in-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-20px); }
  to { opacity: 1; transform: translateY(0); }
}

.title {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 10px;
}

.highlight {
  color: #facc15;
}

.subtitle {
  font-size: 1.2rem;
  opacity: 0.9;
  margin-bottom: 20px;
}

.login-btn {
  background: #2563eb;
  color: white;
  font-size: 1.2rem;
  font-weight: bold;
  padding: 12px 24px;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

.login-btn:hover {
  background: #1e40af;
  transform: scale(1.05);
}
</style>