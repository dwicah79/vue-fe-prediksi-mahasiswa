<script setup>
import { ref, watch } from 'vue'
import jsPDF from 'jspdf'
import autoTable from 'jspdf-autotable'
import Multiselect from 'vue-multiselect'
import 'vue-multiselect/dist/vue-multiselect.css'

const file = ref(null)
const results = ref([])
const error = ref('')
const loading = ref(false)
const page = ref(1)
const totalPages = ref(1)
const selectedPages = ref([])

async function fetchPrediction() {
  error.value = ''
  results.value = []
  loading.value = true

  const formData = new FormData()
  formData.append('file', file.value)

  try {
    const response = await fetch(
      `https://skrisi-be-production.up.railway.app/predict_excel?page=${page.value}&per_page=50`,
      {
        method: 'POST',
        body: formData,
      },
    )

    const data = await response.json()
    if (!response.ok) throw new Error(data.detail || 'Terjadi kesalahan.')

    results.value = data.results
    totalPages.value = data.total_pages

    // Reset selected pages saat file baru di-upload
    selectedPages.value = []
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

function handleFileUpload(e) {
  file.value = e.target.files[0]
}

async function printPDF() {
  error.value = ''

  if (!file.value) {
    error.value = 'Silakan upload file terlebih dahulu.'
    return
  }

  const validPages = selectedPages.value
    .map(Number)
    .filter((n) => !isNaN(n) && n >= 1 && n <= totalPages.value)

  if (validPages.length === 0) {
    error.value = 'Silakan pilih halaman yang valid.'
    return
  }

  const doc = new jsPDF()

  for (let i = 0; i < validPages.length; i++) {
    const pg = validPages[i]
    const formData = new FormData()
    formData.append('file', file.value)

    try {
      const response = await fetch(
        `https://skrisi-be-production.up.railway.app/predict_excel?page=${pg}&per_page=50`,
        {
          method: 'POST',
          body: formData,
        },
      )

      const data = await response.json()
      if (!response.ok) throw new Error(data.detail || 'Gagal ambil data.')

      const pageResults = data.results.map((item) => [
        item['No.'],
        item['NIM'],
        item['Jenis Kelamin'],
        item['SKS'],
        item['Pekerjaan Ayah'],
        item['Pekerjaan Ibu'],
        item['IPK'],
        item['prediction'] === 'LULUS' ? 'Tidak Putus Studi' : 'Putus Studi',
      ])

      autoTable(doc, {
        head: [
          [
            'No',
            'NIM',
            'Jenis Kelamin',
            'SKS',
            'Pekerjaan Ayah',
            'Pekerjaan Ibu',
            'IPK',
            'Prediksi',
          ],
        ],
        body: pageResults,
        startY: 10,
      })

      if (i < validPages.length - 1) doc.addPage()
    } catch (err) {
      error.value = err.message
      return
    }
  }

  doc.save(`hasil_prediksi_halaman_${validPages.join('-')}.pdf`)
}
</script>

<template>
  <div class="max-w-5xl mx-auto mt-10 p-6 bg-white shadow rounded">
    <h1 class="text-3xl font-bold mb-4 text-center">Upload Excel untuk Prediksi Kelulusan</h1>

    <form @submit.prevent="fetchPrediction" class="space-y-4">
      <input
        type="file"
        accept=".xls,.xlsx"
        @change="handleFileUpload"
        required
        class="border p-10 border-pink-300 border-dashed rounded-2xl w-full"
      />

      <button
        type="submit"
        class="w-full bg-pink-600 text-white py-2 rounded hover:bg-pink-700 hover:cursor-pointer"
        :disabled="loading"
      >
        <span v-if="loading">🔄 Memproses...</span>
        <span v-else>📤 Upload dan Prediksi</span>
      </button>
    </form>

    <div v-if="error" class="text-red-600 text-center mt-4">{{ error }}</div>

    <div v-if="results.length" class="mt-8">
      <h2 class="text-2xl font-semibold mb-4">
        Hasil Prediksi (Halaman {{ page }} dari {{ totalPages }})
      </h2>

      <div class="my-8">
        <label class="block mb-2 font-medium">Pilih Halaman untuk Cetak PDF</label>

        <Multiselect
          v-model="selectedPages"
          :options="Array.from({ length: totalPages }, (_, i) => i + 1)"
          :multiple="true"
          :close-on-select="false"
          :clear-on-select="false"
          :preserve-search="true"
          placeholder="Pilih Halaman..."
          class="w-full"
        />

        <button
          @click="printPDF"
          :disabled="!file || selectedPages.length === 0"
          class="mt-4 w-full bg-green-600 text-white py-2 rounded hover:bg-green-700"
        >
          🖨️ Cetak PDF
        </button>
      </div>
      <div class="overflow-x-auto">
        <table class="w-full text-sm border">
          <thead class="bg-gray-100">
            <tr>
              <th class="border p-2">No</th>
              <th class="border p-2">NIM</th>
              <th class="border p-2">Jenis Kelamin</th>
              <th class="border p-2">SKS</th>
              <th class="border p-2">Pekerjaan Ayah</th>
              <th class="border p-2">Pekerjaan Ibu</th>
              <th class="border p-2">IPK</th>
              <th class="border p-2">Prediksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, idx) in results" :key="idx" class="hover:bg-gray-50">
              <td class="border p-2">{{ item['No.'] }}</td>
              <td class="border p-2">{{ item['NIM'] }}</td>
              <td class="border p-2">{{ item['Jenis Kelamin'] }}</td>
              <td class="border p-2">{{ item['SKS'] }}</td>
              <td class="border p-2">{{ item['Pekerjaan Ayah'] }}</td>
              <td class="border p-2">{{ item['Pekerjaan Ibu'] }}</td>
              <td class="border p-2">{{ item['IPK'] }}</td>
              <td class="border p-2 font-semibold">
                {{ item['prediction'] === 'LULUS' ? 'Tidak putus Studi' : 'Putus Studi' }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="mt-4 flex justify-between items-center">
        <button
          @click="(page--, fetchPrediction())"
          :disabled="page <= 1 || loading"
          class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
        >
          ← Sebelumnya
        </button>
        <span>Halaman {{ page }} dari {{ totalPages }}</span>
        <button
          @click="(page++, fetchPrediction())"
          :disabled="page >= totalPages || loading"
          class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
        >
          Selanjutnya →
        </button>
      </div>
    </div>
  </div>
</template>
