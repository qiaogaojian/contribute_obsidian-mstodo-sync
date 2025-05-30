<template>
  <div class="app">
    <header class="app-header">
      <div class="header-container">
        <div class="logo">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4" />
          </svg>
          <h1>Todo</h1>
        </div>
        
        <div class="user-section" v-if="isAuthenticated">
          <button class="logout-btn" @click="logout">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M3 3a1 1 0 00-1 1v12a1 1 0 001 1h12a1 1 0 001-1V4a1 1 0 00-1-1H3zm9 4a1 1 0 11-2 0V5a1 1 0 112 0v2zm0 4a1 1 0 11-2 0 1 1 0 012 0zm-3 4a1 1 0 10-2 0 1 1 0 002 0z" clip-rule="evenodd" />
            </svg>
            登出
          </button>
        </div>
      </div>
    </header>
    
    <main class="app-main">
      <component :is="currentView"></component>
    </main>
    
    <footer class="app-footer">
      <div class="footer-container">
        <p>© {{ new Date().getFullYear() }} Microsoft Todo API 示例应用</p>
      </div>
    </footer>
  </div>
</template>

<script>
import { ref, shallowRef, onMounted } from 'vue'
import TodoTreeMain from './components/TodoTreeMain.vue'
import Auth from './components/Auth.vue'
import todoApi from './services/api'

export default {
  name: 'App',
  components: {
    TodoTreeMain,
    Auth
  },
  setup() {
    const isAuthenticated = ref(false)
    const currentView = shallowRef(null)
    
    // 检查认证状态
    const checkAuth = async () => {
      const token = localStorage.getItem('msft_access_token')
      if (token) {
        try {
          const { authenticated } = await todoApi.auth.checkAuth()
          isAuthenticated.value = authenticated
          currentView.value = authenticated ? TodoTreeMain : Auth
        } catch (error) {
          console.error('认证检查失败:', error)
          isAuthenticated.value = false
          currentView.value = Auth
        }
      } else {
        isAuthenticated.value = false
        currentView.value = Auth
      }
    }
    
    // 登出
    const logout = () => {
      localStorage.removeItem('msft_access_token')
      isAuthenticated.value = false
      currentView.value = Auth
    }
    
    // 路由管理
    const handleRouteChange = () => {
      const path = window.location.pathname
      if (path === '/login') {
        currentView.value = Auth
      } else {
        // 对于所有其他路径，根据认证状态显示TodoTreeMain或Auth
        currentView.value = isAuthenticated.value ? TodoTreeMain : Auth
      }
    }
    
    // 初始化
    onMounted(async () => {
      await checkAuth()
      
      // 监听URL变化
      window.addEventListener('popstate', handleRouteChange)
    })
    
    return {
      isAuthenticated,
      currentView,
      logout
    }
  }
}
</script>

<style>
/* 全局样式 */
:root {
  --primary-color: #ff69b4;
  --primary-light: #ffd6e7;
  --primary-dark: #ff5ba7;
  --text-color: #333;
  --text-secondary: #666;
  --background-color: #f5f5f5;
  --card-background: white;
  --border-color: #e0e0e0;
  --shadow-color: rgba(0, 0, 0, 0.1);
  --success-color: #28a745;
  --danger-color: #dc3545;
  --warning-color: #ffc107;
  --info-color: #17a2b8;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  line-height: 1.6;
  color: var(--text-color);
  background-color: var(--background-color);
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  background-color: var(--card-background);
  box-shadow: 0 1px 3px var(--shadow-color);
  padding: 16px 0;
}

.header-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo {
  display: flex;
  align-items: center;
}

.logo svg {
  width: 24px;
  height: 24px;
  color: var(--primary-color);
  margin-right: 8px;
}

.logo h1 {
  font-size: 20px;
  font-weight: 600;
  margin: 0;
  color: var(--text-color);
}

.user-section {
  display: flex;
  align-items: center;
}

.logout-btn {
  background: none;
  border: 1px solid var(--border-color);
  border-radius: 20px;
  padding: 6px 12px;
  display: flex;
  align-items: center;
  font-size: 14px;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s ease;
}

.logout-btn:hover {
  background-color: var(--background-color);
  color: var(--text-color);
}

.logout-btn svg {
  width: 16px;
  height: 16px;
  margin-right: 6px;
}

.app-main {
  flex: 1;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  padding: 20px;
}

.app-footer {
  background-color: var(--card-background);
  border-top: 1px solid var(--border-color);
  padding: 20px 0;
  margin-top: 40px;
}

.footer-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  text-align: center;
  color: var(--text-secondary);
  font-size: 14px;
}

/* 按钮样式 */
.btn {
  padding: 8px 16px;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  border: none;
  transition: all 0.2s ease;
}

.btn-primary {
  background-color: var(--primary-color);
  color: white;
}

.btn-primary:hover {
  background-color: var(--primary-dark);
}

.btn-secondary {
  background-color: var(--background-color);
  color: var(--text-color);
}

.btn-secondary:hover {
  background-color: var(--border-color);
}

/* 渐变动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style> 