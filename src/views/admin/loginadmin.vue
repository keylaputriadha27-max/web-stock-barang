<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import api from "../../services/api"

// 1. Inisialisasi Router & State Form
const router = useRouter()
const username = ref('')
const password = ref('')
const errorMessage = ref('')

// 2. Fungsi Login
const handleLogin = async () => {
  errorMessage.value = '' // Reset pesan error
  
  try {
    const res = await api.post('/login', {
      username: username.value,
      password: password.value
    })
    
    // Simpan token JWT ke browser partner
    localStorage.setItem('token', res.data.token)
    
    // Redirect ke halaman dashboard admin
    router.push('/dashboardadmin') // Sesuaikan path ini dengan router kamu
  } catch (err) {
    console.error('Gagal terhubung ke laptop backend:', err)
    
    // Tangkap pesan error dari Go backend
    if (err.response) {
      errorMessage.value = err.response.data || 'Username atau password salah!'
    } else {
      errorMessage.value = 'Tidak dapat terhubung ke server backend!'
    }
  }
}
</script>

<template>
  <div class="login-page">
    <div class="login-card">

      <div class="login-header">
        <div class="logo-box">
          📦
        </div>

        <h1>Selamat Datang!</h1>
        <p>Silakan masuk untuk mengelola stok barang</p>
      </div>

      <!-- DIPERBAIKI: Memanggil handleLogin, bukan login -->
      <form @submit.prevent="handleLogin">

        <div class="form-group">
          <label>Username</label>

          <input
            v-model="username"
            type="text"
            placeholder="Masukkan username"
            required
          />
        </div>

        <div class="form-group">
          <label>Password</label>

          <input
            v-model="password"
            type="password"
            placeholder="Masukkan password"
            required
          />
        </div>

        <p v-if="errorMessage" class="error">
          {{ errorMessage }}
        </p>

        <button type="submit">
          Masuk
        </button>

      </form>

      <div class="login-footer">
        <p>© 2026 Sistem Stok Barang</p>
      </div>

    </div>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.login-page {
  width: 100%;
  min-height: 100vh;
  background: #f5f7fb;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.login-card {
  width: 100%;
  max-width: 420px;
  background: white;
  padding: 40px;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
}

.login-header {
  text-align: center;
  margin-bottom: 30px;
}

.logo-box {
  width: 65px;
  height: 65px;
  margin: 0 auto 15px;
  background: #4f46e5;
  color: white;
  border-radius: 15px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 32px;
}

.login-header h1 {
  margin: 0;
  font-size: 26px;
  color: #1f2937;
}

.login-header p {
  margin-top: 8px;
  color: #6b7280;
  font-size: 14px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #374151;
}

.form-group input {
  width: 100%;
  padding: 13px 15px;
  border: 1px solid #d1d5db;
  border-radius: 10px;
  outline: none;
  font-size: 15px;
}

.form-group input:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
}

button {
  width: 100%;
  padding: 13px;
  border: none;
  border-radius: 10px;
  background: #4f46e5;
  color: white;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
}

button:hover {
  background: #4338ca;
}

.error {
  background: #fee2e2;
  color: #dc2626;
  padding: 10px;
  border-radius: 8px;
  font-size: 14px;
  text-align: center;
  margin-bottom: 15px;
}

.login-footer {
  margin-top: 25px;
  text-align: center;
  color: #9ca3af;
  font-size: 13px;
}
</style>