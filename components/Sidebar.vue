<template>
  <aside class="sidebar">
    <h2 class="text-2xl font-bold text-center mb-6 animate-fadeIn">🚀 My Dashboard</h2>

    <ul class="space-y-4">
      <li v-for="item in filteredMenu" :key="item.name">
        <NuxtLink
            :to="item.link"
            class="sidebar-link"
        >
          <span class="mr-3 text-xl">{{ item.icon }}</span>
          {{ item.name }}
        </NuxtLink>
      </li>
    </ul>

    <button @click="logout" class="logout-button">🔓 Logout</button>
  </aside>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const role = ref('user')

const menuAdmin = [
  { name: 'Dashboard', link: '/admin/dashboard', icon: '📊' },
  { name: 'Users', link: '/admin/users', icon: '👥' },
  { name: 'Orders', link: '/admin/orders', icon: '📦' },
  { name: 'Product Management', link: '/admin/products', icon: '🛒' },
]

const menuUser = [
  { name: 'Dashboard', link: '/dashbord', icon: '📊' },
  { name: 'Orders', link: '/orders', icon: '📦' },
  { name: 'Shop Products', link: '/user-products', icon: '🛍' },
]

const filteredMenu = computed(() => (role.value === 'admin' ? menuAdmin : menuUser))

const getUserRole = () => {
  const storedRole = localStorage.getItem('role')
  if (storedRole) {
    role.value = storedRole
  }
}

const logout = () => {
  localStorage.removeItem('token')
  localStorage.removeItem('role')
  localStorage.removeItem('username')
  localStorage.removeItem('user_id')

  router.push('/login')
}

onMounted(getUserRole)
</script>

<style scoped>
.sidebar {
  width: 250px;
  min-height: 100vh;
  background: linear-gradient(135deg, #4f46e5, #3b82f6);
  color: white;
  padding: 20px;
  position: fixed;
  transform: translateX(-100%);
  animation: slideIn 0.5s ease forwards;
}

@keyframes slideIn {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}

.sidebar-link {
  display: flex;
  align-items: center;
  padding: 12px;
  border-radius: 8px;
  transition: all 0.3s ease-in-out;
  text-decoration: none;
  color: white;
}

.sidebar-link:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: scale(1.05);
}

.logout-button {
  margin-top: 20px;
  width: 100%;
  text-align: left;
  font-weight: bold;
  background: none;
  border: none;
  cursor: pointer;
  color: white;
  padding: 10px;
}

.logout-button:hover {
  text-decoration: underline;
  transform: scale(1.1);
}
</style>
