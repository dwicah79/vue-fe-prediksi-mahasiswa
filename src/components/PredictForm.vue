<script setup>
import { ref } from 'vue'

const file = ref(null)
const results = ref([])
const error = ref('')
const loading = ref(false)
const page = ref(1)
const totalPages = ref(1)

async function fetchPrediction() {
  error.value = ''
  results.value = []
  loading.value = true

  const formData = new FormData()
  formData.append('file', file.value)

  try {
    const response = await fetch(`http://127.0.0.1:8000/predict_excel?page=${page.value}&per_page=50`, {
      method: 'POST',
      body: formData,
    })

    const data = await response.json()

    if (!response.ok) throw new Error(data.detail || 'Terjadi kesalahan.')

    results.value = data.results
    totalPages.value = data.total_pages
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

function handleFileUpload(e) {
  file.value = e.target.files[0]
}
</script>

<template>
  <div class="max-w-5xl mx-auto mt-10 p-6 bg-white shadow rounded">
    <h1 class="text-3xl font-bold mb-4 text-center">Upload Excel untuk Prediksi Kelulusan</h1>

    <form @submit.prevent="fetchPrediction" class="space-y-4">
      <input type="file" accept=".xls,.xlsx" @change="handleFileUpload" required class="border p-10 border-pink-300 border-dashed rounded-2xl w-full" />

      <button type="submit" class="w-full bg-pink-600 text-white py-2 rounded hover:bg-pink-700 hover:cursor-pointer" :disabled="loading">
        <span v-if="loading">🔄 Memproses...</span>
        <span v-else>📤 Upload dan Prediksi</span>
      </button>
    </form>

    <div v-if="error" class="text-red-600 text-center mt-4">{{ error }}</div>

    <div v-if="results.length" class="mt-8">
      <h2 class="text-2xl font-semibold mb-4">Hasil Prediksi (Halaman {{ page }} dari {{ totalPages }})</h2>
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
              <td class="border p-2">{{ item["No."] }}</td>
              <td class="border p-2">{{ item["NIM"] }}</td>
              <td class="border p-2">{{ item["Jenis Kelamin"] }}</td>
              <td class="border p-2">{{ item["SKS"] }}</td>
              <td class="border p-2">{{ item["Pekerjaan Ayah"] }}</td>
              <td class="border p-2">{{ item["Pekerjaan Ibu"] }}</td>
              <td class="border p-2">{{ item["IPK"] }}</td>
              <td class="border p-2 font-semibold">{{ item["prediction"] === "LULUS" ? "Tidak putus Studi" : "Putus Studi" }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="mt-4 flex justify-between items-center">
        <button @click="page--, fetchPrediction()" :disabled="page <= 1 || loading" class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300">
          ← Sebelumnya
        </button>
        <span>Halaman {{ page }} dari {{ totalPages }}</span>
        <button @click="page++, fetchPrediction()" :disabled="page >= totalPages || loading" class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300">
          Selanjutnya →
        </button>
      </div>
    </div>
  </div>
</template>
