<script setup>
import { ref, computed, onMounted } from "vue"
import api from "../../services/api"

const search = ref("")
const showForm = ref(false)
const loading = ref(false)

// Ref untuk list data dari Backend
const barangKeluar = ref([])
const listMasterBarang = ref([])

// Form state yang disesuaikan dengan struct Go
const idBarang = ref("")
const jumlah = ref(1)
const tanggal = ref(new Date().toISOString().split("T")[0])
const keterangan = ref("")

// 1. Fetch Data Barang Keluar dari Backend
const fetchBarangKeluar = async () => {
  loading.value = true
  try {
    const res = await api.get("/barang-keluar")
    barangKeluar.value = Array.isArray(res.data) ? res.data : (res.data.data || [])
  } catch (err) {
    console.error("Gagal mengambil data barang keluar:", err)
    alert("Gagal memuat data barang keluar!")
  } finally {
    loading.value = false
  }
}

// 2. Fetch Master Barang (untuk Pilihan Select Option)
const fetchMasterBarang = async () => {
  try {
    const res = await api.get("/barang")
    listMasterBarang.value = Array.isArray(res.data) ? res.data : (res.data.data || [])
  } catch (err) {
    console.error("Gagal mengambil master barang:", err)
  }
}

// Filter Pencarian
const filteredBarangKeluar = computed(() => {
  return barangKeluar.value.filter((item) => {
    const nama = item.nama_barang || item.namaBarang || ""
    const kode = item.kode_barang || item.kode || ""
    const q = search.value.toLowerCase()
    
    return nama.toLowerCase().includes(q) || kode.toLowerCase().includes(q)
  })
})

// 3. Tambah Barang Keluar ke Backend (POST)
async function tambahBarang() {
  if (!idBarang.value || !jumlah.value || !tanggal.value) {
    alert("Barang, jumlah, dan tanggal wajib diisi!")
    return
  }

  try {
    const payload = {
      id_barang: Number(idBarang.value),
      jumlah: Number(jumlah.value),
      tanggal: tanggal.value,
      keterangan: keterangan.value
    }

    await api.post("/barang-keluar", payload)
    alert("Barang keluar berhasil dicatat!")

    // Reset Form
    idBarang.value = ""
    jumlah.value = 1
    tanggal.value = new Date().toISOString().split("T")[0]
    keterangan.value = ""
    showForm.value = false

    // Refresh Data
    await fetchBarangKeluar()
    await fetchMasterBarang()
  } catch (err) {
    console.error("Gagal menambah barang keluar:", err)
    alert(err.response?.data || "Gagal mencatat barang keluar. Cek kembali sisa stok!")
  }
}

onMounted(() => {
  fetchBarangKeluar()
  fetchMasterBarang()
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
        <router-link to="/dashboard">📊 Dashboard</router-link>
        <router-link to="/barang">📦 Data Barang</router-link>
        <router-link to="/barang-masuk">📥 Barang Masuk</router-link>
        <router-link to="/barang-keluar" class="active">📤 Barang Keluar</router-link>
        <router-link to="/user">👥 Data User</router-link>
        <router-link to="/riwayat">🕒 Riwayat</router-link>
        <router-link to="/pengaturan">⚙️ Pengaturan</router-link>
      </nav>

      <router-link to="/" class="logout">🚪 Logout</router-link>
    </aside>

    <!-- CONTENT -->
    <main class="content">

      <header class="header">
        <div>
          <h1>Barang Keluar</h1>
          <p>Kelola data barang yang keluar</p>
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
            <h2>Daftar Barang Keluar</h2>
            <p>Total {{ barangKeluar.length }} transaksi</p>
          </div>

          <button class="btn-tambah" @click="showForm = !showForm">
            + Tambah Barang Keluar
          </button>
        </div>

        <!-- FORM -->
        <div v-if="showForm" class="form">
          <h3>Tambah Barang Keluar</h3>

          <div class="form-grid">
            <!-- Dropdown Pilihan Barang -->
            <select v-model="idBarang" class="input-select">
              <option value="" disabled>-- Pilih Barang --</option>
              <option 
                v-for="b in listMasterBarang" 
                :key="b.id_barang || b.id" 
                :value="b.id_barang || b.id"
              >
                {{ b.kode_barang || b.kode }} - {{ b.nama_barang || b.nama }} (Stok: {{ b.stok }})
              </option>
            </select>

            <input
              v-model="jumlah"
              type="number"
              min="1"
              placeholder="Jumlah Keluar"
            />

            <input
              v-model="tanggal"
              type="date"
            />

            <input
              v-model="keterangan"
              type="text"
              placeholder="Keterangan / Tujuan Barang"
            />
          </div>

          <div class="form-actions">
            <button class="btn-simpan" @click="tambahBarang">Simpan</button>
            <button class="btn-batal" @click="showForm = false">Batal</button>
          </div>
        </div>

        <!-- SEARCH -->
        <div class="search">
          <input
            v-model="search"
            type="text"
            placeholder="🔍 Cari barang atau kode..."
          />
        </div>

        <!-- TABLE -->
        <div class="table-container">
          <div v-if="loading" class="empty">Memuat data...</div>

          <table v-else>
            <thead>
              <tr>
                <th>No</th>
                <th>Kode</th>
                <th>Nama Barang</th>
                <th>Jumlah</th>
                <th>Tanggal</th>
                <th>Keterangan / Tujuan</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="(item, index) in filteredBarangKeluar" :key="item.id || index">
                <td>{{ index + 1 }}</td>
                <td><strong>{{ item.kode_barang || item.kode || '-' }}</strong></td>
                <td>{{ item.nama_barang || item.namaBarang || '-' }}</td>
                <td>
                  <span class="jumlah">-{{ item.jumlah }}</span>
                </td>
                <td>{{ item.tanggal }}</td>
                <td>{{ item.keterangan || item.tujuan || '-' }}</td>
              </tr>

              <tr v-if="filteredBarangKeluar.length === 0">
                <td colspan="6" class="empty">Data tidak ditemukan</td>
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
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.form-grid input, .input-select {
  padding: 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  background-color: white;
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

.jumlah {
  padding: 6px 12px;
  border-radius: 20px;
  background: #fee2e2;
  color: #dc2626;
  font-weight: bold;
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