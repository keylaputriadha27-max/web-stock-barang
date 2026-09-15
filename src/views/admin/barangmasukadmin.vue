<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import api from "../../services/api"

const router = useRouter()

// =================================
// DATA
// =================================

const listBarangMasuk = ref([])
const listMasterBarang = ref([])
const loading = ref(true)
const errorMessage = ref('')

// =================================
// FORM
// =================================

const formData = ref({
  id_barang: '',
  jumlah: 1,
  tanggal: new Date().toISOString().split('T')[0],
  keterangan: ''
})

// =================================
// FETCH BARANG MASUK
// =================================

const fetchBarangMasuk = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const res = await api.get('/barang-masuk')

    if (Array.isArray(res.data)) {
      listBarangMasuk.value = res.data
    } else if (Array.isArray(res.data.data)) {
      listBarangMasuk.value = res.data.data
    } else {
      listBarangMasuk.value = []
    }
  } catch (err) {
    console.error('Gagal mengambil data barang masuk:', err)

    if (
      err.response?.status === 401 ||
      err.response?.status === 403
    ) {
      alert('Sesi telah berakhir, silakan login kembali.')

      localStorage.removeItem('token')
      router.push('/')
    } else {
      errorMessage.value =
        err.response?.data?.message ||
        (typeof err.response?.data === 'string'
          ? err.response.data
          : null) ||
        'Gagal memuat data barang masuk dari server.'
    }
  } finally {
    loading.value = false
  }
}

// =================================
// FETCH MASTER BARANG
// =================================

const fetchMasterBarang = async () => {
  try {
    const res = await api.get('/barang')

    if (Array.isArray(res.data)) {
      listMasterBarang.value = res.data
    } else if (Array.isArray(res.data.data)) {
      listMasterBarang.value = res.data.data
    } else {
      listMasterBarang.value = []
    }
  } catch (err) {
    console.error('Gagal mengambil master barang:', err)
  }
}

// =================================
// TAMBAH BARANG MASUK
// =================================

const handleSubmit = async () => {
  if (!formData.value.id_barang) {
    alert('Silakan pilih barang terlebih dahulu.')
    return
  }

  if (!formData.value.jumlah || formData.value.jumlah < 1) {
    alert('Jumlah barang harus lebih dari 0.')
    return
  }

  try {
    await api.post('/barang-masuk', formData.value)

    alert('Barang masuk berhasil dicatat!')

    resetForm()

    await fetchBarangMasuk()
  } catch (err) {
    console.error('Gagal mencatat barang masuk:', err)

    if (
      err.response?.status === 401 ||
      err.response?.status === 403
    ) {
      alert('Sesi telah berakhir, silakan login kembali.')

      localStorage.removeItem('token')
      router.push('/')
    } else {
      alert(
        err.response?.data?.message ||
        (typeof err.response?.data === 'string'
          ? err.response.data
          : null) ||
        'Gagal mencatat barang masuk.'
      )
    }
  }
}

// =================================
// RESET FORM
// =================================

const resetForm = () => {
  formData.value = {
    id_barang: '',
    jumlah: 1,
    tanggal: new Date().toISOString().split('T')[0],
    keterangan: ''
  }
}

// =================================
// FORMAT TANGGAL
// =================================

const formatDate = (dateString) => {
  if (!dateString) return '-'

  const date = new Date(dateString)

  if (isNaN(date)) return dateString

  return date.toLocaleDateString('id-ID', {
    day: '2-digit',
    month: 'long',
    year: 'numeric'
  })
}

// =================================
// LOGOUT
// =================================

const handleLogout = () => {
  localStorage.removeItem('token')
  router.push('/')
}

// =================================
// ON MOUNTED
// =================================

onMounted(() => {
  fetchBarangMasuk()
  fetchMasterBarang()
})
</script>


<template>
  <div class="page">

    <!-- ================================= -->
    <!-- SIDEBAR -->
    <!-- SAMA SEPERTI DATA USER -->
    <!-- ================================= -->

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

        <router-link to="/dashboardadmin">
          📊 Dashboard
        </router-link>

        <router-link to="/barangadmin">
          📦 Data Barang
        </router-link>

        <router-link
          to="/barangmasukadmin"
          class="active"
        >
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
      <a
        href="#"
        class="logout"
        @click.prevent="handleLogout"
      >
        🚪 Logout
      </a>

    </aside>


    <!-- ================================= -->
    <!-- CONTENT -->
    <!-- ================================= -->

    <main class="content">

      <!-- HEADER -->
      <header class="header">

        <div>

          <h1>
            Barang Masuk
          </h1>

          <p>
            Kelola data barang yang masuk
          </p>

        </div>


        <!-- PROFILE -->
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


      <!-- ================================= -->
      <!-- FORM BARANG MASUK -->
      <!-- ================================= -->

      <section class="card">

        <div class="card-header">

          <div>

            <h2>
              Tambah Barang Masuk
            </h2>

            <p>
              Catat transaksi barang yang masuk ke gudang
            </p>

          </div>

        </div>


        <form
          class="form-grid"
          @submit.prevent="handleSubmit"
        >

          <!-- PILIH BARANG -->
          <div class="input-group">

            <label>
              Pilih Barang
            </label>

            <select
              v-model="formData.id_barang"
              required
            >

              <option value="">
                -- Pilih Barang --
              </option>

              <option
                v-for="barang in listMasterBarang"
                :key="barang.id"
                :value="barang.id"
              >

                {{
                  barang.kode_barang ||
                  barang.kode ||
                  '-'
                }}

                -

                {{
                  barang.nama_barang ||
                  barang.nama ||
                  '-'
                }}

              </option>

            </select>

          </div>


          <!-- JUMLAH -->
          <div class="input-group">

            <label>
              Jumlah Masuk
            </label>

            <input
              v-model.number="formData.jumlah"
              type="number"
              min="1"
              placeholder="1"
              required
            />

          </div>


          <!-- TANGGAL -->
          <div class="input-group">

            <label>
              Tanggal Masuk
            </label>

            <input
              v-model="formData.tanggal"
              type="date"
              required
            />

          </div>


          <!-- KETERANGAN -->
          <div class="input-group full-width">

            <label>
              Keterangan / Supplier
            </label>

            <input
              v-model="formData.keterangan"
              type="text"
              placeholder="Contoh: Restok dari Supplier A"
            />

          </div>


          <!-- BUTTON -->
          <div class="form-actions">

            <button
              type="submit"
              class="btn-primary"
            >
              📥 Simpan Transaksi
            </button>

          </div>

        </form>

      </section>


      <!-- ================================= -->
      <!-- RIWAYAT BARANG MASUK -->
      <!-- ================================= -->

      <section class="card">

        <div class="card-header">

          <div>

            <h2>
              Riwayat Barang Masuk
            </h2>

            <p>
              Daftar seluruh transaksi barang masuk yang tercatat
            </p>

          </div>


          <div class="total-box">

            <span>
              Total Transaksi
            </span>

            <strong>
              {{ listBarangMasuk.length }}
            </strong>

          </div>

        </div>


        <!-- LOADING -->
        <div
          v-if="loading"
          class="state-box"
        >

          <div class="loading-icon">
            ⏳
          </div>

          <p>
            Memuat riwayat barang masuk...
          </p>

        </div>


        <!-- ERROR -->
        <div
          v-else-if="errorMessage"
          class="state-box error"
        >

          <div class="error-icon">
            ⚠️
          </div>

          <p>
            {{ errorMessage }}
          </p>

          <button
            @click="fetchBarangMasuk"
            class="retry-btn"
          >
            Coba Lagi
          </button>

        </div>


        <!-- TABLE -->
        <div
          v-else
          class="table-wrapper"
        >

          <table class="data-table">

            <thead>

              <tr>

                <th>
                  #
                </th>

                <th>
                  Tanggal
                </th>

                <th>
                  Kode Barang
                </th>

                <th>
                  Nama Barang
                </th>

                <th>
                  Jumlah Masuk
                </th>

                <th>
                  Keterangan
                </th>

              </tr>

            </thead>


            <tbody>

              <tr
                v-for="(item, index) in listBarangMasuk"
                :key="item.id || index"
              >

                <!-- NO -->
                <td>
                  {{ index + 1 }}
                </td>


                <!-- TANGGAL -->
                <td>
                  {{
                    formatDate(
                      item.tanggal ||
                      item.created_at
                    )
                  }}
                </td>


                <!-- KODE -->
                <td>

                  <span class="code-badge">

                    {{
                      item.barang?.kode_barang ||
                      item.kode_barang ||
                      item.barang?.kode ||
                      item.kode ||
                      '-'
                    }}

                  </span>

                </td>


                <!-- NAMA -->
                <td>

                  <strong>

                    {{
                      item.barang?.nama_barang ||
                      item.nama_barang ||
                      item.barang?.nama ||
                      item.nama ||
                      '-'
                    }}

                  </strong>

                </td>


                <!-- JUMLAH -->
                <td>

                  <span class="stock-badge">
                    +{{ item.jumlah || 0 }}
                  </span>

                </td>


                <!-- KETERANGAN -->
                <td>
                  {{ item.keterangan || '-' }}
                </td>

              </tr>


              <!-- DATA KOSONG -->
              <tr
                v-if="listBarangMasuk.length === 0"
              >

                <td
                  colspan="6"
                  class="empty-data"
                >

                  <div class="empty-icon">
                    📥
                  </div>

                  <strong>
                    Belum ada transaksi barang masuk
                  </strong>

                  <p>
                    Gunakan form di atas untuk mencatat barang masuk baru.
                  </p>

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

/* ================================= */
/* RESET */
/* ================================= */

* {
  box-sizing: border-box;
}


/* ================================= */
/* PAGE */
/* SAMA SEPERTI DATA USER */
/* ================================= */

.page {
  min-height: 100vh;

  display: flex;

  background: #f5f7fb;

  font-family: Arial, sans-serif;
}


/* ================================= */
/* SIDEBAR */
/* SAMA SEPERTI DATA USER */
/* ================================= */

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


/* ================================= */
/* LOGO */
/* ================================= */

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


/* ================================= */
/* MENU */
/* ================================= */

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

  display: block;
}


.menu a:hover {
  background: #f3f4f6;
}


.menu .active {
  background: #eef2ff;

  color: #4f46e5;

  font-weight: bold;
}


/* ================================= */
/* LOGOUT */
/* ================================= */

.logout {
  margin-top: auto;

  padding: 13px;

  text-align: center;

  text-decoration: none;

  background: #fee2e2;

  color: #dc2626;

  border-radius: 9px;
}


.logout:hover {
  background: #fecaca;
}


/* ================================= */
/* CONTENT */
/* SAMA SEPERTI DATA USER */
/* ================================= */

.content {
  flex: 1;

  padding: 35px;

  min-width: 0;
}


/* ================================= */
/* HEADER */
/* ================================= */

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

  margin: 8px 0 0;
}


/* ================================= */
/* PROFILE */
/* ================================= */

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


/* ================================= */
/* CARD */
/* ================================= */

.card {
  background: white;

  padding: 25px;

  border-radius: 15px;

  margin-bottom: 22px;
}


/* ================================= */
/* CARD HEADER */
/* ================================= */

.card-header {
  display: flex;

  justify-content: space-between;

  align-items: center;

  margin-bottom: 25px;
}


.card-header h2 {
  margin: 0;

  font-size: 21px;

  color: #1f2937;
}


.card-header p {
  color: #6b7280;

  margin: 7px 0 0;

  font-size: 14px;
}


/* ================================= */
/* FORM */
/* ================================= */

.form-grid {
  display: grid;

  grid-template-columns: repeat(3, 1fr);

  gap: 12px;

  align-items: end;
}


.input-group {
  display: flex;

  flex-direction: column;

  gap: 7px;
}


.input-group label {
  font-size: 13px;

  font-weight: 600;

  color: #374151;
}


.input-group input,
.input-group select {
  width: 100%;

  height: 42px;

  padding: 0 12px;

  border: 1px solid #d1d5db;

  border-radius: 8px;

  background: white;

  outline: none;

  font-size: 13px;
}


.input-group input:focus,
.input-group select:focus {
  border-color: #4f46e5;

  box-shadow:
    0 0 0 2px rgba(79, 70, 229, 0.1);
}


.full-width {
  grid-column: 1 / -1;
}


/* ================================= */
/* FORM ACTION */
/* ================================= */

.form-actions {
  grid-column: 1 / -1;

  display: flex;

  justify-content: flex-end;

  margin-top: 5px;
}


.btn-primary {
  border: none;

  padding: 12px 18px;

  border-radius: 8px;

  background: #4f46e5;

  color: white;

  cursor: pointer;

  font-weight: 600;
}


.btn-primary:hover {
  background: #4338ca;
}


/* ================================= */
/* TOTAL TRANSAKSI */
/* ================================= */

.total-box {
  display: flex;

  align-items: center;

  gap: 10px;

  padding: 9px 14px;

  border-radius: 8px;

  background: #eef2ff;

  color: #374151;

  font-size: 14px;
}


.total-box strong {
  color: #4f46e5;

  font-size: 17px;
}


/* ================================= */
/* TABLE */
/* ================================= */

.table-wrapper {
  width: 100%;

  overflow-x: auto;
}


.data-table {
  width: 100%;

  border-collapse: collapse;

  min-width: 700px;
}


.data-table th {
  text-align: left;

  padding: 14px;

  background: #f9fafb;

  color: #6b7280;

  font-size: 13px;

  font-weight: 600;
}


.data-table td {
  padding: 14px;

  border-bottom: 1px solid #f0f0f0;

  font-size: 13px;

  color: #555;
}


.data-table tbody tr:last-child td {
  border-bottom: none;
}


/* ================================= */
/* CODE BADGE */
/* ================================= */

.code-badge {
  display: inline-block;

  padding: 6px 10px;

  border-radius: 6px;

  background: #e0e7ff;

  color: #4f46e5;

  font-size: 11px;

  font-weight: bold;
}


/* ================================= */
/* STOCK BADGE */
/* ================================= */

.stock-badge {
  display: inline-block;

  padding: 6px 11px;

  border-radius: 20px;

  background: #dcfce7;

  color: #16a34a;

  font-size: 12px;

  font-weight: bold;
}


/* ================================= */
/* STATE */
/* ================================= */

.state-box {
  text-align: center;

  padding: 35px;

  color: #9ca3af;
}


.loading-icon,
.error-icon,
.empty-icon {
  font-size: 28px;

  margin-bottom: 10px;
}


.state-box p {
  margin: 5px 0;
}


/* ================================= */
/* ERROR */
/* ================================= */

.state-box.error {
  color: #dc2626;
}


.retry-btn {
  margin-top: 10px;

  padding: 10px 18px;

  border: none;

  border-radius: 7px;

  background: #4f46e5;

  color: white;

  cursor: pointer;
}


.retry-btn:hover {
  background: #4338ca;
}


/* ================================= */
/* EMPTY DATA */
/* ================================= */

.empty-data {
  text-align: center !important;

  padding: 40px !important;

  color: #9ca3af !important;
}


.empty-data strong {
  display: block;

  color: #555;

  font-size: 14px;
}


.empty-data p {
  margin-top: 7px;

  color: #9ca3af;

  font-size: 13px;
}


/* ================================= */
/* RESPONSIVE */
/* ================================= */

@media (max-width: 900px) {

  .sidebar {
    width: 220px;
  }

  .content {
    padding: 25px;
  }

  .form-grid {
    grid-template-columns: 1fr 1fr;
  }

}


@media (max-width: 700px) {

  .sidebar {
    width: 200px;
  }

  .content {
    padding: 20px;
  }

  .header {
    align-items: flex-start;

    gap: 20px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }

  .full-width {
    grid-column: auto;
  }

}


@media (max-width: 550px) {

  .sidebar {
    width: 180px;

    padding: 20px 10px;
  }

  .logo {
    padding-left: 5px;

    font-size: 19px;
  }

  .menu a {
    padding: 11px 10px;

    font-size: 13px;
  }

  .content {
    padding: 15px;
  }

  .profile {
    display: none;
  }

  .card {
    padding: 18px;
  }

}

</style>