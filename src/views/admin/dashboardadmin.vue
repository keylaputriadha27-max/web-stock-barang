<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import api from '../../services/api'

const router = useRouter()

const listBarang = ref([])
const loading = ref(true)
const errorMessage = ref('')

const totalBarang = computed(() => listBarang.value.length)

const barangMasuk = ref(45)
const barangKeluar = ref(30)
const totalUser = ref(5)

const totalBarangMasuk = computed(() => barangMasuk.value)
const totalAngkaKeluar = computed(() => barangKeluar.value)

const transaksiTerbaru = computed(() => {
  return listBarang.value.slice(0, 5)
})

const fetchDashboardData = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const res = await api.get('/barang')

    if (Array.isArray(res.data)) {
      listBarang.value = res.data
    } else if (Array.isArray(res.data?.data)) {
      listBarang.value = res.data.data
    } else {
      listBarang.value = []
    }

  } catch (err) {
    console.error('Gagal mengambil data dashboard:', err)

    if (
      err.response?.status === 401 ||
      err.response?.status === 403
    ) {
      alert('Sesi telah berakhir, silakan login kembali.')

      localStorage.removeItem('token')

      router.push('/')
    } else {
      errorMessage.value =
        'Gagal memuat data dari server backend.'
    }

  } finally {
    loading.value = false
  }
}

const handleLogout = () => {
  localStorage.removeItem('token')
  router.push('/')
}

const goTo = (path) => {
  router.push(path)
}

const formatTanggal = (tanggal) => {
  if (!tanggal) {
    return '01 September 2026'
  }

  const date = new Date(tanggal)

  if (isNaN(date.getTime())) {
    return tanggal
  }

  return date.toLocaleDateString('id-ID', {
    day: '2-digit',
    month: 'long',
    year: 'numeric'
  })
}

onMounted(() => {
  fetchDashboardData()
})
</script>

<template>
  <div class="page">

    <!-- SIDEBAR -->
    <aside class="sidebar">

      <!-- LOGO -->
      <div class="logo">
        <div class="logo-icon">
          📦
        </div>

        <span>
          StockKu
        </span>
      </div>

      <!-- MENU -->
      <nav class="menu">

        <router-link
          to="/dashboardadmin"
          class="active"
        >
          📊 Dashboard
        </router-link>

        <router-link to="/barangadmin">
          📦 Data Barang
        </router-link>

        <router-link to="/barangmasukadmin">
          📥 Barang Masuk
        </router-link>

        <router-link to="/barangkeluaradmin">
          📤 Barang Keluar
        </router-link>

        <router-link to="/useradmin">
          👥 Data User
        </router-link>

        <router-link to="/riwayatadmin">
          🕒 Riwayat
        </router-link>

        <router-link to="/pengaturan">
          ⚙️ Pengaturan
        </router-link>

      </nav>

      <!-- LOGOUT -->
      <router-link
        to="/"
        class="logout"
        @click="handleLogout"
      >
        🚪 Logout
      </router-link>

    </aside>

    <!-- CONTENT -->
    <main class="content">

      <!-- HEADER -->
      <header class="header">

        <div>
          <h1>
            Dashboard
          </h1>

          <p>
            Selamat datang kembali, Admin 👋
          </p>
        </div>

        <div class="profile">

          <div class="avatar">
            A
          </div>

          <div>
            <strong>
              Admin
            </strong>

            <small>
              Administrator
            </small>
          </div>

        </div>

      </header>

      <!-- LOADING -->
      <div
        v-if="loading"
        class="loading"
      >
        Memuat data dari server...
      </div>

      <!-- ERROR -->
      <div
        v-else-if="errorMessage"
        class="error-box"
      >
        {{ errorMessage }}
      </div>

      <!-- DASHBOARD -->
      <div v-else>

        <!-- STATISTIK -->
        <div class="stats-grid">

          <div class="stat-card">

            <div class="stat-icon blue">
              📦
            </div>

            <div class="stat-content">

              <span>
                Total Barang
              </span>

              <strong>
                {{ totalBarang }}
              </strong>

            </div>

          </div>

          <div class="stat-card">

            <div class="stat-icon green">
              📥
            </div>

            <div class="stat-content">

              <span>
                Barang Masuk
              </span>

              <strong>
                {{ totalBarangMasuk }}
              </strong>

            </div>

          </div>

          <div class="stat-card">

            <div class="stat-icon orange">
              📤
            </div>

            <div class="stat-content">

              <span>
                Barang Keluar
              </span>

              <strong>
                {{ totalAngkaKeluar }}
              </strong>

            </div>

          </div>

          <div class="stat-card">

            <div class="stat-icon purple">
              👥
            </div>

            <div class="stat-content">

              <span>
                Total User
              </span>

              <strong>
                {{ totalUser }}
              </strong>

            </div>

          </div>

        </div>

        <!-- TRANSAKSI -->
        <section class="transaction-card">

          <div class="transaction-header">

            <div>

              <h2>
                Transaksi Terbaru
              </h2>

              <p>
                Daftar aktivitas barang terbaru
              </p>

            </div>

            <button
              class="view-all"
              @click="goTo('/riwayatadmin')"
            >
              Lihat Semua
            </button>

          </div>

          <div class="table-container">

            <table>

              <thead>

                <tr>
                  <th>Kode</th>
                  <th>Nama Barang</th>
                  <th>Jenis</th>
                  <th>Jumlah</th>
                  <th>Tanggal</th>
                </tr>

              </thead>

              <tbody>

                <tr
                  v-for="(item, index) in transaksiTerbaru"
                  :key="item.id || index"
                >

                  <td>
                    {{
                      item.kode_barang ||
                      item.kode ||
                      `BRG00${index + 1}`
                    }}
                  </td>

                  <td>
                    <strong class="product-name">
                      {{
                        item.nama_barang ||
                        item.nama ||
                        '-'
                      }}
                    </strong>
                  </td>

                  <td>
                    <span class="badge masuk">
                      Barang Masuk
                    </span>
                  </td>

                  <td>
                    {{
                      item.stok ??
                      item.jumlah ??
                      0
                    }}
                  </td>

                  <td>
                    {{
                      formatTanggal(
                        item.created_at ||
                        item.tanggal
                      )
                    }}
                  </td>

                </tr>

                <tr
                  v-if="transaksiTerbaru.length === 0"
                >
                  <td
                    colspan="5"
                    class="empty"
                  >
                    Belum ada data barang.
                  </td>
                </tr>

              </tbody>

            </table>

          </div>

        </section>

      </div>

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

/* SIDEBAR */

.sidebar {
  width: 250px;
  min-height: 100vh;
  background: white;
  padding: 25px 15px;
  border-right: 1px solid #e5e7eb;

  display: flex;
  flex-direction: column;

  flex-shrink: 0;
}

/* LOGO */

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
  font-size: 21px;
}

/* MENU */

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

  display: flex;
  align-items: center;

  gap: 8px;

  font-size: 15px;
}

.menu a:hover {
  background: #f3f4f6;
}

.menu .active {
  background: #eef2ff;
  color: #4f46e5;
  font-weight: bold;
}

/* LOGOUT */

.logout {
  margin-top: auto;

  padding: 13px;

  text-align: center;

  text-decoration: none;

  background: #fee2e2;

  color: #dc2626;

  border-radius: 9px;

  font-size: 14px;
}

.logout:hover {
  background: #fdd4d4;
}

/* CONTENT */

.content {
  flex: 1;
  padding: 35px;
  min-width: 0;
}

/* HEADER */

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
  margin-top: 8px;
}

/* PROFILE */

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

/* STATS */

.stats-grid {
  display: grid;

  grid-template-columns:
    repeat(4, 1fr);

  gap: 20px;

  margin-bottom: 30px;
}

.stat-card {
  background: white;

  min-height: 90px;

  border-radius: 15px;

  padding: 20px;

  display: flex;

  align-items: center;

  gap: 16px;

  box-shadow:
    0 3px 12px rgba(0, 0, 0, 0.04);
}

.stat-icon {
  width: 50px;
  height: 50px;

  border-radius: 11px;

  display: flex;
  align-items: center;
  justify-content: center;

  font-size: 22px;
}

.stat-icon.blue {
  background: #dceaff;
}

.stat-icon.green {
  background: #dcfce8;
}

.stat-icon.orange {
  background: #fff0d7;
}

.stat-icon.purple {
  background: #eee2ff;
}

.stat-content {
  display: flex;
  flex-direction: column;

  gap: 5px;
}

.stat-content span {
  color: #686868;
  font-size: 14px;
}

.stat-content strong {
  font-size: 25px;
  color: #202020;
}

/* TRANSACTION */

.transaction-card {
  background: white;

  border-radius: 15px;

  padding: 26px;

  box-shadow:
    0 3px 12px rgba(0, 0, 0, 0.04);
}

.transaction-header {
  display: flex;

  justify-content: space-between;

  align-items: center;

  margin-bottom: 26px;
}

.transaction-header h2 {
  margin: 0;

  font-size: 23px;

  color: #202020;
}

.transaction-header p {
  margin: 7px 0 0;

  font-size: 14px;

  color: #777;
}

.view-all {
  border: none;

  background: #4f46e5;

  color: white;

  padding: 10px 18px;

  border-radius: 8px;

  font-size: 13px;

  font-weight: 600;

  cursor: pointer;
}

/* TABLE */

.table-container {
  width: 100%;
  overflow-x: auto;
}

table {
  width: 100%;

  border-collapse: collapse;

  min-width: 700px;
}

thead {
  background: #f8f9fb;
}

th {
  padding: 14px;

  text-align: left;

  font-size: 13px;

  font-weight: 600;

  color: #666;
}

td {
  padding: 15px 14px;

  border-bottom:
    1px solid #eeeeee;

  font-size: 13px;

  color: #555;
}

.product-name {
  color: #4b4b4b;
}

.badge {
  display: inline-flex;

  align-items: center;

  padding: 6px 13px;

  border-radius: 20px;

  font-size: 12px;

  font-weight: 600;
}

.badge.masuk {
  background: #dcfce7;

  color: #24964c;
}

.empty {
  text-align: center;

  padding: 30px;

  color: #888;
}

/* LOADING */

.loading {
  background: white;

  border-radius: 15px;

  padding: 30px;

  text-align: center;

  color: #666;
}

/* ERROR */

.error-box {
  background: #fee2e2;

  color: #dc2626;

  padding: 18px;

  border-radius: 10px;
}

/* RESPONSIVE */

@media (max-width: 1100px) {
  .stats-grid {
    grid-template-columns:
      repeat(2, 1fr);
  }
}

@media (max-width: 800px) {
  .sidebar {
    width: 200px;
  }

  .content {
    padding: 20px;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 600px) {
  .sidebar {
    width: 70px;
    padding: 20px 10px;
  }

  .logo span,
  .menu a,
  .logout {
    font-size: 0;
  }

  .logo {
    justify-content: center;
    padding: 0 0 30px;
  }

  .menu a {
    justify-content: center;
    padding: 13px 0;
  }

  .logout {
    padding: 13px 0;
  }

  .content {
    padding: 20px 15px;
  }

  .profile > div:last-child {
    display: none;
  }
}

</style>