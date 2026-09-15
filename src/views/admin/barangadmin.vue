<script setup>
import { ref, onMounted } from 'vue'
import Navbar from "../../components/Navbar.vue"
import api from '../../services/api.js'

// ===============================
// DATA BARANG
// ===============================
const listBarang = ref([])
const loading = ref(true)
const errorMessage = ref('')

// ===============================
// FORM
// ===============================
const isEditing = ref(false)
const selectedId = ref(null)

const formData = ref({
  kode_barang: '',
  nama_barang: '',
  kategori: '',
  stok: 0,
  harga: 0
})

// ===============================
// GET DATA BARANG
// ===============================
const fetchBarang = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const res = await api.get('/barang')

    console.log('Data barang:', res.data)

    // Menyesuaikan jika backend mengembalikan array langsung
    // atau { data: [...] }
    if (Array.isArray(res.data)) {
      listBarang.value = res.data
    } else if (Array.isArray(res.data.data)) {
      listBarang.value = res.data.data
    } else {
      listBarang.value = []
    }

  } catch (err) {
    console.error('Gagal mengambil data barang:', err)

    errorMessage.value =
      err.response?.data ||
      'Gagal memuat data barang dari server Go.'
  } finally {
    loading.value = false
  }
}

// ===============================
// TAMBAH / UPDATE
// ===============================
const handleSubmit = async () => {
  try {
    if (isEditing.value) {

      await api.put(
        `/barang/${selectedId.value}`,
        formData.value
      )

      alert('Data barang berhasil diperbarui!')

    } else {

      await api.post(
        '/barang',
        formData.value
      )

      alert('Data barang berhasil ditambahkan!')
    }

    resetForm()
    await fetchBarang()

  } catch (err) {
    console.error('Gagal menyimpan barang:', err)

    alert(
      err.response?.data ||
      'Gagal menyimpan data barang.'
    )
  }
}

// ===============================
// EDIT
// ===============================
const prepareEdit = (item) => {

  isEditing.value = true

  selectedId.value =
    item.id_barang ??
    item.id

  formData.value = {
    kode_barang:
      item.kode_barang || '',

    nama_barang:
      item.nama_barang ||
      item.nama ||
      '',

    kategori:
      item.kategori || '',

    stok:
      item.stok ?? 0,

    harga:
      item.harga ?? 0
  }

  // Scroll ke form
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

// ===============================
// HAPUS
// ===============================
const handleDelete = async (id) => {

  if (!id) {
    alert('ID barang tidak ditemukan.')
    return
  }

  const yakin = confirm(
    'Apakah kamu yakin ingin menghapus barang ini?'
  )

  if (!yakin) return

  try {

    await api.delete(`/barang/${id}`)

    alert('Barang berhasil dihapus!')

    await fetchBarang()

  } catch (err) {

    console.error(
      'Gagal menghapus barang:',
      err
    )

    alert(
      err.response?.data ||
      'Gagal menghapus barang.'
    )
  }
}

// ===============================
// RESET FORM
// ===============================
const resetForm = () => {

  isEditing.value = false
  selectedId.value = null

  formData.value = {
    kode_barang: '',
    nama_barang: '',
    kategori: '',
    stok: 0,
    harga: 0
  }
}

// ===============================
// FORMAT RUPIAH
// ===============================
const formatRupiah = (value) => {

  if (
    value === null ||
    value === undefined ||
    value === ''
  ) {
    return 'Rp 0'
  }

  return new Intl.NumberFormat(
    'id-ID',
    {
      style: 'currency',
      currency: 'IDR',
      minimumFractionDigits: 0
    }
  ).format(value)
}

// ===============================
// LOAD DATA
// ===============================
onMounted(() => {
  fetchBarang()
})
</script>

<template>
  <div class="layout">

    <!-- =========================
         SIDEBAR
    ========================== -->
    <Navbar />

    <!-- =========================
         MAIN CONTENT
    ========================== -->
    <main class="main-content">

      <!-- HEADER -->
      <div class="page-header">

        <div>
          <h1>Data Barang</h1>

          <p>
            Kelola data barang dan stok dalam sistem
          </p>
        </div>

        <div class="header-icon">
          📦
        </div>

      </div>


      <!-- =========================
           FORM CARD
      ========================== -->
      <section class="card form-card">

        <div class="card-header">

          <div>
            <h2>
              {{ isEditing
                ? 'Edit Data Barang'
                : 'Tambah Barang'
              }}
            </h2>

            <p>
              {{
                isEditing
                  ? 'Perbarui informasi barang yang dipilih'
                  : 'Tambahkan barang baru ke dalam sistem'
              }}
            </p>
          </div>

          <div class="card-icon">
            {{ isEditing ? '✏️' : '➕' }}
          </div>

        </div>


        <form
          @submit.prevent="handleSubmit"
          class="form-grid"
        >

          <!-- KODE -->
          <div class="input-group">

            <label>
              Kode Barang
            </label>

            <input
              v-model="formData.kode_barang"
              type="text"
              placeholder="Contoh: BRG001"
              required
            />

          </div>


          <!-- NAMA -->
          <div class="input-group">

            <label>
              Nama Barang
            </label>

            <input
              v-model="formData.nama_barang"
              type="text"
              placeholder="Masukkan nama barang"
              required
            />

          </div>


          <!-- KATEGORI -->
          <div class="input-group">

            <label>
              Kategori
            </label>

            <input
              v-model="formData.kategori"
              type="text"
              placeholder="Contoh: Elektronik"
              required
            />

          </div>


          <!-- STOK -->
          <div class="input-group">

            <label>
              Stok
            </label>

            <input
              v-model.number="formData.stok"
              type="number"
              min="0"
              placeholder="0"
              required
            />

          </div>


          <!-- HARGA -->
          <div class="input-group">

            <label>
              Harga
            </label>

            <input
              v-model.number="formData.harga"
              type="number"
              min="0"
              placeholder="Contoh: 50000"
            />

          </div>


          <!-- BUTTON -->
          <div class="form-actions">

            <button
              type="submit"
              class="btn-primary"
            >
              <span>
                {{ isEditing ? '💾' : '➕' }}
              </span>

              {{
                isEditing
                  ? 'Update Barang'
                  : 'Simpan Barang'
              }}
            </button>


            <button
              v-if="isEditing"
              type="button"
              class="btn-secondary"
              @click="resetForm"
            >
              Batal
            </button>

          </div>

        </form>

      </section>


      <!-- =========================
           TABLE CARD
      ========================== -->
      <section class="card table-card">

        <div class="card-header">

          <div>

            <h2>
              Daftar Barang
            </h2>

            <p>
              Semua barang yang tersimpan dalam database
            </p>

          </div>


          <!-- TOTAL -->
          <div class="total-box">

            <span>
              Total Barang
            </span>

            <strong>
              {{ listBarang.length }}
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
            Memuat data barang...
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
            @click="fetchBarang"
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
                  Kode Barang
                </th>

                <th>
                  Nama Barang
                </th>

                <th>
                  Kategori
                </th>

                <th>
                  Stok
                </th>

                <th>
                  Harga
                </th>

                <th>
                  Aksi
                </th>

              </tr>

            </thead>


            <tbody>

              <!-- DATA -->
              <tr
                v-for="(item, index) in listBarang"
                :key="
                  item.id_barang ||
                  item.id ||
                  index
                "
              >

                <td>
                  {{ index + 1 }}
                </td>


                <td>

                  <span class="code-badge">
                    {{ item.kode_barang }}
                  </span>

                </td>


                <td>

                  <div class="product-name">

                    <div class="product-icon">
                      📦
                    </div>

                    <strong>
                      {{
                        item.nama_barang ||
                        item.nama ||
                        '-'
                      }}
                    </strong>

                  </div>

                </td>


                <td>

                  <span class="category-badge">
                    {{ item.kategori || '-' }}
                  </span>

                </td>


                <td>

                  <span
                    class="stock-badge"
                    :class="{
                      'stock-low':
                        Number(item.stok) <= 5
                    }"
                  >
                    {{ item.stok ?? 0 }}
                  </span>

                </td>


                <td class="price">

                  {{
                    formatRupiah(item.harga)
                  }}

                </td>


                <td>

                  <div class="action-buttons">

                    <button
                      class="edit-btn"
                      @click="prepareEdit(item)"
                      title="Edit"
                    >
                      ✏️
                    </button>

                    <button
                      class="delete-btn"
                      @click="
                        handleDelete(
                          item.id_barang ||
                          item.id
                        )
                      "
                      title="Hapus"
                    >
                      🗑️
                    </button>

                  </div>

                </td>

              </tr>


              <!-- KOSONG -->
              <tr
                v-if="listBarang.length === 0"
              >

                <td
                  colspan="7"
                  class="empty-data"
                >

                  <div class="empty-icon">
                    📦
                  </div>

                  <strong>
                    Belum ada data barang
                  </strong>

                  <p>
                    Silakan tambahkan barang baru
                    menggunakan form di atas.
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

/* ==============================
   LAYOUT
============================== */

.layout {
  min-height: 100vh;
  display: flex;
  background: #f5f6fa;
  font-family: Arial, Helvetica, sans-serif;
}

.main-content {
  margin-left: 240px;
  width: calc(100% - 240px);
  min-height: 100vh;
  padding: 32px 44px;
}


/* ==============================
   HEADER
============================== */

.page-header {
  margin-bottom: 24px;
}

.page-header h1 {
  margin: 0 0 5px;
  font-size: 27px;
  font-weight: 700;
  color: #111827;
}

.page-header p {
  margin: 0;
  font-size: 14px;
  color: #64748b;
}


/* ==============================
   CARD
============================== */

.card {
  background: #fff;
  border-radius: 15px;
  padding: 24px 26px;
  margin-bottom: 22px;
  border: none;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.04);
}


/* ==============================
   CARD HEADER
============================== */

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.card-header h2 {
  margin: 0 0 5px;
  font-size: 19px;
  font-weight: 700;
  color: #111827;
}

.card-header p {
  margin: 0;
  font-size: 13px;
  color: #64748b;
}


/* ==============================
   FORM
============================== */

.form-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 18px;
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

.input-group input {
  width: 100%;
  height: 42px;
  padding: 0 13px;
  border: 1px solid #d9dee8;
  border-radius: 8px;
  background: #fff;
  outline: none;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 13px;
  color: #111827;
}

.input-group input::placeholder {
  color: #9ca3af;
}

.input-group input:focus {
  border-color: #5548e8;
}


/* ==============================
   BUTTON
============================== */

.form-actions {
  display: flex;
  gap: 8px;
}

.btn-primary {
  height: 42px;
  padding: 0 18px;
  border: none;
  border-radius: 8px;
  background: #5146e5;
  color: white;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}

.btn-primary:hover {
  background: #4338ca;
}

.btn-secondary {
  height: 42px;
  padding: 0 16px;
  border: none;
  border-radius: 8px;
  background: #e5e7eb;
  color: #374151;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}


/* ==============================
   TOTAL
============================== */

.total-box {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 13px;
  border-radius: 8px;
  background: #f0efff;
}

.total-box span {
  font-size: 12px;
  color: #64748b;
}

.total-box strong {
  font-size: 17px;
  color: #5146e5;
}


/* ==============================
   TABLE
============================== */

.table-wrapper {
  width: 100%;
  overflow-x: auto;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
}

.data-table th {
  padding: 13px 14px;
  background: #f8f9fc;
  color: #64748b;
  font-size: 12px;
  font-weight: 600;
  text-align: left;
}

.data-table td {
  padding: 13px 14px;
  border-bottom: 1px solid #edf0f4;
  color: #475569;
  font-size: 13px;
}

.data-table tbody tr:hover {
  background: #fafaff;
}


/* ==============================
   BADGE
============================== */

.code-badge {
  display: inline-block;
  padding: 5px 8px;
  border-radius: 6px;
  background: #eeecff;
  color: #5146e5;
  font-size: 11px;
  font-weight: 600;
}

.category-badge {
  display: inline-block;
  padding: 5px 8px;
  border-radius: 6px;
  background: #f1f5f9;
  color: #64748b;
  font-size: 11px;
}

.stock-badge {
  display: inline-block;
  min-width: 35px;
  padding: 5px 8px;
  text-align: center;
  border-radius: 6px;
  background: #dcfce7;
  color: #16a34a;
  font-size: 11px;
  font-weight: 600;
}

.stock-badge.stock-low {
  background: #fee2e2;
  color: #dc2626;
}

.price {
  font-weight: 600;
  color: #334155 !important;
}


/* ==============================
   PRODUCT NAME
============================== */

.product-name {
  display: flex;
  align-items: center;
  gap: 8px;
}

.product-icon {
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #eef2ff;
  border-radius: 7px;
  font-size: 14px;
}

.product-name strong {
  font-size: 13px;
  font-weight: 500;
  color: #334155;
}


/* ==============================
   ACTION
============================== */

.action-buttons {
  display: flex;
  gap: 6px;
}

.edit-btn,
.delete-btn {
  width: 32px;
  height: 32px;
  border: none;
  border-radius: 7px;
  cursor: pointer;
  font-size: 13px;
}

.edit-btn {
  background: #fff7ed;
}

.delete-btn {
  background: #fef2f2;
}

.edit-btn:hover {
  background: #ffedd5;
}

.delete-btn:hover {
  background: #fee2e2;
}


/* ==============================
   STATE
============================== */

.state-box {
  padding: 40px;
  text-align: center;
  color: #64748b;
}

.state-box p {
  margin: 0;
  font-size: 13px;
}

.state-box.error {
  color: #dc2626;
}

.retry-btn {
  margin-top: 10px;
  padding: 8px 14px;
  border: none;
  border-radius: 7px;
  background: #5146e5;
  color: white;
  cursor: pointer;
}


/* ==============================
   EMPTY
============================== */

.empty-data {
  padding: 40px !important;
  text-align: center !important;
  color: #94a3b8;
}

.empty-icon {
  font-size: 28px;
  margin-bottom: 8px;
}

.empty-data strong {
  display: block;
  margin-bottom: 4px;
  color: #475569;
  font-size: 13px;
}

.empty-data p {
  margin: 0;
  font-size: 12px;
}


/* ==============================
   RESPONSIVE
============================== */

@media (max-width: 1000px) {
  .form-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 700px) {
  .main-content {
    margin-left: 0;
    width: 100%;
    padding: 20px;
  }

  .form-grid {
    grid-template-columns: 1fr;
  }
}

</style>