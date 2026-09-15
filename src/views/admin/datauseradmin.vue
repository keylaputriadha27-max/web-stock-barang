<script setup>
import { ref, computed, onMounted } from "vue"
import api from "../../services/api"
const search = ref("")
const showForm = ref(false)
const loading = ref(false)

// Form Input Fields
const username = ref("")
const password = ref("")
const role = ref("petugas") // Default role

const users = ref([])

// Fetch Data User dari Backend
const fetchUsers = async () => {
  loading.value = true
  try {
    const res = await api.get("/users")
    users.value = Array.isArray(res.data) ? res.data : (res.data.data || [])
  } catch (err) {
    console.error("Gagal mengambil data user:", err)
    alert("Gagal memuat data pengguna!")
  } finally {
    loading.value = false
  }
}

// Filter Pencarian
const filteredUsers = computed(() => {
  const keyword = search.value.toLowerCase()
  return users.value.filter((item) => {
    const usr = item.username ? item.username.toLowerCase() : ""
    const rl = item.role ? item.role.toLowerCase() : ""
    return usr.includes(keyword) || rl.includes(keyword)
  })
})

// Simpan User Baru
const tambahUser = async () => {
  if (!username.value || !password.value || !role.value) {
    alert("Username, password, dan role harus diisi!")
    return
  }

  try {
    await api.post("/register", {
      username: username.value,
      password: password.value,
      role: role.value
    })

    alert("User berhasil ditambahkan!")
    
    // Reset Form
    username.value = ""
    password.value = ""
    role.value = "petugas"
    showForm.value = false

    // Refresh Data Table
    fetchUsers()
  } catch (err) {
    console.error("Gagal menambah user:", err)
    const errorMsg = err.response?.data || "Gagal menambahkan user baru!"
    alert(errorMsg)
  }
}

// Hapus User
const hapusUser = async (id) => {
  if (!confirm("Apakah Anda yakin ingin menghapus pengguna ini?")) return

  try {
    await api.delete(`/users/${id}`)
    alert("User berhasil dihapus!")
    fetchUsers()
  } catch (err) {
    console.error("Gagal menghapus user:", err)
    alert("Gagal menghapus pengguna!")
  }
}

onMounted(() => {
  fetchUsers()
})
</script>

<template>
  <div class="page">

    <!-- SIDEBAR -->
    <aside class="sidebar">

      <div class="logo">
        <div class="logo-icon">📦</div>
        <span>StockKu</span>
      </div>

      <nav class="menu">
        <router-link to="/dashboardadmin">📊 Dashboard</router-link>
        <router-link to="/barangadmin">📦 Data Barang</router-link>
        <router-link to="/barangmasukadmin">📥 Barang Masuk</router-link>
        <router-link to="/barangkeluaradmin">📤 Barang Keluar</router-link>
        <router-link to="/useradmin" class="active">👥 Data User</router-link>
        <router-link to="/riwayatadmin">🕒 Riwayat</router-link>
        <router-link to="/pengaturan">⚙️ Pengaturan</router-link>
      </nav>

      <router-link to="/" class="logout">🚪 Logout</router-link>

    </aside>

    <!-- CONTENT -->
    <main class="content">

      <header class="header">

        <div>
          <h1>Data User</h1>
          <p>Kelola data pengguna sistem dan hak aksesnya</p>
        </div>

        <div class="profile">
          <div class="avatar">A</div>
          <div>
            <strong>Admin</strong>
            <small>Administrator</small>
          </div>
        </div>

      </header>

      <section class="card">

        <div class="card-header">

          <div>
            <h2>Daftar User</h2>
            <p>Total {{ users.length }} pengguna</p>
          </div>

          <button
            class="btn-tambah"
            @click="showForm = !showForm"
          >
            {{ showForm ? 'Batal' : '+ Tambah User' }}
          </button>

        </div>

        <!-- FORM TAMBAH USER -->
        <div
          v-if="showForm"
          class="form"
        >

          <h3>Tambah Pengguna Baru</h3>

          <div class="form-grid">

            <input
              v-model="username"
              type="text"
              placeholder="Username"
            />

            <input
              v-model="password"
              type="password"
              placeholder="Password"
            />

            <select v-model="role">
              <option value="admin">Admin</option>
              <option value="petugas">Petugas</option>
            </select>

          </div>

          <div class="form-actions">

            <button
              class="btn-simpan"
              @click="tambahUser"
            >
              Simpan
            </button>

            <button
              class="btn-batal"
              @click="showForm = false"
            >
              Batal
            </button>

          </div>

        </div>

        <!-- SEARCH -->
        <div class="search">

          <input
            v-model="search"
            type="text"
            placeholder="🔍 Cari username atau role..."
          />

        </div>

        <!-- TABLE -->
        <div class="table-container">

          <div v-if="loading" class="empty">Memuat data pengguna...</div>

          <table v-else>

            <thead>
              <tr>
                <th>No</th>
                <th>ID User</th>
                <th>Username</th>
                <th>Role</th>
                <th>Aksi</th>
              </tr>
            </thead>

            <tbody>

              <tr
                v-for="(item, index) in filteredUsers"
                :key="item.id"
              >

                <td>{{ index + 1 }}</td>

                <td>
                  <strong>#{{ item.id }}</strong>
                </td>

                <td>{{ item.username }}</td>

                <td>
                  <span
                    class="role"
                    :class="item.role ? item.role.toLowerCase() : ''"
                  >
                    {{ item.role }}
                  </span>
                </td>

                <td>
                  <button
                    class="btn-hapus"
                    @click="hapusUser(item.id)"
                  >
                    🗑️
                  </button>
                </td>

              </tr>

              <tr v-if="filteredUsers.length === 0">

                <td
                  colspan="5"
                  class="empty"
                >
                  User tidak ditemukan
                </td>

              </tr>

            </tbody>

          </table>

        </div>

      </section>

    </main>

  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.page {
  min-height: 100vh;
  display: flex;
  background: #f5f7fb;
  font-family: Arial, sans-serif;
}

.sidebar {
  width: 250px;
  min-height: 100vh;
  background: white;
  padding: 25px 15px;
  border-right: 1px solid #e5e7eb;
  display: flex;
  flex-direction: column;
}

.logo {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 21px;
  font-weight: bold;
  padding: 0 10px 30px;
}

.logo-icon {
  width: 42px;
  height: 42px;
  background: #4f46e5;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}

.menu {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.menu a {
  padding: 13px 15px;
  border-radius: 9px;
  text-decoration: none;
  color: #6b7280;
}

.menu a:hover {
  background: #f3f4f6;
}

.menu .active {
  background: #eef2ff;
  color: #4f46e5;
  font-weight: bold;
}

.logout {
  margin-top: auto;
  padding: 13px;
  text-align: center;
  text-decoration: none;
  background: #fee2e2;
  color: #dc2626;
  border-radius: 9px;
}

.content {
  flex: 1;
  padding: 35px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}

.header h1 {
  margin: 0;
  font-size: 28px;
  color: #1f2937;
}

.header p {
  color: #6b7280;
}

.profile {
  display: flex;
  align-items: center;
  gap: 10px;
}

.profile small {
  display: block;
  color: #6b7280;
  margin-top: 4px;
}

.avatar {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  background: #4f46e5;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}

.card {
  background: white;
  padding: 25px;
  border-radius: 15px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-header h2 {
  margin: 0;
}

.card-header p {
  color: #6b7280;
}

.btn-tambah {
  border: none;
  padding: 12px 18px;
  border-radius: 8px;
  background: #4f46e5;
  color: white;
  cursor: pointer;
}

.form {
  margin-top: 20px;
  padding: 20px;
  background: #f9fafb;
  border-radius: 10px;
}

.form h3 {
  margin-top: 0;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.form-grid input,
.form-grid select {
  padding: 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background: white;
}

.form-actions {
  margin-top: 15px;
  display: flex;
  gap: 10px;
}

.btn-simpan {
  border: none;
  padding: 10px 20px;
  background: #16a34a;
  color: white;
  border-radius: 7px;
  cursor: pointer;
}

.btn-batal {
  border: none;
  padding: 10px 20px;
  background: #e5e7eb;
  border-radius: 7px;
  cursor: pointer;
}

.search {
  margin: 25px 0 15px;
}

.search input {
  width: 350px;
  max-width: 100%;
  padding: 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th {
  text-align: left;
  padding: 14px;
  background: #f9fafb;
  color: #6b7280;
}

td {
  padding: 14px;
  border-bottom: 1px solid #f0f0f0;
}

.role {
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 13px;
  font-weight: bold;
  text-transform: capitalize;
}

.role.admin {
  background: #e0e7ff;
  color: #4f46e5;
}

.role.petugas {
  background: #dcfce7;
  color: #16a34a;
}

.btn-hapus {
  border: none;
  padding: 8px;
  border-radius: 6px;
  background: #fee2e2;
  cursor: pointer;
}

.empty {
  text-align: center;
  padding: 30px;
  color: #9ca3af;
}

@media (max-width: 800px) {
  .sidebar {
    width: 200px;
  }

  .content {
    padding: 20px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }
}
</style>